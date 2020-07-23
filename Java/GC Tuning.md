### Garbage Collection 튜닝

GC 튜닝의 목적은 상황에 따라 다르지만 대게 2가지를 목적으로 한다.

1. Young -> Old 영역으로 이동하는 객체의 수를 최소화

   Old 영역에서 발생하는 Full GC의 경우, Young 영역에서 발생하는 Minor GC 보다 실행시간이 길기 떄문에 Old 영역으로 이동하는 객체의 수를 줄여 Full GC가 발생하는 빈도를 줄일 수 있다.

2. Full GC 시간 단축

   Full GC의 실행 시간이 길어지면 (대게 1초 이상) 어플리케이션과 연동된 다른 정상적인 서비스에서 부터 타임아웃(장애)가 발생할 수 있다. Full GC의 시간을 단축시키는 가장 빠른 방법은 Old 영역의 크기를 줄이는 방법이지만 이는 장기적으로 보았을 때, OOM이 발생할 수 있다.

   따라서, 여러 설정값들의 적절한 활용을 통해 최적화를 진행해야 한다.

참고 : https://d2.naver.com/helloworld/37111



자세한 GC Tuning에 대한 글은 추후 Tuning을 하게 될 떄 다시 참고하기로 한다.

- https://johngrib.github.io/wiki/java-gc-tuning/

- https://johngrib.github.io/wiki/java-g1gc/
