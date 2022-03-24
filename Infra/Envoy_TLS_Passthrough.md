Ref: https://www.envoyproxy.io/docs/envoy/latest/start/sandboxes/tls

- Example
```yaml
static_resources:
  listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.filters.network.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.tcp_proxy.v3.TcpProxy
          cluster: service-https
          stat_prefix: https_passthrough

  clusters:
  - name: service-https
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service-https
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service-https
                port_value: 443

```

위의 예제처럼 Envoy를 TCP proxy로 사용하게 되면, TLS passthrough를 수행한다. (Upstream에게 TLS 인증을 위임한다.)
