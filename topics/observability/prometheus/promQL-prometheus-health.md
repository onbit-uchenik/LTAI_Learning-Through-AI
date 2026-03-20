1. Prometheus container current memory usage. This gives the ballpark idea of how much data is stored as head series - 
```
container_memory_working_set_bytes{pod=~"prometheus-prometheus-prometheus-0", container="prometheus"}
```
2. How much disk Prometheus is using (bytes)
```
prometheus_tsdb_storage_blocks_bytes{container="prometheus"}
```
3. Total number of series in the Head Block ( count )
```
prometheus_tsdb_head_series
```
4. 20 most expensive metric. Most expensive metrics is defined as total number of unique series per metric
```
topk(20,
  count by (__name__) ({__name__=~".+"})
)
```
5. WAL (Write-Ahead Log) disk size. A bloated WAL means checkpoint failures or high churn, and causes slow restarts
```
prometheus_tsdb_wal_storage_size_bytes
```
6. WAL checkpoint creation failures
```
prometheus_tsdb_checkpoint_creations_failed_total
```
7. WAL truncation failures
```
prometheus_tsdb_wal_truncations_failed_total
```
8. Compaction failures. Failed compactions mean blocks aren't being merged, leading to query slowness and disk bloat
```
prometheus_tsdb_compactions_failed_total
```
9. Samples ingested per second. Best metric for understanding ingestion load
```
rate(prometheus_tsdb_samples_appended_total[5m])
```
10. Out of order samples rejected per second. High rate means wasted scrapes or misconfigured targets
```
rate(prometheus_tsdb_out_of_order_samples_total[5m])
```
11. Series churn - creation rate. High creation rate means high cardinality churn and memory pressure
```
rate(prometheus_tsdb_head_series_created_total[5m])
```
12. Series churn - removal rate. Pair with creation rate to understand net series growth
```
rate(prometheus_tsdb_head_series_removed_total[5m])
```
13. Scrape targets that are down
```
up == 0
```
14. Scrape duration per target. Slow scrapes can cause timeouts and data gaps
```
scrape_duration_seconds
```
15. Per-target series count. Helps find targets pushing too many series
```
scrape_samples_scraped
```
16. Query performance. Slow queries consume memory and CPU
```
prometheus_engine_query_duration_seconds
```
17. OOM risk percentage. How close the container is to its memory limit
```
container_memory_working_set_bytes{container="prometheus"} / container_spec_memory_limit_bytes{container="prometheus"}
```
18. PVC disk usage percentage for Prometheus persistent volume
```
kubelet_volume_stats_used_bytes{persistentvolumeclaim=~"prometheus.*"} / kubelet_volume_stats_capacity_bytes{persistentvolumeclaim=~"prometheus.*"}
```
19. Config reload status. 0 means the last reload failed and a broken config is being silently ignored
```
prometheus_config_last_reload_successful
```