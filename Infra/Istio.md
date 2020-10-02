### Istio란?
MSA를 사용할 때 서비스 호출 도중 오류 처리 및 재시도, 통신 및 트랜잭션 기록와 같은 네트워크 작업이 필요하다. 하지만 이런 작업을 어플리케이션 내부에서 부가적으로 처리하게 될 경우, 비즈니스 이외의 로직이 많아지고 모든 서비스 마다 로직이 중복되기 때문에 비효율적이다.

Istio는 이런 네트워크 작업을 위한 (서비스 간 통신을 위한) 인프라 계층을 제공해 복잡성과 문제를 제거한다.

**Istio 제공 기능?**

- Service Discovery

    사용 가능한 서비스 목록을 유지하고 요청을 보낸다. 

- Routing
    - 로드 밸런싱

        R-R, Weight와 같은 다양한 알고리즘을 기반으로 로드밸런싱 지원

    - 상태 확인

        사용 가능한 노드를 확인하기 위해 서비스의 작동, 실행, 응답 여부를 판단

- Resilience

    어플리케이션이 회로 차단기나 타임아웃, 재시도와 같은 알고리즘을 처리해준다.

- Security

    TLS 기반 암호화 지원

- Telemetry

    네트워크 호출 추적 및 메트릭 수집 가능

**아키텍쳐**

Istio는 사이드카 패턴을 사용하여 위와 같은 기능을 제공하며, 데이터 플레인과 컨트롤 플레인의 두 가지 구성 요소로 나누어진다.

- 데이터 플레인

    인스턴스로 들어오고 나가는 모든 네트워크 패킷을 변환, 전달 및 모니터링하는 역할을 한다.

    기본적으로 서비스 메시 전체에 배포된 사이드카 프록시로 볼 수 있고, Envoy를 주로 사용한다.

- 컨트롤 플레인

    데이터 플레인(프록시)는 자신 이외의 다른 프록시에 대해 알지 못하고, 컨트롤 플레인에게 정보를 제공받아 새 서비스의 존재를 파악한다.

    회로 차단, 로드 밸런싱, 타임아웃, 보안과 같은 정보는 컨트롤 플레인에 저장된다.

    > 기존에도 데이터 플레인은 Nginx, HAproxy, Envoy와 같은 프로젝트들이 존재하였지만 이를 위해 수동이나 다른 여러 도구를 사용해 구성을 설정해야했다. Istio는 이러한 boilerplate 구성을 제거하고 솔루션을 제공한다.

    컨트롤 플레인은 Mixer, Pilot, Citadel, Galley로 이루어진다.

    - Mixer

        플랫폼에 독립적으로 메트릭 수집, 권한 부여, 할당량 적용과 같은 기능을 수행한다. 이런 기능을 수행하는 부분을 `인프라 백엔드` 라고 하며 Mixer는 인프라 백엔드와 상호작용하는 부분을 추상화하여 제공한다.

        예를 들면, 로그를 전송하는 부분을 어플리케이션에서 책임지지 않고 Istio가 수집한 후 Mixer를 통해 Log 수집을 책임지는 인프라 백엔드(CloudWatch, Fluentd)와 상호작용한다.

        인프라 백엔드는 제공 업체마다 달라질 수 있기 때문에 Mixer는 `Adapter` 라는 범용 플러그인을 통해 인프라 제공 업체로 부터 추상화된 상태로 유지할 수 있게 해준다.

        다양한 인프라 백엔드를 사용하며 데이터나 요청을 어디로 보낼지 결정하는 설정은 `Configuration Model` 을 통해 정의한다.

        이를 통해 운영자는 서비스를 변경하지 않고도 인프라의 추가 및 제거를 제어할 수 있게 된다.

        > Istio 1.7 버전에서는 Mixer가 deprecated 되고 해당 기능이 envoy proxy로 옮겨졌다.

    - Pilot

         envoy에 대한 설정 관리를 담당하며, 전체 트래픽을 관리한다.

        > 트래픽 관리란?
        라우팅, 서비스 디스커버리 제공, 타임아웃, 재시도, 회로 차단 등

        - 플랫폼별 서비스 디스커버리 방식을 Istio와 분리하여 제공하기 때문에 여러 환경(k8s, nomad 등)에서 Istio를 실행할 수 있게 해준다.
        - 트래픽 관련 구성을 Envoy 구성으로 변환하고 런타임 시 사이드카로 푸시한다.
        - 파일럿 내부에 서비스 모델을 유지하고 여러 환경에서 들어오는 데이터를 서비스 모델로 변환함으로 환경에 무관하게 서비스 디스커버리를 담당할 수 있다.

    - Citadel

        Istio 메시 내에 요청을 TLS 기반으로 암호화 하는 역할, 서비스에 관한 RBAC를 제공한다. 

        (이 구성 또한 Pilot에 의해 Envoy로 푸시된다.)

    - Galley

        사용자 지정 이스티오 설정을 수용하고 이를 유효한 구성설정으로 변환한다.
        
### GKE Istio 설치

먼저 istio를 다운받는다.

```bash
▶ curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.2.2 sh
```

먼저 istio를 구성할 때 사용하는 CRD(Custom Resource Definition)를 설치한다.

```bash
▶ helm install istio-init istio-1.2.2/install/kubernetes/helm/istio-init
```

정상적으로 CRD가 설치되었는지 확인한다.

