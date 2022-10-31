# Réaliser des sauvegardes sur un groupe de disponibilité

1. Télécharger et installer [la procédure d'Ola Hallengren](https://ola.hallengren.com/sql-server-backup.html).
2. Planifier des sauvegardes complètes en COPY ONLY sur chaque réplica. On part du principe que le groupe de disponibilité est configuré pour préférer les sauvegardes sur le réplica secondaire.
   Pour le vérifier, vous pouvez utiliser [la requête suivante](https://github.com/rudi-bruchez/tsql-scripts/blob/main/hadr/availability-groups.sql), et vérifier la valeur de la colonne `automated_backup_preference`.
3. Planifier des sauvegardes de journaux sur chaque réplica. La proécure d'Ola Hallengren déclanche autoamtiquement la sauvegarde sur le réplica préféré.
   Vous pouvez utiliser [les exemples d'appels sur mon Github](https://github.com/rudi-bruchez/tsql-scripts/blob/main/database-administration/dba-database/012.ola-backups-ag.sql).
4. Alternativement, vous supprimer les anciennes sauvegardes, éventuellement de façon planifiée, vous pouvez utiliser [dbatools](https://dbatools.io/download/) et la commande suivante :

```powershell
Remove-DbaBackup -Path '\\dump-sql.naxos.local\repo$\srv-cnet-clu01$CLUSTER-SQL-AG\Century_21\' -BackupFileExtension bak -RetentionPeriod 48h
```