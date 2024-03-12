# JIT Compile에 대해
## JIT Compile
* 프로그램을 실행하는 중에 기계어로 번역하는 컴파일 기법
* byte code ➡️ native code(기계어)로 변환한다

# JIT Watch
* 오픈소스이다
* 실행중인 java application이 생성한 핫스판 컴파일 상세 로그를 파싱/분석하여 GUI형태로 보여준다
* application 실행 중 핫스팟이 실제 byte code에 무슨일을 했는지 알아보는데 도움이 된다
* 다음 플래그를 추가해야 JITWatch에 입력할 로그를 생성한다
``` shell
-XX:+UnlockDiagnosticVMOptions -XX:+TraceClassLoading -XX:+LogCompilation
```

# JIT Compile
* 핫스팟(JVM의 한 종류)은 프로파일 기반 최적화(PGO)를 이용해 JIT 컴파일 여부를 판단한다
* 핫스팟은 실행 프로그램 정보를 메서드 데이터 객체(MDO)라는 구조체에 저장한다

**MDO?**
바이트코드 인터프리터와 C1 컴파일러에서 JIT 컴파일러가 언제, 무슨 최적화를 할지 결정하는데 필요한 정보를 기록하는 것

**컴파일러 최적화 기법**
* 인라이닝
* 루프 펼치기
* 탈출 분석
* 락 생략/확장
* 단일형 디스패치
* 인트린직
* 온-스택 치환

C1, C2 컴파일러
* C1: 추측성 최적화를 하지 않는다
* C2: 공격적인 최적화기 이다. 추측을 통해 최적화를 한다. 추측성 최적화 전엔 "가드" 라는 타당성 검사를 한다

# 컴파일러 최적화 기법
## 인라이닝
* 호출된 메서드의 콘텐츠를 호출한 지점에 복사하는 것
### 인라이닝 예시
``` java
// 인라이닝 전
int result = add(a, b);

private int add(int x, int y) {
	return x + y;
}

// 인라이닝 후
int result = a + b;
```
### 인라이닝 장점
다음과 같은 오버헤드를 줄일 수 있다
* 전달한 매개변수 세팅
* 호출할 메서드를 정확하게 룩업
* 새 호출 프레임에 맞는 런타임 자료구조 생성
* 새 메서드로 제어권 이송
* 호출부에 결과 반환

### 인라이닝 특징
다른 최적화(탈출 분석, 죽은 코드 제거, 루프 펼치기, 락 생략) 의 범위를 확장시키는 역할을 한다

### 인라이닝 제한
* VM 차원에서 인라이닝 서브시스템에 제한을 걸어야 할 때도 있다
* 제약조건이 없다면 컴파일러는 매우 깊은 호출 체인까지 파헤치며 인라이닝 할 것 이기 때문
  * 그렇게 된다면 코드캐시를 거대한 native method로 가득 채우게 된다
  
**아래 항목을 따져보며 어떤 메서드를 인라이닝 할 지 결정한다**
* 인라이닝할 메서드의 바이트코드 크기
* 현재 호출 체인에서 인라이닝 할 메서드의 깊이
* 메서드를 컴파일한 버전이 코드 캐시에서 차지하는 공간

JIT Watch를 통해서도 인라이닝 거부된 메서드를 확인할 수 있다.

### 인라이닝 스위치
* `-XX:MaxInlineSize=<n>`: 메서드를 이 크기 이하로 인라이닝 한다
* `-XX:FreqInlineSize=<n>`: 자주 호출되는 메서드를 이 크기 이하로 인라이닝 한다
* `-XX:InlineSmallCode=<n>`
* `-XX:MaxInlineLevel=<n>`: 이 수준 보다 더 깊이 호출프레임을 인라이닝 하지 않는다

중요메서드가 최대 허용 크기를 살짝 초과해 인라이닝 되지 않는 경우, 위의 스위치로 JVM 매개변수를 조정할 수 있다(`-XX:MaxInlineSize`, `-XX:FreqInlineSize`)

