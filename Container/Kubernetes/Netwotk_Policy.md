Kubernetes에서는 NetworkPolicy를 통해  특정 application(pod)에 대한 L3/L4 트래픽을 제한할 수 있다.
> Network Policy는 network plugin에 의해 구현되기 때문에, 자신이 사용하는 network plugin이 Network policy를 지원하는지 먼저 확인이 필요하다. 

기본적으로 Kubernetes에서 Pod은 incoming traffic을 제한하지 않는다.  

=> 트래픽 제한을 위해  Network Policy를 설정하면 Pod은 이에 해당하는 트래픽을 제한한다. (여러 개인 경우에는 Union으로 적용됨, 순서에 상관없이  결과는 항상 동일함)

### Example
https://github.com/ahmetb/kubernetes-network-policy-recipes/blob/master/01-deny-all-traffic-to-an-application.md


### Reference
- [Network Policies | Kubernetes](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Securing Kubernetes Cluster Networking](https://ahmet.im/blog/kubernetes-network-policy/)
