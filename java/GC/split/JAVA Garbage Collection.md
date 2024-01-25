# GC
개발중에 발생하는 유효하지 않은 메모리를 가비지라고 하는데 HEAP 영역의 가비지를를 찾아서 주기적으로 제거하여 메모리를 회수하는 기능이다.

<aside>
💡 C, C++ 에는 메모리를 직접 해제해주어야 한다.
</aside>
## 장점
가비지 컬렉션이 메모리 관리를 해주기 때문에 한정된 메모리를 효율적으로 사용할 수 있고 개발자는 메모리 관리, 메모리 누수 문제에 신경쓰지 않고 개발에만 집중할 수 있다.
## 단점
- GC 가 동작할 때마다 Stop-The-World 가 발생하기에 실시간성이 매우 중요할 경우 큰 문제가 된다.
	- Stop-The-World : GC 가 작동하는 동안 GC 관련 Thread를 제외한 모든 JVM Thread 가 멈추며 어플리케이션 실행이 멈추는 현상
- 메모리가 언제 해제되는지 정확하게 알 수 없어 제어하기 힘들다. 
	- `System.gc()` 호출로 GC 를 동작하게 할 수 있지만 성능에 매우 큰 영향을 미친다.
- GC 가 너무 자주 실행되면 성능 하락의 문제가 된다. 

이런 단점들로 인해 개발자는 GC 를 효율적이게 실행하는 최적화 작업인 GC 튜닝을 생각해야 하고 JDK1.2 부터 java.lang.ref 패키지를 제공하여 코드로 GC와 상호작용할 수 있게 되었다.

# Weak Generational Hypothesis
아래 두가지를 포함한 가설로 `객체는 대부분 일회성이며 메모리에 남아있는 경우는 드물다` 라는 특성을 뜻한다.

1. 대부분의 객체는 금방 접근 불가능 ( Unreachable ) 한 상태가 된다. 
2. 오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다

이를 고려하여 HotSpot VM에서는 효율적인 메모리 관리를 위해서 객체의 생존 시간에 따라 Heap 영역을 물리적인 두가지 공간인 Young Generation, Old Generation 으로 나누었다.
## Young 영역 ( Young Generation )
새롭게 생성된 객체의 대부분이 위치하는 영역이다. 대부분의 객체가 금방 Unreachable 상태가 되기에 많은 객체가 Young 영역에 할당되었다가 사라진다. Young 영역이 가득 찬 경우 실행되는 가비지 컬렉션을 Minor GC 라고 부른다.
### Young 영역의 구성
- Eden : 새로 생성된 객체가 대부분 할당되는 영역
- Survivor 0, Survivor 1 : Minor GC 에서 1번 이상 살아남은 객체들이 있는 영역
Eden, Survivor 로 세부적으로 나눔으로써 보다 정확하게 불필요한 객체를 제거할 수 있게 한다.
### Minor GC 처리 과정
1. Eden, Survivor 의 Reachable 객체를 탐색한다.
2. Eden 에서 Reachable 객체들만 Survivor 영역으로 이동한다. 
3. Survivor 영역이 가득 차있는 경우 Survivor 영역의 Reachable 객체들만 다른 Survivor 영역으로 옮긴다.
4. Unreachable 한 객체들의 메모리를 해제한다.
5. Minor GC 에서 살아남은 객체들의 age 를 1 증가시킨다.

<aside>
💡 Age 란 Object Header 에 저장되는 Survivor 영역에서 객체의 생존 횟수로 일반적인 JVM 의 경우 생존 횟수가 31(6 bit)이 되면 Old Generation 으로 Promotion 된다.
</aside>
## Old  영역 ( Old Generation )
Young 영역에서 Reachable 상태를 유지하여 살아남은 객체가 있는 영역으로 Old 영역이 가득 찬 경우 실행되는 가비지 컬렉션을 Majoc GC 라고 한다. 
- 일부 JVM 이나, GC 알고리즘 구조에 따라 크기가 큰 객체는 바로 old generation 에 할당하여 Minor GC 가 너무 빈번하지 않게 일어나도록 한다.
- Old 영역에서 Young 영역의 객체의 참조가 이뤄질 때마다 그에 대한 정보가 저장되는 카드 테이블이 존재하는데  Minor GC 가 발생할 때 카드 테이블만 확인하여 효율적으로 참조되는 객체들을 탐색한다.
### Major GC 처리 과정
 Old 영역의 reachable 객체를 탐색한 뒤 Mark 되지 않은 객체들의 메모리를 해제한다.
- Young 영역보다 Old 영역이 크기 때문에 자주 일어나지는 않지만 오래 걸린다.
- Minor, Major 가 같이 이루어지는 것을 Full GC 라고 한다.

