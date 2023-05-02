HPA의 동작은 resources.limits이 아닌 resources.requests를 기반으로 동작한다.
즉, resources.requests.cpu: 100m / resources.limits.cpu: 1000m 으로 설정하고 80%일 때 Auto Scaling 되도록 설정한다면, request의 80%인 80m일 때 동작하게 된다.

- 관련 Issue: https://github.com/kubernetes/kubernetes/issues/72811
- https://github.com/kubernetes/design-proposals-archive/blob/8da1442ea29adccea40693357d04727127e045ed/node/resource-qos.md#compressible-resource-guaranteess
