**GC엔 여러 종류가 있다.**
* CMS (거의 동시 수집기)
* G1 (최신 범용 수집기)
* 셰난도아
* C4
* 밸런스드
* 레거시 핫스팟 수집기

**모든 java구현체에 GC가 있는건 아니다**
➡️ GC는 탈착형 서브시스템으로 취급된다

**GC 선정 시 고려해야 할 것!**
* 중단시간 (최고의 관심사🌟)
* 처리율
  * application 런타임 대비 GC 시간 %
* 중단 빈도
  * GC 때문에 얼마나 자주 멈추는지
* 회수 효율
  * GC 사이클 당 얼마나 많은 가비지가 수집되는지
* 중단 일관성

**관심사와 trade off를 고려해여 수집기를 선택해야 한다**

# 동시 GC 이론
동시 수집기를 써서 application thread 실행 도중 수집에 필요한 작업 일부를 수행해서 중단 시간을 줄이는 것도 방법이다

## JVM Savepoint
STW GC를 실행하려면 application thread를 모두 중단시켜야 한다.
JVM이 이런 작업을 어떻게 수행하는걸까?
➡️ application thread 마다 savepoint라는 특별한 실행 지점을 둔다
➡️  GC가 수행되기 전과 같은 중요한 작업을 실행하기 전에, 모든 애플리케이션 스레드를 일시 정지시키는 지점을 말한다

### 풀 STW GC의 경우
* 전체 application thread를 중단시켜야 한다
* GC thread가 OS에게 무조건 application thread를 강제 중단시켜달라고 요청할 수는 없다
➡️ GC thread와 application thread는 반드시 공조 해야 한다
* savepoint 요청을 받았을 때 thread가 제어권을 반납하게 만드는 코드가 VM 인터프리터 구현체 어딘가에 있어야 한다

### Savepoint로 바뀌는 경우
1. JVM이 전역 savepoint time flag를 세팅한다
2. 각 application thread는 polling하면서 이 flag가 세팅되었는지 확인한다
3. application thread는 일단 멈췄다가 다시 깨어날 때 까지 대기한다

➡️ savepoint time flag를 세팅하면 모든 application thread는 반드시 멈추야 한다


## 삼색마킹(Tri-color marking)
savepoint는 GC 수행을 위해 모든 application thread를 중지 시키는 지점이다.
모두 중지시키고 난 후, 삼색마킹 알고리즘을 실행한다

> 삼색마킹 알고리즘에 사용되는 color
> * White: 아직 접근되지 않은, 즉 가비지 컬렉터가 아직 방문하지 않은 객체들을 나타냄. 이 객체들은 잠재적으로 가비지일 수 있다
> * Gray: 가비지 컬렉터가 방문했지만, 그 객체가 참조하는 다른 객체들을 아직 모두 검사하지 않은 상태. 즉, 이 객체는 살아있지만, 그 참조가 가리키는 객체들이 아직 검사 대기 상태
> * Black: 해당 객체와 그 객체가 참조하는 모든 객체들이 가비지 컬렉터에 의해 검사된 상태. 이는 더 이상 검사할 필요가 없는, 살아 있는 객체를 의미

![](https://velog.velcdn.com/images/ujin2021/post/1ec3ae43-0b9f-4c6d-9d16-2c7fa238dc7b/image.png)
(출처: [JVM Garbage Collection(GC)](https://velog.io/@zaccoding/JVM-Garbage-CollectionGC))

1. GC 루트를 Gray 표시 한다 (A)
2. 다른 객체는 모두 White 표시한다 (A 제외)
3. 마킹 thread가 임의의 Gray 노드로 이동한다 (A)
4. 마킹 thread가 White 표시된 자식노드를 만나면, 그 자식노드를 모두 Gray 표시한 후(B, C, D), 해당 노드를 Black 표시한다(A)
5. Gray노드가 하나도 남지 않을 때 까지 위 과정을 되풀이한다
6. Black 객체는 모두 접근가능하므로 살아남는다
7. White 노드는 더 이상 접근 불가능한 객체이므로 수집 대상이 된다

동시 수집을 하기 때문에, 변경자 thread(application thread)가 삼색마킹 알고리즘을 수행하는 동안 새 객체를 생성할 수 있다

# CMS
중단 시간을 아주 짧게 하려고 설계된 테뉴어드 공간 전용 수집기 이다.
마킹은 삼색 마킹 알고리즘을 따른다
➡️ 수집기가 heap을 탐색하는 도중 객체 그래프가 변경될 수 있다
➡️ 초기 수행 단계가 복잡하다

> 1. 초기 마킹(STW): GC 출발점을 얻는 것
> 2. 동시 마킹: 삼색마킹 알고리즘 적용
> 3. 동시 사전 정리
> 4. 재마킹
> 5. 동시 스위프
> 6. 동시 리셋

## 장점과 단점
* application thread가 오랫동안 멈추지 않는다
* CMS의 GC 처리 동안 application 처리율은 감소한다
* GC가 객체를 추적해야 하므로 메모리를 더 많이 쓴다
* GC 수행에 더 많은 CPU 시간이 필요한다

## 작동 원리
CMS는 application thread와 동시에 작동한다

**CMS 실행 도중 에덴 공간이 다 차버리면?**
➡️ application thread가 더 이상 진행할 수 없으니 실행이 중단되고, CMS 도중 영 GC가 일어난다
* 영 GC: Java 힙의 젊은 세대(Young Generation)에서 사용되지 않는 객체를 정리하는 과정

영 GC 이후, 일부 객체는 테뉴어드로 승격된다
CMS 실행동안에도 승격된 객체는 테뉴어드로 이동해야 한다
➡️ 두 수집기 간 긴밀한 조정이 필요하다

할당률이 급증하면 영 GC 수행 시, 조기 승격이 일어난다
➡️ 테뉴어드 공간이 부족해진다
➡️ **동시 모드 실패(CMF)** 라고 부른다

**동시 모드 실패가 일어나는 원인**
![](https://velog.velcdn.com/images/ujin2021/post/22adac63-7ff0-4aa6-b134-7b99c81367fd/image.png)

## 기본 JVM Flag
``` shell
$ -XX:+UseConcMarkSweepGC
```

플래그로 작동한다
