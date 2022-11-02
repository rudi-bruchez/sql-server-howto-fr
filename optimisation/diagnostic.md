# Comment diagnostiquer les problèmes de performances dans SQL Server

Si votre serveur SQL présente des lenteurs, vous devez absolument recueillir des indices, pour ne pas supposer des problèmes et des solutions qui ne correspondent pas à la nature du problème.

Si vous étiez médecin, prescririez-vous un traitement sans effectuer de diagnostic ?

## Hypothèses

D'où peuvent provenir les problèmes de performance ?

1. D'un serveur mal dimensionné;
2. D'une mauvaise configuration de SQL Server ou des bases de données;
3. De requêtes mal écrites qui sont trop longues;
4. D'allers-retours excessifs entre le client et le serveur;
5. De blocages dus à trop de verrouillage;
6. D'exécution de triggers, de transactions trop longues;
7. D'un manque d'index;
8. D'attentes, par exemple sur le parallélisme des requêtes;
9. Du code client, en non pas de SQL Server;
10. De problèmes de plans d'exécution, de compilation et de *parameter sniffing*.

## Comment vérifier les hypothèses ?

### Serveur mal dimensionné

SQL Server peut très bien fonctionner sur une machine moyennement dimensionnée, si la base de données et les requêtes sont optimisées.

Quelques pistes :

