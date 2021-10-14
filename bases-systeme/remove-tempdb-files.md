# Supprimer des fichiers de tempdb

La recommandation de Microsoft pour l'optimisation de `tempdb` est de créer plusieurs fichier, pour parraléliser les threads d'IO et diminuer la contention sur les pages d'allocations dans les fichiers.

A la base, la recommandation est de créer autant de fichiers que le système a de CPU. Mais cette recommandation ne doit pas être suivie strictement. Si vous avez un système avec 32 coeurs, vous n'allez pas créer 32 fichiers. Cela ne sert à rien et rend la gestion de l'espace dans `tempdb` plus difficile.

En général, quatre ou huit fichiers suffisent.

Si vous avez créé trop de fichiers, comment faire pout en diminuer le nombre ? L'opération habituelle est de vider les fichiers, et de les supprimer ensuite.

Pas si facile dans `tempdb`. [Erin Stellato a blogué sur le sujet en 2017](https://www.sqlskills.com/blogs/erin/remove-files-from-tempdb/)

La commande suivante, qui sert à vider un fichier pour le supprimer ensuite, provoque en général une erreur indiquant que la fichier contient une table de travail, qui bloque le vidage du fichier.

```sql
USE [tempdb];
GO
DBCC SHRINKFILE (logicalname, EMPTYFILE);
GO
```

La solution d'Erin Stellato est de redémarrer SQL Server en mode de configuration minimale (avec l'option `-f`) pour effectuer la suppression. En production, cela implique du temps d'indisponibilité ...

Andy Mallon [propose une solution qui semble fonctionner](https://am2.co/2020/04/fixing-tempdb/) (je ne l'ai pas encore testée). Je vous renvoie à son blog. L'idée est de créer un travail de l'agent SQL qui effectue es opération au démarrage du service. En effet, on peut planifier un travail pour s'exécuter au moment du démarrage, avant que SQL Server ait eu le temps de créer les fichiers de travail dans `tempdb`.


*[Besoin de services avec SQL Server ? Contactez-moi](https://www.pachadata.com/contact/)*