## 루프 펼치기
컴파일러는 매번 순회할 때마다 루프 처음으로 되돌아가는 횟수를 줄이기 위해 루프를 펼칠 수 있다

루프 처음으로 돌아가는 작업을 **백 브랜치** 라고 하며, 이 비용은 높다
➡️ 따라서 루프 바디가 짧으면 백 브랜치 비용은 상대적으로 높다

따라서 핫스팟은 아래 기준에 따라 루프 펼치기 여부를 결정한다
* 루프 카운터 변수 유형(대부분 객체가 아닌 int나 long형을 사용)
* 루프 보폭(한번 루프돌때마다 루프 카운터 값이 얼마나 바뀌는가)
* 루프 내부 탈출 지점 개수(return or break)

> 루프 펼치기는 핫스팟 버전별로 로직이 상이하고, 아키텍처마다 많이 다르다

## 탈출 분석
* 메서드 내부에서 할당된 객체를 메서드 범위 밖에서 바라볼 수 있는지 알아보는 용도로 쓰인다
* **탈출 분석 최적화는 반드시 인라이닝을 수행한 이후에 시도한다**

탈출할 객체의 세가지 유형
* `hotspot/src/share/vm/opto/escape.hpp`
* NoEscape: 객체가 메서드, 스레드 탈출을 하지 않고 호출 인수로 전달되지 않는다
* ArgEscape: 객체가 메서드, 스레드 탈출은 하지 않지만 호출 인수로 전달된다
* GlobalEscape: 객체가 메서드, 스레드를 탈출한다

**NoEscape 예시**
``` java
for (int i = 0; i < 1_000_000; i++){
	MyObj foo = new MyObj(i); // foo 메서드 탈출 안함
    sum += foo.bar();
}
```

**ArgEscape 예시**
``` java
for (int i = 0; i < 1_000_000; i++){
	MyObj foo = new MyObj(i); 
    sum += extBar(foo); // foo 는 메서드의 인수로 전달됨
}
```

**GlobalEscape 예시**
(by chatgpt)
``` java
class MyGlobal {
    static MyObj globalObj = null; // 전역 변수

    public static void setGlobalObj(MyObj obj) {
        globalObj = obj; // 전달받은 객체를 전역 변수에 저장
    }
}

public class EscapeExample {
    public static void main(String[] args) {
        long sum = 0;
        for (int i = 0; i < 1_000_000; i++) {
            MyObj foo = new MyObj(i);
            MyGlobal.setGlobalObj(foo); // foo 객체가 Global Escape
            sum += foo.bar();
        }
    }
}
```

### 탈출 분석 하는 이유
* 루프 안에서 객체를 새로 만들면 단명 객체가 끊임없이 양산되고, 그러면 마이너 GC 이벤트가 자주 발생할 것이다
* 만약 할당된 객체가 `NoEscape`라면, **스칼라 치환** 이라는 최적화를 적용해 지역변수였던 것 처럼 스칼라 값으로 바꾼다
* 힙 할당을 막을 수 있는지 추론할 수 있다

## 단형성 디스패치
``` java
class Animal {
    void speak() {
        System.out.println("Some generic noise");
    }
}

class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    @Override
    void speak() {
        System.out.println("Meow");
    }
}
```
``` java
public void makeItSpeak(Animal animal) {
    animal.speak();
}
```

요런 코드가 있다고 하자.

`makeItSpeak` 메서드를 호출한다고 하면, Input에는 어떤게 들어올까?
Animal 타입의 구체적인 객체인 Dog or Cat이 들어올 것 이다
(➡️ 각 호출부 마다 딱 한가지 런타임 타입이 수신자 객체 타입이 된다)

즉, 어떤 객체에 있는 메서드를 호출할 때, 그 메서드를 최초로 호출한 객체의 런타임 타입(Dog or Cat)을 알아내면 그 **이후 모든 호출도 동일한 타입일 가능성이 크다**

