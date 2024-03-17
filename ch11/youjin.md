**"개발자" 측면에서 성능 튜닝 시 고려해야 할 부분은?**
➡️ 코드 설계
코드 설계는 외부 요인(네트워크 연결, I/O, DB) 다음으로 병목을 일으킬 가능성이 제일 크다

**코드의 기본 원칙**
➡️ 데이터를 application에 어떻게 저장할지 고려하기

**데이터를 저장하는 방법**
➡️ Collection, 도메인 객체

# Collection 최적화
대부분의 프로그래밍 언어 라이브러리는 최소한 두 가지 컨테이너를 제공한다
* 순차 컨테이너: 수치인덱스로 표기한 특정 위치에 객체를 저장한다
* 연관 컨테이너: 객체 자체를 이용해 컬렉션 내부에 저장할 위치를 결정한다

컨테이너에서 메서드가 정확히 작동하려면 저장할 객체가 호환성과 동등성 개념을 지니고 있어야 한다
➡️ 모든 객체가 반드시 `hashCode()`, `equals()` 메서드를 구현해야 한다

## List 최적화
Java에서 List: ArrayList, LinkedList

### ArrayList
* 고정 크기 배열에 기반한 리스트
* 배열의 최대 크기 만큼 원소를 추가할 수 있고, 배열이 꽉 차면 더 큰 배열을 새로 할당한 다음 기존값을 복사한다
  * Java의 경우, 보통 현재 배열의 크기보다 약 50% 더 큰 크기의 배열을 새로 할당한다
  
### LinkedList
* 동적으로 증가하는 리스트
* 이중 연결 리스트로 구현되어있기 때문에 리스트에 덧붙이는 작업은 항상 O(1)이다

### ArrayList vs LinkedList
두개의 선택은 데이터 접근/수정 패턴에 따라 다르다

**추가의 경우**
* ArrayList의 경우 특정 인덱스에 원소를 추가한다면 다른 원소들을 모두 한칸씩 우측으로 이동시켜야 한다
* Linked는 노드를 하나 생성한 경우, 두개의 레퍼런스만 세팅하면 되므로 간단하다
* 삭제도 비슷하다

**랜덤 access하는 경우**
* ArrayList를 사용한다
  * 모든 원소를 O(1) 시간만에 가져올 수 있기 때문이다
* LinkedList는 처음부터 인덱스 카운트만큼 원소를 방문해야 한다

## Map 최적화
### HashMap
### TreeMap
* TreeMap이 제공하는 get, put, containsKey, remove 메서드는 log(n) 작업 성능을 보장한다
* 대부분의 요건은 HashMap만으로 충분하지만, stream or lambda로 map일부를 처리해아 할 때 데이터 분할이 주특기인 TreeMap을 쓰는게 바람직 하다

## Set 최적화
* HashSet은 HashMap으로 구현되어있다
* TreeSet 역시 TreeMap을 활용한다

## 도메인 객체
* Order, User, ... 등의 객체

## 종료화
### finalize
* 자동으로 리소스를 해제/정리하는 해체기 메서드
* Java9부터 Object.finalize는 디프리케이트 되었다

### try-with-resources
* java7부터 추가되었다
* try 키워드 다음의 괄호 안에 리소스를 지정해서 생성할 수 있다

* 기존 코드
  * finally block에서 `close()`를 호출해야 한다
  
![](https://velog.velcdn.com/images/ujin2021/post/7ee75ce3-a6df-490f-a691-06c166741695/image.jpg)

* try 키워드 다음 괄호안에 리소스 지정

![](https://velog.velcdn.com/images/ujin2021/post/e6c5ae4c-2f15-478f-807f-2e331bccbb52/image.jpg)
