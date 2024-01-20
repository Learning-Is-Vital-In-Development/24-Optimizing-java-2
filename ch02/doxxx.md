# Ch2. JVM 이야기

JVM이 자바 코드를 실행하는 방법

## 2.1 인터프리팅과 클래스로딩

1. JVM 실행 과정

    - 명령어 실행: java HelloWorld 명령으로 자바 애플리케이션 실행 시작.
    - OS 구동: OS는 자바 가상 머신 프로세스(자바 바이너리)를 구동.
    - 가상 환경 초기화: JVM 구성 및 스택 머신 초기화.

2. 자바 클래스로딩 메커니즘

    - 부트스트랩 클래스로더: 부트스트랩 클래스로더가 자바 런타임 코어 클래스 로드. 필수 클래스만 로드.
    - 확장 클래스로더: 확장 클래스로더 생성, 부트스트랩 클래스로더를 부모로 설정. 특정 OS나 플랫폼에 네이티브 코드를 제공하고 환경을 오버라이드 가능.
    - 애플리케이션 클래스로더: 애플리케이션 클래스로더 생성, 지정된 클래스패스에서 유저 클래스 로드.

3. 클래스로딩 순서 및 특이사항

    - 디펜던시 로드: 프로그램 실행 중 처음 보는 새 클래스는 디펜던시에 로드. 클래스로더는 찾지 못한 클래스를 부모에게 룩업을 요청.
    - 클래스 식별: 클래스는 풀 클래스명과 로드한 클래스로더로 식별.

4. 메인 메서드 실행

    - 진입점 지정: 애플리케이션 클래스로더가 메인 메서드를 찾아 제어권을 해당 클래스로 넘김.
    - main() 메서드 실행: 실제 유저가 작성한 HelloWorld 클래스의 main() 메서드 실행.
5. 주의사항

    - ClassNotFoundException 처리: 클래스를 찾지 못한 클래스로더는 부모 클래스로더에게 룩업을 요청하고, 부트스트랩까지 찾지 못하면 ClassNotFoundException 예외 발생.
    - 중복 클래스로딩 주의: 동일한 클래스를 상이한 클래스로더가 두 번 로드할 수 있으므로 주의 필요.
    - 클래스 식별: 클래스는 풀 클래스명과 로드한 클래스로더로 식별되므로 빌드 프로세스에서 운영 환경과 동일한 클래스패스로 컴파일하는 것이 좋음.

## 2.2 바이트코드 실행

- 컴파일 단계: 자바 소스 코드를 javac 컴파일러를 사용하여`.class` 파일로 변환. 최적화는 거의 하지
  않아 `javap`와 같은 역 어셈블리툴로 해독 가능한 바이트코드 생성.
    - 바이트코드 특징:
        - 바이트코드는 특정 컴퓨터 아키텍처에 종속되지 않은 중간 표현형(IR)이며, JVM이 이를 실행.
        - 컴파일된 소프트웨어는 JVM 지원 플랫폼 어디서든 실행 가능하며 자바 언어에 대해 추상화 되어 있음.
- 클래스 파일 구조:
    - 클래스 파일은 0XCAFEBABE 매직 넘버로 시작.
    - 메이저/마이너 버전 숫자로 JVM 호환성 확인, `UnsupportedClassVersionError` 예외 발생.
    - 상수 풀에는 상숫값 등이 포함되며, 코드 실행 시 참조됨.
    - 액세스 플래그는 클래스의 수정자 결정. 예: Public, final, 인터페이스, 추상 클래스 등.
    - this 클래스, 슈퍼클래스, 인터페이스 엔트리는 타입 계층을 나타내는 인덱스로 표시.
    - 필드와 메서드는 시그니처와 수정자를 포함하는 구조로 정의.
    - 속성 세트는 메서드의 바이트코드 등을 나타내는데 사용.
- 런타임 검사 및 실행:
    - JVM은 클래스를 로드할 때 VM 명세서에 정의된 구조를 검사하여 올바른 형식인지 확인.
    - 런타임에는 상수 풀 등을 참조하며, 실행 중에 메모리에 배치되는 것이 아니라 상수 풀 테이블을
      참조하여 필요한 값을 얻음.

```Bash
javap -c HelloWorld
Compiled from "HelloWorld.java"
public class HelloWorld {
  public HelloWorld();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_0
       1: istore_1
       2: iload_1
       3: bipush        10
       5: if_icmpge     22
       8: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
      11: ldc           #13                 // String Hello, World!
      13: invokevirtual #15                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      16: iinc          1, 1
      19: goto          2
      22: return
}
```

위 명렁을 해석해보면 다음과 같다.

1. 기본 생성자

    - `aload_0`: `this` 의 레퍼런스를 stack에 로드한다
    - `invokespecial`: super 생성자를 호출하고 객체를 생성하는 method 를 실행한다.
        - default constructor를 override 하지 않았으므로 `java/lang/Object` 의 default constructor 매치

2. main method

    - `iconst_0`: 정수형 상수 0을 스택에 푸쉬한다
    - `istore_1`: 스택의 값을 로컬 변수 1에 저장합니다.
        - 0 부터 시작한다
        - instance method 에서 0번째 entry 는 this 이다
    - `iload_1`: 은 로컬 변수 1의 값을 스택에 로드합니다.
    - `if_icmpge`: 스택에 있는 값이 10보다 크거나 같으면 목표 지점(22번째 줄)으로 이동합니다.
    - `getstatic`: `System.out`의 정적 필드를 스택에 넣습니다.
    - `ldc`: 상수 풀에서 문자열 "Hello World"를 스택에 넣습니다.
    - `invokevirtual`: `PrintStream.println` 메서드를 호출하여 화면에 "Hello World"를 출력합니다.
    - `iinc(1, 1)`: 로컬 변수를 증가시킵니다.
    - `goto`:  주어진 목표 지점으로 이동합니다.
    - `if_icmpge` 가 성공할 때 까지 반복하다가 마지막 22번 `return` 을 실행하고 제어권이 넘어간다

## 마무리

이후 내용은 뒤의 장들에서 다뤄질 예정