그렇다면 해당 호출부 메서드 호출을 최적화할 수 있다
vtable에서 메서드를 찾을 필요가 없고, 항상 타입이 같다면 호출 대상을 계산하여 `invokevirtual` 명령어를 가드 후 컴파일드 메서드 바디로 분기하는 코드로 치환할 수 있다

**좀 더 자세하게!💡**

객체의 메서드 호출 시, JVM은 객체의 가상 메서드 테이블인 `vtable`을 조회하여 실제 실행할 메서드를 찾는다.

이 과정은 `invokevirtual` JVM 명령어를 통해 이루어 진다.
`invokevirtual`는 객체의 런타임 타입을 확인하고, 해당 타입의 메서드 구현을 `vtable`에서 찾는다.

하지만 동일한 메서드를 반복적으로 호출하고, 호출되는 객체의 타입이 변경되지 않는다면?
➡️ 매번 `vtable` 을 조회하지 않고 호출될 메서드의 주소를 캐시해두고 캐시된 주소를 사용해 메서드를 직접 호출할 수 있다

## 인트린직
* JIT 서브시스템이 동적 생성하기 이전에 JVM이 이미 알고 있는, 고도로 튜닝된 native 메서드 구현체를 가리키는 용어이다
* 주로 OS나 CPU 아키텍처의 특정 기능을 응용하는, **성능이 필수적인 코어 메서드에 쓰인다**
* 인트린직은 C1/C2, JIT Compiler 및 인터프리터에도 구현이 가능하다
* OpenJDK 핫스팟 소스 코드에서 확장자가 `.ad(architect dependent` 인 파일이 인트린직 템플릿이다
* Java 9부터 메서드 앞에 `@HotSpotIntrinsicCandidate` annotation을 붙여 인트린직을 사용할 수 있다

**자주 사용되는 인트린직**
* `java.lang.System.arraycopy()`: CPU 벡터 지원 기능으로 배열을 빨리 복사한다
* `java.lang.System.currentTimeMillis()`: 대부분 OS가 제공하는 구현체가 빠르다
* `java.lang.Math.min()`: 일부 CPU에서 분기 없이 연산이 가능하다

## 온-스택 치환
* 컴파일을 일으킬 정도로 호출 빈도가 높지는 않지만 메서드 내부에 핫 루프가 포함된 경우가 있다(java의 `main()`)
* 핫스팟은 이런 코드를 온-스택 치환(OSR)을 이용해 핫 루프만 native code로 치환 한다

# 세이브포인트
> **세이브포인트(Safepoint)**
* 정의: 세이브포인트는 JVM이 안전하게 시스템 작업을 수행할 수 있는 지점을 의미합니다. 이 지점에서는 모든 스레드가 일정한 상태에 있으며, 가비지 컬렉션과 같은 작업을 안전하게 수행할 수 있습니다. 즉, 스레드가 실행 중인 코드의 특정 지점에서만 가비지 컬렉터나 다른 시스템 작업이 실행될 수 있도록 합니다.
* 용도: 세이브포인트는 가비지 컬렉션, 코드의 핫스팟 컴파일, 스레드 덤프 생성, 클래스 리로딩 등 다양한 시스템 작업을 안전하게 수행하기 위해 사용됩니다.

**세이브 포인트에 걸리는 경우**
* 메서드를 역최적화
* 힙 덤프를 생성
* 바이어스 락을 취소
* 클래스 재정의

* 세이브 포인트 체크(세이브포인트에 도달했는지 확인하는 과정) 발급은 JIT 컴파일러가 담당한다
* 핫스팟에서는 다음 지점에 세이브포인트 체크 코드를 넣는다
  * 루프 백 브랜치 지점
  * 메서드 반환 지점

# 추가
메서드를 작게 하면 좋은점
* 인라이닝 가짓수가 늘어난다
* 다양한 인라이닝 트리를 구축하여 핫 경로를 더욱 최적화할 여지가 생긴다
