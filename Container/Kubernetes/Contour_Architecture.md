Envoy / Contour로 이루어짐
- Envoy : reverse proxy
	- Daemonset or Deployment로 배포
- Contour: management server for Envoy & provides it with configuration
	- Deployment로 배포

Envoy는 contour를 management server로 인식
- 만약 unavailable하면 gracefully retry => container startup ordering issue를 방지

Contour는 Kubernetes API Client로 Ingress/HTTPProxy/Gateway API/ Secret/ Service/ Endpoint를 watch
xDS로 envoy에 동적으로 설정 주입

### Reference
- XDS : https://m.blog.naver.com/alice_k106/222000680202
