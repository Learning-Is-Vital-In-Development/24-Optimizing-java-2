# Chapter2 - JVM 이야기
< JVM이 Java 코드를 실행하는 방법 >

* JVM은 Stack 기반의 해석 머신
![](https://velog.velcdn.com/images/ujin2021/post/4b0fdcc3-1c06-42de-85d1-9cbd59f2b3b3/image.png)

(출처: https://tcpschool.com/java/java_intro_programming)

## Java Classloading
Java Compiler(javac)를 통해 .class 파일로 변환한다.
변환된 class 파일을 JVM 메모리에 탑재 해야한다 ➡️ 이 과정은 컴파일 타임이 아니라 런타임에 일어난다 (Java의 동적 클래스 로딩 기능이라고 함)
이것을 Class load라고 하고, classloader가 해당 작업을 수행한다.
![](https://velog.velcdn.com/images/ujin2021/post/a46d458a-fc38-4246-89b0-24e0f951e6e9/image.png)

* 참고
  * https://www.baeldung.com/java-classloaders
  * https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader/


* JVM이 Class를 요청하면, classloader가 class를 찾는다
* 하위에서 못찾으면 상위에 요청한다
* 만약 가장 상위(부트스트랩)에서도 찾지 못하면 `ClassNotFoundException`이 발생한다.

## Bytecode
* javac(java compiler)는 Java파일(`.java`)을 bytecode(`.class`)로 변환한다
* Java bytecode는 JVM이 알아들을 수 있는 코드이다
* bytecode는 특정 아키테처에 특정하지 않은 중간표현형(IR)이다
  * JVM만 있다면 어떤 OS에서도 실행될 수 있다 (이식성이 좋다)
  
**.class file**
![](https://velog.velcdn.com/images/ujin2021/post/c90a8806-49b9-42fb-a29b-61bf346caa73/image.png)
(출처: https://blog.hexabrain.net/397)

**javap (역어셈블러)**
``` shell
$javap -c [Class명]
```

``` java
public class com.company.Foo {
  int data;
 
  public com.company.Foo(); // 클래스 Foo의 디폴트 생성자
    Code:
       // 여기서는 this를 피연산자 스택에 푸시한다.
       0: aload_0
       // 부모 클래스인 Object의 디폴트 생성자를 호출한다. (super())
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return
 
  public void doSomething();
    Code:
       0: return
}
```

## 핫스팟
* 프로그램의 런타임 동작을 분석하고, **성능에 가장 유리한 방향으로 최적화를 적용하는 가상머신**

프로그램이 성능을 최대로 내려면?
➡️ native 기능을 활용해 CPU에서 직접 프로그램을 실행시켜야 한다
(native code는 CPU가 이해할 수 있는 code)

> 따라서 핫스팟은 가장 자주 실행되는 bytecode를 native code로 컴파일 한다. (핫스팟의 기능1)
> ➡️ JIT(Just-In-Time, 적시, 그때그때 하는) 컴파일

핫스팟은 동적 인라이닝도 수행한다 (핫스팟의 기능2)

## JVM 메모리
* Java는 Garbage Collection 프로세스로 Heap 메모리를 자동 관리한다
* Garbage Collection(GC): 불필요한 메모리 회수 or 재사용
* GC가 실행되는 동안 다른 application은 중단된다

## JVM 모니터링, 툴링
* JMX
* Java agent
* JVMTI
* SA