```bash
▶ kubectl get crds
NAME                                        CREATED AT
adapters.config.istio.io                    2020-09-24T15:44:05Z
attributemanifests.config.istio.io          2020-09-24T15:44:05Z
authorizationpolicies.rbac.istio.io         2020-09-24T15:44:09Z
backendconfigs.cloud.google.com             2020-09-24T15:17:42Z
clusterrbacconfigs.rbac.istio.io            2020-09-24T15:44:05Z
destinationrules.networking.istio.io        2020-09-24T15:44:05Z
envoyfilters.networking.istio.io            2020-09-24T15:44:05Z
gateways.networking.istio.io                2020-09-24T15:44:05Z
handlers.config.istio.io                    2020-09-24T15:44:05Z
httpapispecbindings.config.istio.io         2020-09-24T15:44:05Z
httpapispecs.config.istio.io                2020-09-24T15:44:05Z
instances.config.istio.io                   2020-09-24T15:44:05Z
managedcertificates.networking.gke.io       2020-09-24T15:17:01Z
meshpolicies.authentication.istio.io        2020-09-24T15:44:05Z
policies.authentication.istio.io            2020-09-24T15:44:05Z
quotaspecbindings.config.istio.io           2020-09-24T15:44:05Z
quotaspecs.config.istio.io                  2020-09-24T15:44:05Z
rbacconfigs.rbac.istio.io                   2020-09-24T15:44:05Z
rules.config.istio.io                       2020-09-24T15:44:05Z
scalingpolicies.scalingpolicy.kope.io       2020-09-24T15:17:00Z
serviceentries.networking.istio.io          2020-09-24T15:44:05Z
servicerolebindings.rbac.istio.io           2020-09-24T15:44:05Z
serviceroles.rbac.istio.io                  2020-09-24T15:44:05Z
sidecars.networking.istio.io                2020-09-24T15:44:05Z
storagestates.migration.k8s.io              2020-09-24T15:17:01Z
storageversionmigrations.migration.k8s.io   2020-09-24T15:17:01Z
templates.config.istio.io                   2020-09-24T15:44:05Z
updateinfos.nodemanagement.gke.io           2020-09-24T15:17:01Z
virtualservices.networking.istio.io         2020-09-24T15:44:05Z
```

대표적으로 많이 사용되는 CRD를 살펴보면 다음과 같다.

- Virtual Service

    서비스가 다른 호스트를 호출할 때 사용되는 트래픽 규칙을 정의한다.

- Destination Rule

    라우팅이 끝났을 때 작동되며, 로드 밸런싱, 연결 풀의 크기와 같은 구성을 다룬다.

- Service Entry

    자동 검색된 서비스가 수동으로 정의된 서비스에 액세스할 수 있도록 Istio 서비스 레지스트리에 추가 항목을 추가한다. (서비스 메시 외부의 서비스를 사용할 때 유용)

- Gateway

    서비스 메시 입구에서 특정 포트로의 외부 연결을 수신하고 메시 내부로 트래픽을 분산키는 역할

- Envoy Filter

    파일럿이 이미 생성한 것에서 Envoy 프록시 관련 필터를 정의하기 위해 사용된다.

- Policy

    서비스에 관한 트래픽 속도 제한, 헤더 재작성, 블랙리스트 작성과 같은 규칙을 실행한다.

이제 Istio를 설치한다.

```python
▶ helm install istio-init istio-1.2.2/install/kubernetes/helm/istio
```

배포된 Pod과 Service를 확인해보면 다음과 같다.

```bash
▶ kcl get svc
NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                                                                                                                                      AGE
istio-citadel            ClusterIP      10.3.243.134   <none>         8060/TCP,15014/TCP                                                                                                                           19m
istio-galley             ClusterIP      10.3.251.47    <none>         443/TCP,15014/TCP,9901/TCP                                                                                                                   19m
istio-ingressgateway     LoadBalancer   10.3.250.135   34.64.238.13   15020:31937/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:31265/TCP,15030:30673/TCP,15031:30408/TCP,15032:31206/TCP,15443:30714/TCP   19m
istio-pilot              ClusterIP      10.3.255.201   <none>         15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       19m
istio-policy             ClusterIP      10.3.248.245   <none>         9091/TCP,15004/TCP,15014/TCP                                                                                                                 19m
istio-sidecar-injector   ClusterIP      10.3.255.17    <none>         443/TCP                                                                                                                                      19m
istio-telemetry          ClusterIP      10.3.251.255   <none>         9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       19m
kubernetes               ClusterIP      10.3.240.1     <none>         443/TCP                                                                                                                                      50m
prometheus               ClusterIP      10.3.246.63    <none>         9090/TCP

▶ kcl get pod
NAME                                      READY   STATUS      RESTARTS   AGE
istio-citadel-ff6d9d6cd-tz5mm             1/1     Running     0          20m
istio-galley-5d56cc749-rrvdt              1/1     Running     0          20m
istio-ingressgateway-66b6494665-vkt9t     1/1     Running     0          20m
istio-init-crd-10-dj26j                   0/1     Completed   0          23m
istio-init-crd-11-8g6mb                   0/1     Completed   0          23m
istio-init-crd-12-z76t7                   0/1     Completed   0          23m
istio-pilot-589fdf8555-9kghj              2/2     Running     0          20m
istio-policy-67bfcf6d4f-qw2gt             2/2     Running     1          20m
istio-sidecar-injector-5867896465-nrd75   1/1     Running     0          20m
istio-telemetry-69c8c9bc97-2rwmk          2/2     Running     1          20m
prometheus-6bcbc95fb4-k59n5               1/1     Running     0          20m
```
