

## JMX

**상황**

Graylog Cluster 운영 중 Heap Usage / Buffer Usage와 같은 metric에 대한 dashboard가 필요

**해결 방법**
1. JMX Collector 사용
2. Prometheus Exporter 사용

먼저 JMX Collector를 사용하는 방법 선택하여 진행하였다.

---

#### JMX란?

Java Application 실핼 중 여러 Metric을 관리하기 위한 Java API로 Java 5부터 추가되었다.

Java Application을 실행한 후, 터미널에 `jconsole`을 입력하면 JVM 및 JMX 설정을 확인할 수 있으며,  jconsole의 MBeans 탭에 JMX를 통해 수집할 수 있는 패키지와 Metric Variable들을 확인할 수 있다.

- 예를 들면, `java.lang` 패키지에서 기본적으로 제공하는 `Threading` Metric을 확인하면 `ThreadCount`, `PeakThreadCount`와 같은 Variable들을 확인할 수 있다.



JMX Collector가 제공된 JMX Metric을 수집하기 위해서는 Java Application을 실행할 때 remote connection이 가능하도록 설정을 추가해줘야 한다. 또한,  사용자 인증 없이 접근할 수 있도록 하기 위한 옵션도 추가했다.

**추가한 설정은 다음과 같다.**

```
-Dcom.sun.management.jmxremote.port=9333 
-Dcom.sun.management.jmxremote.rmi.port=9333 
-Dcom.sun.management.jmxremote.local.only=false
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false 
-Dcom.sun.management.jmxremote=true 
```

