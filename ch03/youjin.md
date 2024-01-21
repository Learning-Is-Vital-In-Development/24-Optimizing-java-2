# Chapter3 - 하드웨어와 운영체제
< 최적화 이해에 필요한 HW와 OS 지식 기본 >
* 성능을 고민하는 개발자에겐 가용 리소스를 최대한 활용할 수 있도록 자바 플랫폼의 근간 원리와 기술을 알고 있어야 한다

## Memory
![](https://velog.velcdn.com/images/ujin2021/post/48acac0c-3961-4423-9d8e-6b0ba266c558/image.png)
![](https://velog.velcdn.com/images/ujin2021/post/d9bc0000-c0fc-4150-aa06-760f70cb19dc/image.png)

(출처: 에브리타임) 

트랜지스터 개수 증가 
➡️ 클록 속도 향상(초당 더 많은 명령어 처리 가능) 
➡️ 클록속도 만큼 데이터도 빨리 움직여야 하는데 메모리가 이 속도를 맞추기 어렵다💥
➡️ 데이터 도착 전 까지 CPU는 노는중 🤸🏻

### Memory Cache
* CPU Cache이다 (CPU에 있는 메모리 영역)
* 자주 access하는 데이터는 CPU cache에 보관한다
➡️ access 빈도가 높을수록 processor core(cpu)에 더 가까이 위치시킨다

**따라서 CPU에 가장 가까운 cache가 L1 cache, 그 다음은 L2 cache**
![](https://velog.velcdn.com/images/ujin2021/post/b8feca87-77a6-41be-bff5-46038817f4b7/image.png)

(출처: https://it.donga.com/215/)

### Cache 일관성 프로토콜
Memory Data ↔️ Cache Data 어떻게 맞추지?
원래는 `write-through` (동시기록) 방법을 사용했다
> write-through란, 매번 cache 연산 결과를 memory에 기록하는 것
➡️ 메모리 대역폭을 너무 많이 소요. 현재 거의 안씀

현재는 `write-back` 방법을 사용한다
> write-back이란, 매번 dirty cache 블록만 memory에 기록
➡️ 메인 메모리 traffic이 확 줄어든다

## Processor (CPU)
### TLB
* 가상 메모리 주소를 물리 메모리 주소로 매핑하는 속도를 높이기 위해 사용되는 cache

> 가상 메모리는 실제 물리적 메모리(RAM)의 한 부분을 사용하는 것이 아니라, 컴퓨터의 하드 드라이브나 SSD와 같은 보조 저장 장치의 일부를 실제 메모리처럼 사용하는 기술

CPU가 가상 주소를 가지고 메모리에 접근하려고 할 때 우선은 TLB에 접근하여 가상 주소에 해당되는 물리 주소를 찾는다

* 참고
  * https://ahnanne.tistory.com/15
  * https://about-myeong.tistory.com/35
  
### 분기 예측과 추측 실행
![](https://velog.velcdn.com/images/ujin2021/post/b055a3b8-dcb7-4bc7-8ec1-3d619152ae19/image.png)
(출처: https://rmagur1203.tistory.com/17)

### 하드웨어 메모리 모델
서로 다른 여러 CPU(멀티코어)가 일관되게 동일한 메모리 주소를 access 할 수 있는 방법?
➡️ JIT compiler인 javac, CPU는 일반적으로 코드 실행 순서를 바꿀 수 있다

![](https://velog.velcdn.com/images/ujin2021/post/64cec3cc-2806-4fa1-9d29-3eceb17a61ba/image.png)
(메모리 일관성과 코드 실행 순서 바꾸는 것이 어떤 관련이 있는지 아직 잘 이해가 안됨... ➡️ 책 뒷부분에서 다시 한번 설명을 한다고 하니 그때 이해해보려고 함)
  
## OS
OS의 주 임무는 여러 실행 process가 공유하는 리소스 access를 관장하는 일.
➡️ 한정된 리소스를 process에게 골고루 나눠주기

### MMU
가상주소를 물리주소로 변환한다
위에서 설명한 TLB 캐시가 들어있다

### Process Scheduler
CPU access를 통제한다
실행 Queue에 실행대상인 process/thread가 대기하고 있다.
실행 중인 process가 있는데 인터럽트 발생 시(혹은 I/O 요청 등으로 인해 Block 상태가 되면), 실행 Queue에 있는 process/thread를 실행시킨다.

### Context Switch
Context Switch는 OS의 Process Scheduler가 현재 실행 중인 process를 중지하고 다른 process를 실행시킬 때 발생
➡️ 비싼 작업이다 (오버헤드가 크다)

Context Switch 교환 시, TLB를 비롯한 다른 Cache들이 무효화 된다.

## 성능 핵심 지표 찾기
### CPU 사용률
``` shell
$vmstat 1 # 1초마다 한번씩 상태 결과 표시
```
* 결과보는 법(참고): https://chloro.tistory.com/108

## JVM과 OS
JVM에서 OS에 반드시 access해야 하는 경우가 있다(시스템 클록에서 시간 정보를 얻는 작업, thread scheduling 등)
➡️ `native` 키워드를 붙인 native 메서드로 구현
➡️ native 메서드는 C언어로 구현하지만, 여느 Java 메서드 처럼 액세스 가능
➡️ 어떻게? `JNI(Java Native Interface)`가 처리

![](https://velog.velcdn.com/images/ujin2021/post/af12bf76-f513-4540-9137-5c3a1817ff7a/image.png)

(출처: https://velog.io/@vrooming13/JNI-JAVA-Native-Interface)
