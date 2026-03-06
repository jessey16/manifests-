Dashboard Component,PromQL Query,Purpose
CPU Utilization Spike,sum(node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate)---->Captures the real-time processing load during migration.
Migration Status,"max(kube_pod_status_phase{pod=""migration-project-demo"", phase=""Succeeded""})"----------->"Returns 1 when the pod finishes successfully, triggering the Green Box."
Cluster Memory,sum(container_memory_usage_bytes) ---------->/ 1024 / 1024,Monitors RAM usage in MB to ensure cluster stability.
