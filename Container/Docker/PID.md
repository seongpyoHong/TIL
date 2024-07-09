인턴 프로젝트 진행 중 Graylog Docker Image의 빌드 과정은 다음과 같다.
1. Graylog Binary 설치
2. Confd Binary 설치
3. Confd+환경 변수를 통해 동적으로 graylog.conf 생성
4. 생성한 graylog.conf로 graylog process 실행

처음 구성한 `entrypoint.sh`는 다음과 같다.
```shell
confd -onetime -confdir <confdir path> -backend env
graylogctl run
```
위의 entrypoint를 통해 생성한 이미지의 프로세스 목록을 보면 PID 1은 entrypoint가 수행되고, graylogctl은 PID 1에서 파생된 다른 PID에서 수행된다.

하지만, 이와 같은 상황은 다음 2가지 관점에서 수정할 필요가 있다.
1. 컨테이너는 하나의 역할을 해야하고 graylog 컨테이너의 목적은 graylog process 실행이다.
   
2. docker / kubernetes는 PID 1인 프로세스에게만 컨테이너 종료를 위한 신호를 보낼 수 있다.
    메인 프로세스가 PID 1이 아닌 곳에서 실행되는 경우, `SIGTERM`/`SIGINT`와 같은 신호가 아무런 영향을 미치지 않기 떄문에 SIGKILL을 발생시켜야 하고 이는 예상치 못한 장애를 발생시킬 수 있다. (모니터링 시스템에 에러 발생, 쓰기 중단, 좀비 프로세스에 따른 리소스 부족 문제)

해결 방법은 다음과 같다.
1. CMD / ENTRYPOINT를 통해 메인 프로세스를 직접 실행해 PID가 1이 되도록 설정한다.
    하지만, 메인 프로세스 이전에 환경 설정을 위한 shell script가 실행되어야 하는 경우가 존재한다. 이런 경우에는 `entrypoint.sh`을 통해 환경 설정을 진행하고 `exec` 명령어를 통해 shell script에서 프로세스를 실행한 후 원하는 프로세스로 대체해줘야한다.

2. 컨테이너용으로 제작된 `tini`와 같은 init 시스템을 사용해야 한다.

인턴 프로젝트에는 1번 방법을 적용하여 graylog 프로세스가 PID 1에 할당되도록 설정하였다. `Dockerfile`과 `Entrypoint.sh`은 다음과 같다.

**Dockerfile**
```
ENTRYPOINT ['entrypoint.sh]
CMD ['graylogctl', 'run']
```

**Entrypoint**
```
confd -onetime -confdir <confdir path> -backend env
exec "$@"
```