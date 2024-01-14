JVM이 자바 코드를 실행하는 방법을 소개한다.

### 인터프리팅과 클래스로딩 (Interpreting and Classloading)

JVM은 스택 기반의 해석 머신이다. register 는 없지만 일부 결과를 실행 스택에 보관하며, 이 스택의 가장 위에 쌓인 값을 가져와 계산한다.<br>
JVM interpreter 는 while문 안에 switch 문 으로 이해할 수 있다.


java HelloWorld 라는 명령을 실행하면
```
1. OS는 가상 머신 프로세스(java binary)를 구동한다.
2. 자바 가상 환경이 구성된다.
3. stack 머신이 초기화 된다.
4. HelloWorld 클래스가 실행된다.
   - 여기서 application 의 진입은 HelloWorld.class 의 main() method 이다.
```

제어권을 이 클래스로 넘기기 위해 가상 머신(이하 VM) 이 실행되기 전에 이 클래스를 load 해야한다.

#### classloading 매커니즘

1. Bootstrap ClassLoader
- 최소한의 필수 클래스(java.lang.Object, Class, ClassLoader..)만 로드
- Java 8 이전 : `${JAVA_HOME}/jre/lib/rt.jar` 및 기타 핵심 라이브러리와 같은 JDK의 내부 클래스를 로드한다.
- Java 9 이후 : `/re.jar`이 존재하지 않으며, `/lib` 내에 모듈화되어 포함됐다. 이제는 정확하게 ClassLoader 내 최상위 클래스들만 로드한다.

2. Extension ClassLoader(Platform Loader)
- 부트스트랩 클래스 로더를 부모로 갖는 클래스 로더로서, 확장 자바 클래스들을 로드한다.
- `java.ext.dirs` 환경 변수에 설정된 디렉토리의 클래스 파일을 로드하고, 이 값이 설정되어 있지 않은 경우 `${JAVA_HOME}/jre/lib/ext` 에 있는 클래스 파일을 로드한다.
- OS나 플랫폼에 native code 를 제공하고 기본 환경을 overriding 하는데 사용된다.

3. Application ClassLoader(System ClassLoader)
- 실행 시 지정한 Classpath에 있는 클래스 파일 혹은 jar에 속한 클래스들을 로드한다. (== 사용자가 생성한 .class 확장자 파일을 로드한다.)

ClassLoader는 모두 상속 관계이다.

`bootstrap (최상단 부모) → extension classload → application classloader (최하단 자식)`

새 클래스를 발견하여 찾지 못하면 한 단계씩 상위로 올려 lookup 을 하고, 끝까지 찾지 못한다면 ClassNotFoundException 을 일으킨다.<br>
`클래스 = 패키지명을 포함한 full class name + 자신을 로드한 classloader` 두가지 정보로 식별하여 이중 로딩 방지한다.