- Interrogez la requête suivante : [Paul Randall, tell me where it hurts](https://www.sqlskills.com/blogs/paul/wait-statistics-or-please-tell-me-where-it-hurts/) et observez les attentes les plus communes. Observez bien le `signal time` pour juger de la pression sur les CPU.
- Interrogez le Query Store, rapport "Overall Resource Consumption"
- Vérifiez les compteurs de buffer (cache de données) [à l'aide de la requête suivante](https://github.com/rudi-bruchez/tsql-scripts/blob/master/server-information/quick-audit.sql).
- Faut-il désactiver l'hyperthreading ? [Cela dépend](https://dba.stackexchange.com/questions/135982/should-i-disable-hyperthreading). En général, [vous pouvez](https://www.sqlrx.com/the-pros-cons-of-hyperthreading/). Si c'est un host VMWare, laissez-le activé, [VMWare se débrouille bien avec l'HT](https://www.davidklee.net/2020/04/10/sql-server-and-cpu-hyper-threading-in-virtual-environments/).

#### Y a-t-il un problème de disque ?

Pour déterminer si les performances des disques sont à blamer, vous pouvez :

- Suivre les compteurs de performance 
    - `Disque Physique / Moyenne disque s/lecture` sur le disque des fichiers de données et de tempdb.
    - `Disque Physique / Moyenne disque s/écriture` sur le disque des fichiers de journaux de transactions et de tempdb.
    - Quelques idées de valeurs dans [cet article de blog](https://www.sqlskills.com/blogs/paul/are-io-latencies-killing-your-performance/).
- Vérifiez les latences d'IO sur les fichiers [à l'aide de la requête suivante](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/IO/dm_io_virtual_file_stats.sql).
- Utiliser l'outil [Diskspd](https://docs.microsoft.com/fr-fr/azure-stack/hci/manage/diskspd-overview) ([Vous trouvez ici un article en anglais sur son utilisation](https://www.brentozar.com/archive/2015/09/getting-started-with-diskspd/))

### Mauvaise configuration de SQL Server ou des bases de données

Pour savoir si la configuration de l'instance est concernée :

- parallélisme ? La seule chose configurable qui peut changer les choses.
    - jouez un peu avec.
    - regardez les [attentes](https://www.sqlskills.com/blogs/paul/wait-statistics-or-please-tell-me-where-it-hurts/) de type CXPACKET. Si vous en avez beaucoup, regardez si l'hyperthreading est activé.
    - > The sys.dm_os_latch_stats DMV contains information about the specific latch waits that have occurred in the instance, and if one of the top latch waits is ACCESS_METHODS_DATASET_PARENT, in conjunction with CXPACKET, LATCH_*, and SOS_SCHEDULER_YIELD wait types as the top waits, the level of parallelism on the system is the cause of bottlenecking during query execution, and reducing the 'max degree of parallelism' sp_configure option may be required to resolve the problems.

#### Parallélisme

- Pour optimiser le parallélisme sur l'instance, Configurer les éléments suivants :
  - [Seuil de coût pour le parallélisme](https://learn.microsoft.com/fr-fr/sql/database-engine/configure-windows/configure-the-cost-threshold-for-parallelism-server-configuration-option) -- à mettre à **50**
  - [Degré maximum de parallélisme](https://learn.microsoft.com/fr-fr/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option) -- ne pas le laisser à 0 si vous avez plus de huit processeurs. 4 est souvent une bonne valeur pour les serveurs OLTP. Voir les [recommandations](https://learn.microsoft.com/fr-fr/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option?view=sql-server-ver16#recommendations)

- Depuis SQL Server 2016, vous pouvez configurer le degré de parallélisme par base de données, à l'aide d'une [configuration scopée](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql).

```sql
ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 1 ;
```

#### Configuration des bases

- **auto close** -- ça peut se sentir -- toujours à FALSE
- **auto shrink** -- toujours à FALSE. Cette option ne devrait même pas exister.
- **création automatique des statistiques** -- toujours à vrai
- **mise à jour automatique des statistiques** -- toujours à vrai
- En bref : laissez les options par défaut des bases.
- **niveau de compatibilité** -- le plus élevé possible pour bénéficier des améliorations du moteur relationnel.
- Gérez au besoin la version du moteur d'estimation de cardinalité (*legacy cardinality estimation*) selon les erreurs d'estimation de cardinalité dans vos requêtes.

### Requêtes mal écrites qui sont trop longues

- Utilisez le Query Store pour identifier les requêtes les plus consommatrices.
  - [vidéo : Nicolas Souquet – Tout savoir sur le Query Store](https://youtu.be/Adwtl1QtYvI).
  - [vidéo : Steven Naudet](https://youtu.be/EVchvLCk-IQ)
- Utilisez une session d'évènements étendus :
  - [Exemples de sessions sur mon Github](https://github.com/rudi-bruchez/tsql-scripts/tree/master/extended-events)
  - Pour apprendre à utiliser les évènements étendus, vous trouvez sur ma chaîne Youtube [une vidéo complète sur le sujet](https://youtu.be/SC7MfYsd-p0).
- Requêtez la vue [dm_exec_query_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/execution-stats/dm_exec_query_stats.sql)
- Utilisez, en temps réel, la procédure [sp_whoisactive](https://github.com/amachanic/sp_whoisactive/releases)
- Si vous avez des procédures stockées, utilisez la vue [dm_exec_procedure_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/stored-procedures/procedure-execution-analysis.sql)

### Allers-retours excessifs entre le client et le serveur

Plus difficile à identifier. Le comportement attendu est un excès de petites requêtes répétitives, qui exécutent des allers-retours unitaires entre le client et le serveur, au lieu de requêtes ensemblistes. C'est un comportement à changer dans le code client.

- Cherchez, avec une session d'évènements étendus sans filtre de coût de requête, des appels répétitifs de requêtes semblables, par exemple des suites d'inserts ou des `SELECT` unitaires semblables avec des changements de paramètres.
- Requêtez la vue [dm_exec_query_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/execution-stats/query_stats.sql) en cherchant des requêtes peu consommatrices mais exécutées très souvent (triez sur la colonne `execution_count`)
- Utilisez le Query Store pour identifier les requêtes les plus exécutées.
  - [vidéo : Nicolas Souquet – Tout savoir sur le Query Store](https://youtu.be/Adwtl1QtYvI).
  - [vidéo : Steven Naudet](https://youtu.be/EVchvLCk-IQ)

### Blocages dus à trop de verrouillage

Les blocages sont des attentes sur des verrous posés par d'autres sessions.

Y a-t-il des blocages ?

- Interrogez la requête suivante : [Paul Randall, tell me where it hurts](https://www.sqlskills.com/blogs/paul/wait-statistics-or-please-tell-me-where-it-hurts/) et cherchez les attentes de type `LCK...`
- Si vous trouvez des attentes de ce type, approfondissez le diagnostic [à l'aide des instructions suivantes](../verrouillage-et-transactions/blocages.md).
- Vous pouvez aussi planifier une vérification toutes les quelques minutes avec [le script suivant](https://github.com/rudi-bruchez/tsql-scripts/blob/main/diagnostics/locking/monitor-blocking.sql).
- Surveillez les transactions actives et les longues transactions. Vous pouvez utiliser [cette requête sur mon Github](https://github.com/rudi-bruchez/tsql-scripts/blob/c0530ebe80d5e413e4b57950fb0447e2e43825f0/diagnostics/execution/active-transactions.sql) pour voir les transactions actives.

### Exécution de triggers, de transactions trop longues

- Identifiez le coût de vos déclencheurs à l'aide de la vue de diagnostic [sys.dm_exec_triggers_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/execution-stats/trigger-stats.sql)
- Cherchez les blocages éventuels avec les outils de la section précédente.
- Si vous avez des procédures stockées, utilisez la vue [dm_exec_procedure_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/execution-stats/stored-procedures/procedure-execution-analysis.sql)
- Si vous soupçonnez des déclencheurs, des procédures stockées ou des appels de fonctions, créez une session d'évènements étendus avec l'évènement `sp_statement_completed` -- [code pour Azure à adapter pour on prem](https://github.com/rudi-bruchez/tsql-scripts/blob/master/extended-events/azure-sql-database/trace-procedure-create.sql)

### Manque d'index

- Vérifiez le [rapport d'index manquants](https://github.com/rudi-bruchez/tsql-scripts/blob/master/index-management/missing-indexes.sql).
- Vérifiez que les index sont bien utilisés avec [le rapport d'utilisation des index](https://github.com/rudi-bruchez/tsql-scripts/blob/master/index-management/index-usage.sql).
- Vous pouvez vous référer [à l'article suivant](https://rudi.developpez.com/sqlserver/tutoriel/vuesdm-index/).
- Utilisez le Database Tuning Advisor sur une requête ou une charge de travail.
- Si vous avez des requêtes plutôt analytiques, essayez les index ColumnStore.

### Problèmes de plans d'exécution, de compilation et de *parameter sniffing*

Lorsque les problèmes se posent, videz le cache de plans à l'aide de la commande suivante : `DBCC FREEPROCCACHE`. Est-ce que cela résout le problème ? Vous pouvez exécuter cette commande au lieu de redémarrer un serveur SQL.

Vérifiez les statistiques :
    - [requête de diagnostic](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/tables/statistics.sql)

### Erreur d'estimation de cardinalité

- un signe classique : la requête se dégrade avec le temps. Planifiez un recalcul des statistiques plus régulièrement.
    - vérifiez les statistiques avec [ce script](https://github.com/rudi-bruchez/tsql-scripts/blob/main/diagnostics/optimizer/statistics.sql), regardez les tables qui ont eu beaucoup de modifications.
        - [article un peu ancien](https://rudi.developpez.com/sqlserver/tutoriel/statistiques/)
    - regardez le plan d'exécution **actuel**
    - Utilisez [plan explorer](https://www.sentryone.com/plan-explorer)
    - lancez un recalcul avec `UPDATE STATISTICS `
    - Activez l'ancien moteur d'estimation de cardinalité.
    - ajoutez l'option dans la requête `OPTION (USE HINT('FORCE_LEGACY_CARDINALITY_ESTIMATION'))`
    - regardez si on utilise des variables de type table


<p align="right">
<i><small>[<a href="https://www.pachadata.com/contact/">Besoin de services avec SQL Server ? Contactez-moi</a>]</small></i>
</p>