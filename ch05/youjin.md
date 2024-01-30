# 마이크로벤치마킹
## 🤨 마이크로벤치마킹이란?
* 컴퓨터 프로그램의 매우 작고 특정한 부분의 성능을 측정하는 것이다
* 특정한 함수, 알고리즘, 또는 작업 처리 방식의 성능을 측정하는 데 초점을 맞춘다

# 마이크로벤치마킹의 어려움
성능 측정 시, 공정하게 테스트를 해야한다.

* 시스템의 어느 한 곳 만 변경하고, 나머지 부분은 통제할 수 있어야 한다
* 시스템의 가변적인 부분은 테스트 간에 불변성을 유지해야 한다

➡️ 이것을 지키는 것은 매우 어렵다

## 원인. Java Runtime의 정교함
> **JVM과 Java Runtime**
* JVM은 자바 바이트코드를 실행하는 데 중점을 두는 가상 머신
* 자바 런타임은 JVM 뿐만 아니라 자바 애플리케이션을 실행하는 데 필요한 라이브러리와 기타 파일을 포함한 더 넓은 개념
  
➡️ 간단히 말해, JVM은 자바 런타임의 핵심 구성요소 중 하나이며, 자바 런타임은 JVM을 포함하여 자바 프로그램을 실행하는 데 필요한 모든 것을 제공한다

JVM은 개발자가 작성한 코드를 **자동 최적화** 한다.
최적화가 미치는 영향을 구체적으로 완전히 이해하고 설명하는 것은 어렵다

🤯 Java 코드는 JIT 컴파일러, 메모리 관리, 그밖의 Java Runtime이 제공하는 서브 시스템을 떼놓고 생각할 수 없다 <br>
🤯 테스트 실행시의 OS, HW, Runtime 조건의 작용또한 무시할 수 없다

### 고려해야 할 요인1. JVM Warm up
벤치마크 중, JVM이 메서드 호출을 최적화할 수 있다 <br>
➡️ 성능 캡처 전 JVM이 가동 준비를 마칠 수 있게 warm up 기간을 준다 <br>
💁🏻‍♀️ 벤치마크 대상 코드를 여러번 반복 실행한다

> 💡 힙 크기 조정하는 법 법: **-Xms, -Xmx** option
``` shell
java -Xms2048m -Xmx2048m -XX:+PrintCompilation [벤치마크 대상]
```

### 고려해야 할 요인2. Garbage Collection
GC가 일어날 가능성이 큰 시기에 캡쳐하지 않는다
(하지만 GC 수집은 불확정적이어서 어쩔 도리가 없다...)

> 💡 GC Log 보는 법: **-verbose:gc** option
``` shell
java -Xms2048m -Xmx2048m -verbose:gc [벤치마크 대상]
```

### 고려해야 할 요인3. 죽은 코드
테스트하려는 코드에서 생성된 결과를 실제로 사용하지 않는다

``` java
public class ClassicSort {
	private static final List<Integer> testData = new ArrayList<>();
	
	public static void main(String[] args) {
    	// testData List 생성
        
        double startTime = System.nanoTime();
        
        for (int i = 0; i < I; i++) {
        	List<Integer> copy = new ArrayList<>(testData);
            Collections.sort(copy)
        }
        
        double endTime = System.nanoTime();
        
        // 벤치마킹 코드 실행시간 출력
    }
}
```

위 코드에서 copy List를 만들고, sort했지만 그뒤에 sort를 가져다 쓰는 곳이 없기 때문에 **JIT 컴파일러가 이것을 죽은 코드 경로로 식별하고 벤치마킹 대상을 최적화 할 수 있다**


위에서 설명한 것 처럼 마이크로벤치마킹 시 신경써야 할 것이 많았다.
하지만 멀티스레드 코드라면? 더 어려워진다.

## 마이크로벤치마킹 해결법
1. 시스템 전체를 벤치마크 하자
* 저수준 수치는 수집하지 않거나 무시한다
* 수많은 개별 작용의 전체 결과는 더 큰 규모에서 유의미한 결과를 얻는다

2. 공통 프레임워크를 사용하자
* 위에서 말한 여러 고려사항들을 공통 프레임워크를 통해 해결한다
➡️ JMH

# Java 마이크로벤치마킹 Tool(JMH Framework)
위에서 설명한 것 처럼, 마이크로벤치마킹은 어렵지만 해야만 하는 상황이 있다면 마이크로벤치마킹 툴의 **업계 표준**인 JMH(Java Microbenchmark Harness)를 사용하자!

## 🤨 꼭 마이크로벤치마킹을 해야하는 상황은?
* 사용 범위가 넒은 범용 라이브러리 코드 개발 시
* OpenJDK 또는 다른 자바 플랫폼 구현체 개발 시
* 지연에 극도로 민감한 코드 개발 시

## JMH
> JMH는 Java를 비롯해 JVM을 타깃으로 하는 언어로 작성된 나노/마이크로/밀리/매크로 벤치마크를 제작, 실행, 분석하는 Java 도구이다

벤치마크 프레임워크는 어떤걸 벤치마킹 할 지 알 수 없으므로 동적이어야 한다 <br>
➡️ 리플렉션 사용시, 벤치마크 실행 경로에 복잡한 JVM 서브시스템이 끼어들게 된다 <br>
➡️ JMH는 벤치마크 코드에 annotation을 붙여 Java 소스를 추가 생성한다

### JHM의 기능
* 런타임에 죽은 코드를 제거하는 최적화를 못 하게 한다
* 반복되는 계산을 상수폴딩(컴파일 타임에 계산 가능한 표현식을 상수로 바꾸어 처리하는 최적화 과정) 하지 않는다
* 값을 읽거나 쓰는 행위가 현재 캐시 라인에 영향을 끼치는 잘못된 공유 현상을 방지한다
* 쓰기 장벽(병목 지점)으로 부터 보호한다

### JHM 사용 방법
https://github.com/openjdk/jmh
* 참고: https://ysjee141.github.io/blog/quality/java-benchmark/

# JVM 성능 통계
모든 측정은 어느 정도의 오차를 수반한다

## 오차 유형
### 랜덤 오차
* 측정 오차 또는 무관계 요인이 어떤 **상관관계 없이** 결과에 영향을 미친다
* 정밀도와 랜덤 오차는 반비례 관계
* 보통 정규 분포를 따른다

### 계통 오차
* 원인을 알 수 없는 요인이 **상관관계 있는** 형태로 측정에 영향을 미친다
* 정확도와 계통 오차는 반비례 관계

![](https://velog.velcdn.com/images/ujin2021/post/d5db7400-85fa-424a-bb8f-51630a2a7da9/image.png)
(출처: https://www.dailyvet.co.kr/news/practice/companion-animal/88473)

## 통계치 해석
"웹 애플리케이션 응답" 그래프이다
![](https://velog.velcdn.com/images/ujin2021/post/e4ab592a-b2b3-4f83-b522-5c3554df232e/image.jpg)

빨간색 구간의 경우, 404 Error(Client Error)일 것이다
➡️ 매핑되지 않은 URL 요청 시, 곧장 404를 반환하기 때문이다

파란색 구간의 경우, 서버 Error일 것이다(장시간 부하 및 타임아웃)

노란색 구간의 경우, 성공한 요청일 것이다

> 유의미한 하위 구성요소들로 분해하는 개념은 유용하다