[Understanding the Java Class Loader Starting from Java 9](https://sergiomartinrubio.com/articles/understanding-the-java-class-loader-starting-from-java-9/) <br>
[Java9 Class ClassLoader](https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/java/lang/ClassLoader.html)


### 바이트코드 실행 (Executing Bytecode)


bytecode 는 컴퓨터의 아키텍쳐에 특정되지 않는 중간 표현형(Intermediate Representation, IR) 으로 JVM 이 지원되는 플랫폼 어디서든 실행할 수 있다.<br>
컴파일러가 생성한 클래스파일은 아래 형식을 준수한다.

|Component|Description|
|:--:|:--|
|Magic Number|0xCAFEBABE|
|클래스 파일 format version|class 파일의 major / minor 버젼|
|상수 풀(constant pool)|class 파일의 상수가 모여있는 위치|
|access flag|public,static,final 등 modifiers와 interface, annotation, enum 등 클래스 종류 |
|this class|현재 클래스의 이름|
|super class|부모 클래스의 이름|
|interface|현재 클래스가 implements 한 모든 interface|
|field|클래스에 들어가 있는 모든 필드|
|method|클래스에 들어가 있는 모든 methods|
|attribute|클래스의 속성(소스 파일 이름 등)|

```java
// HelloWorld.java
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
				System.out.println(3+4);
    }
}

// HelloWorld.class (bytecode)
public class HelloWorld {
  public HelloWorld();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #13                 // String Hello World
       5: invokevirtual #15                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}

```

### 핫스팟 JVM
zero cost(비용이 들지 않는) 추상화 사상에 근거하는 C++ 과 같은 언어는 AOP(Ahead-of-time: 사전 컴파일)를 하기 때문에 Platfotm(OS)에 최적화 되어 실행된다.

Java 는 이러한 zero-overhead 추상화 원칙을 동조하지 않는다.<br>
hotspot은 runtime 에 동작을 분석하고, 성능에 가장 유리한 방향으로 그때그때 최적화를 적용하는 가상머신이다.

### JIT 컴파일이란?(Just-in-Time)

자바 프로그램은 bytecoe interpreter 가 가상화한 stack 머신에서 실행하며 시작된다.<br>
CPU 를 추상화 한 구조이기 때문에 다른 platform 에서 class 파일을 문제 없이 실행할 수 있지만, 성능을 최대로 내려면 native 기능을 활용해 CPU 에서 직접 실행시켜야 한다.<br>
hotspot 은 자주 실행되는 코드 파트를 찾아내 JIT 컴파일을 수행한다.(interpreted bytecode -> native code)

> **JVM의 실행 엔진의 인터프리터** <br>
> 바로 기계어로 변환하는 컴파일러의 경우는 프로그램이 작성된 기계상에서 매우 효율적으로 실행된다. 그러나 이와 동시에 기계에 종속된다는 말이기도 하다.<br>
> 만약 JVM 내에서 컴파일러를 사용하여 바이트코드의 목적 파일을 생성한다면 이는 기계에 종속되는 파일이다.<br>
> 때문에 자바 인터프리터는 자바 컴파일러를 통해 생성된 바이트코드를 한 줄씩 읽어 기계어로 번역하고 실행한다.<br>
> 하지만 인터프리터는 컴파일러보다 실행 속도가 느리기 때문에 이를 해결하고자 JVM은 부분적으로 JIT 컴파일러를 사용하여 바이트코드를 컴파일하여 사용한다.

> JIT 컴파일러는 실행 시점에서는 인터프리터와 같이 기계어 코드를 생성하면서 해당 코드가 컴파일 대상이 되면 컴파일하고 그 코드를 캐싱한다.<br>
> JIT 컴파일은 코드가 실행되는 과정에 실시간으로 일어나며(Just-In-Time), 전체 코드의 필요한 부분만 변환한다.<br>
> 기계어로 변환된 코드는 캐시에 저장되기 때문에 재사용 시 컴파일을 다시 할 필요가 없다.

[[Java] JIT 컴파일러란?](https://hyeinisfree.tistory.com/26)


### JVM 메모리 관리 (JVM Memory Management)

GC는 JVM이 더 많은 메모리를 할당해야 할 때 불필요한 메모리를 회수하거나 재사용하는 불확정적 프로세스이다.<br>
GC가 실행되는 동안 다른 애플리케이션은 모두 멈춰야 하는데 이것은 애플리케이션의 부하가 늘어날 수록 무시할 수 없는 시간이 된다.

### Threading 과 자바 메모리 모델(JMM)
```java 
Thread t = new Thread(() -> { System.out.println("Hello World"); });
```

java 환경 자체가 JVM 처럼 multi-thread 기반이기 때문에 java 프로그램이 작동하는 방식은 더 복잡하고 성능 분석도 어렵다.<br>
JVM 구현체에서 java application thread 는 각각 하나의 전용 OS thread 에 대응한다.<br>

자바 멀티스레드 설계 원칙
- java process의 모든 thread는 GC되는 하나의 공용 heap 을 갖는다
- 한 thread 가 생성한 객체는 그 객체를 참조하는 다른 thread 가 access 할 수 있다.
- 기본적으로 객체는 변경이 가능하다.(final 이 붙은 클래스가 아니라면)
- exclusive lock은 코드가 동시 실행되는 중 객체가 손상되는 현상을 막을 수 있다

### JVM 구현체 종류

- openJDK
  - 자바 기준 구현체를 제공하는 특별한 open source 이다.
  - 오라클이 직접 주관/지원하며 java release 기준을 발표한다
- oracle Java
  - 가장 널리 알려진 구현체로 OpenJDK 기반이지만, 오라클의 상용 라이센스이다.

### JVM 모니터링과 툴링 

- JMX(Java management Extension)
- Java agent

### VisualVM

Attach machanism을 이용해 실행 프로세스를 실시간으로 모니터링 한다.

- 개요(java process 요약)
- 모니터: CPU, heap 사용량 등 JVM 을 고수준에서 원격 측정한 값 제공
- thread: 애플리케이션의 각 thread 를 시간대별로 표시 (thread dump 가능)
- sampler and profiler 
