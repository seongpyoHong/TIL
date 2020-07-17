## Garbage Collection

### stop-the-world

GC를 실행하기 위해 JVM이 어플리케이션 실행을 멈추는 것으로, GC를 실행하는 쓰레드를 제외한 나머지 쓰레드의 모든 작업을 중지한다.

- GC를 튜닝하는 작업은 `stop-the-world` 시간을 줄이는 작업

Java는 프로그램 코드에서 메모리를 명시적으로 해제하지 않고, GC가 더 이상 필요없는 객체를 찾아 지우는 작업을 담당한다. GC는 두가지 전제조건을 가지고 있다. (Weak Generatinal Hypothesis)

1. 대부분의 객체는 생성된 후 곧 unreachable 상태가 된다.
2. 오래된 객체에서 젊은 객체로의 참조는 매우 적게 존재한다.

Weak Generational Hypothesis의 장점을 활용하기 위해 HotSpot VM에서는 크게 2개(Young/Old)로 물리적 공간을 나눈다 

- Young Generation Area

  새롭게 생성된 객체의 대부분이 위치하는 영역으로, 대부분의 객체가 금방 접근 불가능한 상태가 되기 때문에 Young 영역에서 많은 객체가 생성되었다가 사라진다. 객체가 소멸되는 과정을 `Minor GC`라고 한다.

- Old Generation Area

  Young 영역에서 살아남은 객체가 복사되는 영역으로 대게 Young 영역보다 크게 할당하며 GC는 적게 발생한다. Old 영역에서 객체가 소멸되는 과정을 `Major GC`라고 한다.

##### Card Table

Old 영역의 객체가 Young 영역의 객체를 참조하는 경우 정보가 표시되는 곳으로 Minor GC를 실행할 때에는 Old 영역의 모든 객체의 참조를 확인하지 않고, Card Table만 확인하여 GC 대상인지 식별한다.

---

### Young Generation Area 구성

Young 영역은 3개의 영역으로 구성된다.

- Eden 영역
- Survivor 영역 (2개)

각 영역의 처리 절차는 다음과 같다.

1. 새로 생성된 객체는 Eden 영역에 위치한다.
2. Eden 영역에서 GC가 한 번 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동한다.
3. Eden 영역에서 GC가 발생하여 살아남은 객체는 이전에 살아남은 객체가 존재하는 Survirvor 영역으로 이동한다.
4. 하나의 Survivor 영역이 가득 차게 되면 그 중 살아남은 객체를 비어있는 Survivor 영역으로 이동시킨다. 가득 차있던 Survivor 영역은 비어있는 상태가 된다.
5. 1 - 4 과정을 반복하여 살아남은 객체는 Old  영역으로 이동된다.

이 때, Survivor 영역 중 하나는 반드시 비어 있는 상태로 남아 있어야 한다. 그렇지 않다면 비정상적인 상황이라고 인식하면 된다.



**참고**

HotSpot JVM에서는 보다 빠른 메모리 할당을 위해 `bump-the-pointer` / `Thread-Local Allocation Buffers` 라는 기술을 사용한다.



---

#### Old Generation Area GC

Old 영역은 기본적으로 데이터가 가득 차면 GC를 실핸한다. JDK 7 기준으로 GC 방식은 5가지가 존재한다.

- Serial GC
- Parallel GC
- Parallel Old GC
- Concurrent Mark & Sweep GC (CMS)
- G1(Garbage First) GC


---
- **Serial GC** (-XX:+UseSerialGC)
  Young 영역에서의 GC는 위에서 설명한 방식을 사용한다. Old 영역의 GC는 `mark-sweep-compact` 알고리즘을 사용한다.

  - mark-sweep-compact
    1. Old 영역에 살아있는 객체를 식별한다. (Mark)
    2. heap의 앞 부분부터 확인하여 살아있는 객체만 남긴다.(Sweep)
    3. 각 객체들이 연속되게 쌓이도록 힙의 가장 앞 부분부터 채워서 객체가 존재하는 부분과 객체가 없는 부분으로 나눈다. (Compaction)

  Serial GC는 CPU Core가 1개만 있을 때 사용하기 위한 방법으로 어플리케이션의 성능을 위해 사용하면 안된다.
---


- **Parallel GC(-XX:+UseParallelGC)**
  Serial GC와 기본적인 알고리즘은 같지만 Parallel GC는 GC를 처리하는 쓰레드가 여러 개이다. 메모리가 충분하고 코어의 개수가 많을 떄 유리하다.

---

- **Parallel Old GC(-XX:+UseParallelOldGC)**
  JDK 5부터 제공한 GC 방식으로 Parallel GC와 다르게 Old 영역의 GC 알고리즘으로 `Mark-Summary-Compaction` 사용한다. 


---
- **CMS GC(-XX:+UseConcMarkSweepGC)**
  1. Initial Mark : 클래스 로더에서 가장 가까운 객체 중 살아있는 객체만 찾는다. => 중단 시간이 매우 짧음
  2. Concurrent Mark : Initial Mark에서 살아있는 객체에서 참조하고 있는 객체들을 따라가며 확인
  3. Remark : Concurrent Mark에서 새로 추가되거나 참조가 끊긴 객체를 확인
  4. Concurrent Sweep : 더이상 접근 불가능한 객체 제거

  이 중, Concurrent Mark/Concurrent Sweep 과정은  다른 스레드가 실행 중인 상태에서 동시에 진행된다. 따라서 `stop-the-world` 시간이 매우 짧고, 모든 어플리케이션의 응답 속도가 중요한 경우 CMS GC를 사용한다.

  하지만, 다른 GC 방식보다 메모리나 CPU를 많이 사용하며, Compaction 단계를 제공하지 않는다는 단점이 존재하기 때문에 검토 후 사용해야 한다.

---

- **G1 GC**

  자바 heap 공간을 고정된 크기의 region으로 나누고 free한 region들의 리스트 형태로 관리한다. 메모리 공간이 필요해지면 free region를 young 영역나 old 영역로 할당한다. 이 때, region의 크기는 1MB 에서 32MB로 전체 heap 사이즈 용량이 2048개의 region으로 나누어 질 수 있도록 하는 범위 내에서 결정된다. region이 비어지면 이 region은 다시 free region 리스트로 돌아간다. 

  **G1 GC의 기본 원리는 자바 heap의 메모리를 회수할때 최대한 살아 있는 객체가 적게 들어 있는 region을 수집하는 것이다**. 

  G1 GC는 기본적인 GC의 동작 방식이 위에서 설명한 Young/Old과 비슷하지만 영역의 개념이 물리적으로 존재하지 않고 논리적으로만 존재함으로써 메모리 공간과 GC에 걸리는 시간을 줄일 수 있다는 장점이 있다. 

---
### 참고자료

- https://d2.naver.com/helloworld/1329
- https://b.luavis.kr/server/g1-gc

