# Pourquoi le disque des journaux de transaction est-il plein ?

Les journaux de transactions des bases SQL Server conservent l'historique transactionnel des écritures dans les bases.

- Si le mode de récupération (_recovery model_) d'une base de données est COMPLET (_FULL_), le journal de transaction conserve toutes les informations pour qu'une sauvegarde de journal de transactions puisse les archiver.
- Si le mode de récupération (_recovery model_) d'une base de données est SIMPLE (_SIMPLE_), le journal de transaction est régulièrement vidé.

Problèmes possibles :

- Le mode de récupération d'une base nouvellement créée est COMPLET par défaut.
- Une base de données de production en mode COMPLET va retenir ce mode si elle est restaurée sur un serveur de développement ou de recette.

**Si une base est en mode COMPLET et qu'aucune sauvegarde de journaux (_BACKUP LOG_) n'est planifiée, le journal grandira à l'infini.**

Il n'y a que deux options :

1. planifier des sauvegardes de journaux, sur un serveur de production.
2. passer la base en mode SIMPLE, sur un serveur hors production.

## Tester le mode de récupération

- Vous pouvez utiliser [la requête suivante](https://github.com/rudi-bruchez/tsql-scripts/blob/main/database-information/databases.sql) pour voir la liste des bases et leur mode de récupération (colonne `recovery`).
- Ma procédure stockée [sp_databases](https://github.com/rudi-bruchez/tsql-scripts/blob/main/stored-procedures/sp_databases.sql) donne la même information.
- Vous pouvez voir les journaux et leur pourcentage de remplissages à l'aide de la commande `DBCC SQLPERF (LOGSPACE)`
- Vous pouvez utiliser ma procédure stockée [sp_logspace](https://github.com/rudi-bruchez/tsql-scripts/blob/main/stored-procedures/sp_logspace.sql) qui remplace avantageusement `DBCC SQLPERF (LOGSPACE)` 

Sinon, la requête suivante suffit

```sql
SELECT 
    name as [db], 
    recovery_model_desc as [recovery]
FROM sys.databases
WHERE database_id > 4
ORDER BY name; 
```

## Changer le mode de récupération

La requête suivante change le mode de récupération d'une base de données en SIMPLE :

```sql
use [master];
GO

ALTER DATABASE [<nom de la base de données>] SET RECOVERY SIMPLE WITH NO_WAIT
```

Vérifiez ensuite avec la commande `DBCC SQLPERF (LOGSPACE)` que le  pourcentage de remplissages du journal a baissé.

## Récupérer l'espace disque

La taille physique du journal ne diminue pas automatiquement. C'est seulement le remplissage du fichier qui est ajusté.

Pour diminuer la taille du fichier de journal, vous devez effectuer un [shrink](./shrink-database-file.md). Par exemple avec la commande suivante.

```sql
USE [<nom de la base de données>]
GO

DBCC SHRINKFILE (N'<nom logique du fichier de journal>' , 200)
```

Cette commande essaie de diminuer la taille du fichier à 200 Mo. Cela peut ne pas réussir tout de suite, si la portion active du journal est à la fin du fichier. Réessayez plus tard si la taille n'a pas diminué.

Ne planifiez pas les shrinks. Consultez [cet autre article](./shrink-database-file.md) et [ma vidéo YouTube](https://youtu.be/Bl0p6GREFg8).
