# Identifier et régler les problèmes de journaux de transactions

Procédure [sp_logspace](https://github.com/rudi-bruchez/tsql-scripts/blob/main/stored-procedures/sp_logspace.sql) pour voir la taille du journal de transactions.

Pour voir la raison pour laquelle le journal de transactions ne se vide pas :

```sql
-- pourquoi le journal ne se vide pas
SELECT 
    d.name, 
    d.log_reuse_wait_desc AS log_reuse_wait
FROM sys.databases d
WHERE d.log_reuse_wait > 0
ORDER BY d.name;
```

```sql
-- sessions ouvertes qui maintiennent une transaction
SELECT *
FROM sys.dm_exec_sessions des
WHERE des.is_user_process = 1
AND des.open_transaction_count > 0;
```

https://github.com/rudi-bruchez/tsql-scripts/blob/c0530ebe80d5e413e4b57950fb0447e2e43825f0/diagnostics/execution/active-transactions.sql

<p align="right">
<i><small>[<a href="https://www.pachadata.com/contact/">Besoin de services avec SQL Server ? Contactez-moi</a>]</small></i>
</p>
