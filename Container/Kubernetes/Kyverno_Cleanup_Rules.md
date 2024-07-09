https://kyverno.io/docs/writing-policies/cleanup/
Kyverno에 cleanup policy라는 기능이 alpha로 추가됨

지금까지 특정 CR을 지우기 위해서 shell script로 로직을 작성하고, 별도 batch job이나 Cronjob으로 실행했었는데, 이런 부분들을 선언적으로 관리할 수 있어서 편리해보임 

보통 특정 시간이 보다 생성된지 오래된 CR을 지우는 니즈가 많았는데, timestamp도 비교가 가능한지는 확인 필요
