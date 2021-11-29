# Comment diagnostiquer les problèmes de performances

Si votre serveur SQL présente des lenteurs, vous devez absolument recueillir des indices, pour ne pas supposer des problèmes et des solutions qui ne correspondent pas à la nature du problème.

Si vous étiez médecin, presecririez-vous un traitement sans effectuer de diagnostic ?

## Hypothèses

D'où peuvent provenir les problèmes de performance ?

1. De requêtes mal écrites qui sont trop longues;
2. D'allers-retours excessifs entre le client et le serveur;
3. De blocages dus à trop de verrouillage;
4. D'exécution de triggers, de transactions trop longues;
5. D'un serveur mal dimensionné;
6. D'un manque d'index;
7. D'attentes, par exemple sur le parallélisme des requêtes;
8. Du code client, en non pas de SQL Server;
9. De problèmes de plans d'exécution, de compilation et de *parameter sniffing*.

## Comment vérifier les hypothèses ?

### Requêtes mal écrites qui sont trop longues

Utilisez le Query Store pour identifier les requêtes les plus consommatrices : [vidéo : Nicolas Souquet – Tout savoir sur le Query Store](https://youtu.be/Adwtl1QtYvI).

Utilisez une session d'évènements étendus :

- [Exemples de sessions sur mon Github](https://github.com/rudi-bruchez/tsql-scripts/tree/master/extended-events)
- Une vidéo YouTube sera bientôt postée

Requêtez la vue [dm_exec_query_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/execution-stats/dm_exec_query_stats.sql)

Utilisez, en temps réel, la procédure [sp_whoisactive](https://github.com/amachanic/sp_whoisactive/releases)

Si vous avez des procédures stockées, utilisez la vue [dm_exec_procedure_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/stored-procedures/procedure-execution-analysis.sql)

### Allers-retours excessifs entre le client et le serveur

Plus difficile à identifier. Le comportement attendu est un excès de petites requêtes répétitives, qui exécutent des allers-retours unitaires entre le client et le serveur, au lieu de requêtes ensemblistes. C'est un comportement à changer dans le code client.

Cherchez, avec une session d'évènements étendus sans filtre de coût de requête, des appels répétitifs de requêtes semblables, par exemple des suites d'inserts ou des `SELECT` unitaires semblables avec des changements de paramètres.

Requêtez la vue [dm_exec_query_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/execution-stats/dm_exec_query_stats.sql) en cherchant des requêtes peu consommatrices mais exécutées très souvent (triez sur la colonne `execution_count`)

Utilisez le Query Store pour identifier les requêtes les plus exécutées : [vidéo : Nicolas Souquet – Tout savoir sur le Query Store](https://youtu.be/Adwtl1QtYvI).

### Blocages dus à trop de verrouillage

Y a-t-il des blocages ? Interrogez la requête suivante : [Paul Randall, tell me where it hurts](https://www.sqlskills.com/blogs/paul/wait-statistics-or-please-tell-me-where-it-hurts/) et cherchez les attentes de type `LCK...`

Si vous trouvez des attentes de ce type, approfondissez le diagnostic [à l'aide des instructions suivantes](../verrouillage-et-transactions/blocages.md).

### Exécution de triggers, de transactions trop longues

Identifiez le coût de vos déclencheurs à l'aide de la vue de diagnostic [sys.dm_exec_triggers_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/execution-stats/trigger-stats.sql)

Cherchez les blocages éventuels avec les outils de la section précédente.

Si vous avez des procédures stockées, utilisez la vue [dm_exec_procedure_stats](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/stored-procedures/procedure-execution-analysis.sql)

Si vous soupçonnez des déclencheurs, des procédures stockées ou des appels de fonctions, créez une session d'évènements étendus avec l'évènement `sp_statement_completed` -- [code pour Azure à adapter pour on prem](https://github.com/rudi-bruchez/tsql-scripts/blob/master/extended-events/azure-sql-database/trace-procedure-create.sql)

### Serveur mal dimensionné

Interrogez la requête suivante : [Paul Randall, tell me where it hurts](https://www.sqlskills.com/blogs/paul/wait-statistics-or-please-tell-me-where-it-hurts/) et observez les attentes les plus communes. Observez bien le `signal time` pour juger de la pression sur les CPU.

Interrogez le Query Store, rapport "Overall Resource Consumption"

Vérifiez les latences d'IO sur les fichiers [à l'aide de la requête suivante](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/IO/dm_io_virtual_file_stats.sql).

Vérifiez les compteurs de buffer (cache de données) [à l'aide de la requête suivante](https://github.com/rudi-bruchez/tsql-scripts/blob/master/server-information/quick-audit.sql).

### Manque d'index

Vérifiez le [rapport d'index manquants](https://github.com/rudi-bruchez/tsql-scripts/blob/master/index-management/missing-indexes.sql).

Vérifiez que les index sont bien utilisés avec [le rapport d'utilisation des index](https://github.com/rudi-bruchez/tsql-scripts/blob/master/index-management/index-usage.sql).

Vous pouvez vous référer [à l'article suivant](https://rudi.developpez.com/sqlserver/tutoriel/vuesdm-index/).

### Problèmes de plans d'exécution, de compilation et de *parameter sniffing*

Lorsque les problèmes se posent, videz le cache de plans à l'aide de la commande suivante : `DBCC FREEPROCCACHE`. Est-ce que cela résoud le problème ? Vous pouvez exécuter cette commande au lieu de redémarrer un serveur SQL.

Vérifiez les statistiques :
    - [article un peu ancien](https://rudi.developpez.com/sqlserver/tutoriel/statistiques/)
    - [requête de diagnostic](https://github.com/rudi-bruchez/tsql-scripts/blob/master/diagnostics/tables/statistics.sql)