**Permanent 영역**
Heap 영역에 위치하며 Class 혹은 Method Code가 저장되는 영역으로 Method Area 라고도 한다. Java8 부터는 Permanent 영역이 사라지고 OS 에 의해 관리되는 Native 메모리 영역에 위치한 Metaspace 영역이 등장했다. ( 로드한 Class들의 Metadata가 저장되는 공간 )
# Mark-Sweep-Compact
GC 에서 Reachable 한 객체들만 살려두는 내부 알고리즘으로 GC 되어야 하는 객체를 식별, 제거한 뒤 파편화된 메모리 영역을 앞에서부터 채워나간다.
## Mark
Root Space 에서 그래프 순회를 통해 연결된 객체들이 각각 어떤 객체를 참조하고 있는지 마킹하여 Unreachable 한 객체를 찾는다.
## Sweep
참조되어 지지 않는 Unreachable 객체를 Heap 에서 제거한다.
## Compact
Sweep 후 분산된 객체들을 Heap의 시작 주소로 모아서 객체가 있는 부분과 없는 부분으로 압축한다. ( 하지 않는 GC 도 있음 )
# GC 알고리즘 종류
GC 수행시 Stop-the-World 가 발생하여 Heap 의 사이즈가 자바의 발전과 함께 커지면서 최적화를 위해 다양한 GC 알고리즘이 개발 되었다. 상황에 따라 필요한 GC 방식을 설정해서 사용할 수 있다.
## Serial GC
서버의 CPU 코어가 1개일 때 사용하기 위해 개발된 가장 단순한 GC 로 사용하면 안된다.
- Minor : Mark - Sweep
- Major : Mark - Sweep - Compact
```bash
java -XX:+UseSerialGC -jar Application.java
```
## Parallel GC
Serial GC 와 기본적인 알고리즘은 같지만 Minor GC 를 멀티 쓰레드로 수행한다. Java 8 의 기본 GC로 Throughput GC 라고도 한다.
- 메모리가 충분하고 코어 개수가 많을 때 유리한 GC
```bash
java -XX:+UseParallelGC -jar Application.java 
# -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
```
## Parallel Old GC
- Majoc GC 도 멀티 쓰레드로 수행도록 Parallel GC 를 개선한 버전
- Mark - Summary - Compact 방식
	- mark : old 영역을 region 별로 나누고 참조되는 객체들을 식별한다.
	- summary : region 별로 살아남은 객체들의 밀도가 높은 부분이 어디까지인지 dense prefix를 정한다. 오랜 기간 참조된 객체는 앞으로 사용할 확률이 높다는 가성하에 dense prefix 를 기준으로 compact 영역을 줄인다.
	- compact : compact 영역을 destination 과 source 로 나누어 살아남은 객체는 destination 으로 이동시키고 참조되지 않는 객체는 제거한다.
```bash
java -XX:+UseParallelOldGC -jar Application.java 
# -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
```
### CMS GC ( Concurrent Mark Sweep )
애플리케이션의 쓰레드, GC 쓰레드가 동시에 실행되어 stop-the-world 시간을 최대한 줄이기 위한 GC이다.
- 복잡한 여러 단계로 수행되기 때문에 다른 GC 대비 CPU 사용량이 높다.
- Compact 단계가 기본적으로 제공되지 않는다.
- 메모리 파편화 문제가 발생하기에 Compact 단계를 거치며 stop-the-world 시간이 더 길어질 수도 있다.
- Java 9 부터 deprecated 되고 java 14 에서부터 사용이 중지 되었다.
#### GC 의 실행 단계
1. initial mark : 클래스 로더에서 가장 가까운 객체 중 살아있는 객체만 찾기에 Stop-the-World 의 시간이 매우 짧다.
2. concurrent mark : initial mark 에서 찾은 객체가 참조하고 있는 객체를 따라가며 확인한다. 다른 쓰레드가 실행중인 상태에서 동시에 실행된다.
3. Remark : concurrent mark 에서 새로 추가되거나 참조가 끊긴 객체를 확인한다.
4. concurrent sweep : 살아있지 않은 객체들을 제거한다.
```bash
# deprecated in java9 and finally dropped in java14
java -XX:+UseConcMarkSweepGC -jar Application.java
```

### G1 GC ( Garbage First )
Heap 영역을 Region 이라는 영역으로 분할하여 사용하는 GC이다. 메모리를 일일히 탐색하지 않고 메모리가 많이 차있는 영역을 인식하여 우선적으로 GC 한다. 살아남은 객체는 더 효율적이라고 생각하는 영역으로 재할당된다.
- 체스같이 Region 영역을 분할하여 상황에 따라 Eden, Survivor, Old 등 역할을 동적으로 부여
	- Humongous region : 크기의 50프로를 초과하는 객체를 저장하는 region
	- Available / Unused : 사용되지 않는 region
- compact 과정에서도 heap 메모리 전체에 대해서 진행하지 않고 region 내에서 이루어진다.
- 가바지로 가득한 영역 자체를 회수하여 빈 공간을 확보한다.
- CMS GC 를 대체하기 위해 jdk 7 버전에서 최초로 release 되고 Java 9 버전 이후의 디폴트 GC 로 지정되었다.
```bash
java -XX:+UseG1GC -jar Application.java
```