# Poblèmes de mémoire

SQL Server consomme toute la mémoire qu'on lui donner. La mémoire maximum est définie à l'aide de l'option `max memory`.

Pour la voir :

```sql
SELECT *
FROM sys.configurations c
WHERE name = N'max server memory (MB)'
OPTION (RECOMPILE);
```

La mémoire réellement consommée est visible avec des compteurs de performance de SQL Server :

```sql
SELECT *
FROM sys.dm_os_performance_counters pc
WHERE pc.[object_name] LIKE '%:Memory Manager%'
OR pc.[object_name] LIKE '%:Memory Node%'
OPTION (RECOMPILE);
```

Les clercks sont les modules qui attribuent de la mémoire aux différentes parties de SQL Server :

```sql
SELECT 
	IIF(GROUPING(mc.type) = 1, '[TOTAL]', mc.type) AS [type],
	SUM(mc.pages_kb) AS pages_kb
FROM sys.dm_os_memory_clerks mc
WHERE mc.pages_kb > 0
GROUP BY mc.type
WITH ROLLUP
ORDER BY pages_kb DESC
OPTION (RECOMPILE);
```

La mémoire peut être consommée par des assemblies .NET.

Quelles sont les assemblies chargées, et la mémoire qu'elles consomment :


```sql
SELECT dca.appdomain_address
	  ,dca.appdomain_id
	  ,dca.appdomain_name
	  ,dca.creation_time
	  ,DB_NAME(dca.db_id) AS db
	  ,dca.state
	  ,dca.strong_refcount
	  ,dca.weak_refcount
	  ,dca.cost
	  ,dca.value
	  ,dca.total_processor_time_ms
	  ,dca.total_allocated_memory_kb
	  ,dca.survived_memory_kb
FROM sys.dm_clr_appdomains dca
OPTION (RECOMPILE);
```