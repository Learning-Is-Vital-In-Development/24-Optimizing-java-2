# Ch10. JIT 컴파일의 세계로

## jitwatch사용기

[AdoptOpenJDK/jitwatch](https://github.com/AdoptOpenJDK/jitwatch/releases)에서 설치 후 책에 나와있는 대로 `sandbox`를 실행시켜보았습니다.

java 21에서 작업했습니다.

[Oracle JDK Migration Guide: Tools and Components Removed and Deprecated in JDK 16](https://docs.oracle.com/en/java/javase/17/migrate/removed-tools-and-components.html#GUID-BBCF36FE-C892-4769-95CB-AB3FFC3A3B13)

에 따르면 JDK 9부터 도입된 `Unified Logging`으로 인해 JDK 16이후부터는 몇가지 플래그들이 삭제/통합되었습니다.

따라서 책에 적힌 플래그들 대신 아래 노트의 플래그를 사용해야합니다.

> [!NOTE]
>
> 책에서는 다음과 같은 플래그를 추가하라고 되어있지만
>
> -XX:+UnlockDiagnosticVMOptions -XX:+TraceClassLoading -XX:+LogCompilation
>
> java 8 이상에서 실행시키기 위해서는
>
> -XX:+UnlockDiagnosticVMOptions -Xlog:class+load=info -XX:+LogCompilation
>
> JITWatch가 클래스 모델을 구축하는 데 필요한 정보를 로깅합니다.

## JITWatch 뷰

소스코드, 바이트코드, 어셈블리로 어떻게 컴파일되는지 보여줍니다.

### 디버그 JVM과 hsdis

> [!TIP]
>
> hsdis (HotSpot disassembler)를 구축했다면
>
> `-XX:+PrintAssembly` 스위치(플래그)를 추가하여 HotSpot에서 `disassemble`된 네이티브 코드를 출력할 수 있습니다.
>
> 또한 `-XX:+PrintAssembly`플래그를 사용할 때는 추가적인 output이 제공되는
>
> `-XX:+DebugNonSafepoints` 플래그를 함께 사용하는 것이 권장됩니다.

이후 내용은 책에서..

