`CGO_ENABLED` Option은 Golang에서 C Code를 쉽게 호출할 수 잇는 도구인 CGO의 사용 여부를 결정하는 옵션.
다만 cross platform 지원을 위해서는 C 라이브러리가 플랫폼마다 다르기 때문에 이를 비활성화 (`CGO_ENABLED=0`) 해야할 수도 있다.
