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