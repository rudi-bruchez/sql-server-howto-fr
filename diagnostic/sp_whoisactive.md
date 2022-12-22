# Utiliser `sp_whoisactive` pour voir les requêtes actives

## Options utiles

- @get_plans -- tinyint -- 0 (default) or 1. If 1, the query plan is included in the output.
- @find_block_leaders BIT = 0 
- @get_memory_info, that exposes memory grant information for the query
- @get_query_stats, that exposes the query statistics for the query
