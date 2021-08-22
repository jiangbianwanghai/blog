---
title: redis info 各个参数的详细说明
date: 2021-08-21 09:13:57
tags: ["Redis", "监控分析"]
---

为了方便对redis进行监控管理，一些公司会自己开发监控，或在已有的系统中添加功能。对redis info信息的获取是必须要处理的。Redis Info信息包括Server,Clients,Memory,Persistence,Stats,Replication,CPU,Commandstats,Cluster,Keyspace等，下边我们结合一个项目的真实信息详细介绍各部分对应关系。

```
# Server #服务器模块
redis_version:5.0.5 #redis的版本是5.0.5
redis_git_sha1:abc22daa
redis_git_dirty:1
redis_build_id:3ebe553cd2fe33fd
redis_mode:standalone
os:Linux
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:0.0.0
process_id:11296
run_id:776c1721d5319d2e9fb9f776b2e04b5374ea1b83
tcp_port:6379
uptime_in_seconds:20634790
uptime_in_days:238
hz:10
configured_hz:10
lru_clock:2117525
executable:
config_file:
support_ptod:1

# Clients
connected_clients:14
client_recent_max_input_buffer:4
client_recent_max_output_buffer:0
blocked_clients:0

# Memory
used_memory:79262664
used_memory_human:75.59M
used_memory_rss:125497344
used_memory_rss_human:119.68M
used_memory_peak:2159708488
used_memory_peak_human:2.01G
used_memory_peak_perc:3.67%
used_memory_overhead:42842930
used_memory_startup:4995912
used_memory_dataset:36419734
used_memory_dataset_perc:49.04%
allocator_allocated:79684488
allocator_active:86990848
allocator_resident:136966144
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:2147483648
maxmemory_human:2.00G
maxmemory_policy:volatile-lru
allocator_frag_ratio:1.09
allocator_frag_bytes:7306360
allocator_rss_ratio:1.57
allocator_rss_bytes:49975296
rss_overhead_ratio:0.92
rss_overhead_bytes:-11468800
mem_fragmentation_ratio:1.58
mem_fragmentation_bytes:46234936
mem_not_counted_for_evict:3700
mem_replication_backlog:33554432
mem_clients_slaves:17042
mem_clients_normal:338216
mem_aof_buffer:3700
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0
oom_err_count:0

# Stats
total_connections_received:29068603
total_commands_processed:582395773
instantaneous_ops_per_sec:5968
instantaneous_write_ops_per_sec:2981
instantaneous_read_ops_per_sec:2987
total_net_input_bytes:174080973852
total_net_output_bytes:465560453466
instantaneous_input_kbps:152.13
instantaneous_output_kbps:44.70
rejected_connections:0
rejected_connections_status:0
sync_full:2
sync_partial_ok:1
sync_partial_err:1
expired_keys:8691520
expired_stale_perc:0.00
expired_time_cap_reached_count:0
evicted_keys:779165
evicted_keys_per_sec:0
keyspace_hits:48424220
keyspace_misses:15231242
hits_per_sec:4
misses_per_sec:0
hit_rate_percentage:100.00
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:7571
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0
traffic_control_input:0
traffic_control_input_status:0
traffic_control_output:0
traffic_control_output_status:0
stat_avg_rt:0
stat_max_rt:157
pacluster_migrate_sum_rt:0
pacluster_migrate_max_rt:0
pacluster_migrate_qps:0
pacluster_import_sum_rt:0
pacluster_import_max_rt:0
pacluster_import_qps:0
pacluster_migrate_start_time:0
pacluster_importing_start_time:0
slot_psync_ok:0
slot_psync_err:0

# Replication
role:master

# CPU
used_cpu_sys:15834.015191
used_cpu_user:41037.999928
used_cpu_sys_children:50.804649
used_cpu_user_children:15.952634

# Cluster
cluster_enabled:0
databases:256
nodecount:1

# paCluster
pacluster_enabled:0

# Keyspace
db0:keys=44930,expires=41583,avg_ttl=37019868443
```
