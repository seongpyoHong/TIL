### GO Environment
- GOROOT : 표준 라이브러리의 모든 패키지를 포함하는 디렉토리
- GOPATH : workspace의 root path
  - src : source code
  - bin : 실행 프로그램
  - pkg : 빌드 툴이 컴파일한 패키지

### Before Go Module
go module 이전에는 GOPATH가 변경되면 필요한 의존성을 다운받아야했다. 

- 만약 같은 패키지의 서로 다른 버전을 사용해야 하는 경우, 이를 해결 불가

왜? 패키지 명 = 디렉토리

⇒ go module 등장

### Go Module?

dependency management system

### Create new module

`go mod init` : 새로운 모듈 생성 (현재 디렉토리를 모듈의 루트로 지정)

- go.mod : 직접적인 연관이 있는 의존성들 표시
- go.sum : 항상 동일한 의존성을 다운받을 수 있도록 hash 저장

#### Dependency Upgrade

`go get -u`를 통해 업그레이드 가능 (같은 major 버전으로만 업그레이드)

- 그냥 go get은 다운로드만 (기존에 존재하면 업그레이드 X)

다른 메이저 버전의 패키지를 사용하려면?

```bash
github.com/seongpyoHong/test
github.com/seongpyoHong/test/v2
```

서로 다른 모듈 경로를 사용

- 같은 메이저 버전에 마이너 버전이 다른거는 불가능한가..?

go mod tidy를 통해 사용하지 않는 의존성 제거 가능

### Reference
- https://go.dev/blog/using-go-modules
- https://medium.com/rungo/everything-you-need-to-know-about-packages-in-go-b8bac62b74cc
