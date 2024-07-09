Elasticsearch Chart에 대한 Prometheus 설정을 수정하며 node 정보를 가져오기 위한 설정을 알아보았다.

Prometheus가 제공하는 example congifiguration은 다음과 같다.

```yaml
# Scrape config for nodes (kubelet).
#
# Rather than connecting directly to the node, the scrape is proxied though the
# Kubernetes apiserver.  This means it will work if Prometheus is running out of
# cluster, or can't connect to nodes for some other reason (e.g. because of
# firewalling).
- job_name: 'kubernetes-nodes'

  scheme: https

  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

  kubernetes_sd_configs:
  - role: node

  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)

# Scrape config for Kubelet cAdvisor.
#
# This is required for Kubernetes 1.7.3 and later, where cAdvisor metrics
# (those whose names begin with 'container_') have been removed from the
# Kubelet metrics endpoint.  This job scrapes the cAdvisor endpoint to
# retrieve those metrics.
#
# In Kubernetes 1.7.0-1.7.2, these metrics are only exposed on the cAdvisor
# HTTP endpoint; use the "/metrics" endpoint on the 4194 port of nodes. In
# that case (and ensure cAdvisor's HTTP server hasn't been disabled with the
# --cadvisor-port=0 Kubelet flag).
#
# This job is not necessary and should be removed in Kubernetes 1.6 and
# earlier versions, or it will cause the metrics to be scraped twice.
- job_name: 'kubernetes-cadvisor'

  # Default to scraping over https. If required, just disable this or change to
  # `http`.
  scheme: https

  # Starting Kubernetes 1.7.3 the cAdvisor metrics are under /metrics/cadvisor.
  # Kubernetes CIS Benchmark recommends against enabling the insecure HTTP
  # servers of Kubernetes, therefore the cAdvisor metrics on the secure handler
  # are used.
  metrics_path: /metrics/cadvisor

  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

  kubernetes_sd_configs:
  - role: node

  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
```

설정을 적용하며 혼란이 왔던 부분은 다음과 같다.

1. kubelet을 통해 kubernetes 정보를 가져온다고 되어 있는데 prometheus에서 제공하는 node exporter와 kubelet이 제공하는 메트릭의 차이점은 무엇인가? (kubelet이라기 보단 api-server를 통해 `/metric` endpoint에서 제공되는 메트릭이라고 표현하는 것이 맞는 표현이지만, 편의상 kubelet이라고 표현) 

    kubelet은 node의 시스템 메트릭이 아닌 kubelet demon에 의해 실행된 operation에 대한 정보를 제공한다. sysdig에서 kubelet에서 수집하는 데이터는 다음과 같다. ([https://sysdig.com/blog/how-to-monitor-kubelet/](https://sysdig.com/blog/how-to-monitor-kubelet/))

    ```
    "kubelet_running_pod_count"
    "kubelet_running_container_count"
    "volume_manager_total_volumes"
    "kubelet_node_config_error"
    "kubelet_runtime_operations_total"
    "kubelet_runtime_operations_errors_total"
    "kubelet_runtime_operations_duration_seconds*"
    "kubelet_pod_start_duration_seconds*"
    "kubelet_pod_worker_duration_seconds*"
    "storage_operation_duration_seconds*"
    "storage_operation_errors_total*"
    ...
    ```

    이와 다르게 node exporter는 node 레벨에서의 시스템 메트릭을 제공한다. Prometheus를 통해 수집된 node exporter의 메트릭은 다음과 같다.

    ```
    - node_network
    - node_netstat
    - node_memory
    - node_disk_write
    ...
    ```

    추가로 cAdvisor와 kubelet으로 제공되는 메트릭 간의 차이점도 알아보자.

    prometheus를 통해 수집한 cAdvisor의 메트릭은 다음과 같다.

    ```
    - container_cpu_*
    - container_fs_*
    - container_memory_*
    - container_network_*
    - container_spec_*
    ...
    ```

    **이를 통해 kubelet은 kubelet에 수행되는 operator에 대한 정보, cAdvisor는 container의 시스템 메트릭을 제공한다는 것을 알 수 있다.**

2. kubernetes 1.17부터 cAdvisor가 kubelet 내부에서 제외되었고, /metrics/cadvisor을 통해 제공된다고 하는데, 이전 버전에서는 cAdvisor에서 수집한 메트릭을 어떤 endpoint에서 노출하는가?

    kubernetes 1.16.3 기준으로 cAdvisor에 대한 endpoint는 `/api/v1/nodes/${node_name}/proxy/metrics/cadvisor` 으로 노출된다.
