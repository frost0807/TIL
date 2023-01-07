# 2023_01

<br>

## 목차

### [JAVA에서 Stream과 for문의 속도 차이](#🔸-java에서-stream과-for문의-속도-차이)

### [JAVA에서 배열을 List로 변환하는 방법](#🔸-java에서-배열을-list로-변환하는-방법)

<br>

---

## 1월 7일

### 🔸 JAVA에서 Stream과 for문의 속도 차이

출처 : https://jypthemiracle.medium.com/java-stream-api%EB%8A%94-%EC%99%9C-for-loop%EB%B3%B4%EB%8B%A4-%EB%8A%90%EB%A6%B4%EA%B9%8C-50dec4b9974b

<br>

JAVA8에 추가된 문법 중 하나인 Stream은 기존 for문보다 가독성, 유지보수성 측면에서 뛰어나다.
성능 측면에서 보면 어떨까?

#### 테스트1

Array에 500,000개의 primitive type int 를 저장하고, 저장된 수 중 가장 큰 수를 찾는 함수를 만들었다.

| 종류       | 시간   |
| ---------- | ------ |
| for-loop   | 0.36ms |
| seq stream | 5.35ms |

for-loop의 경우가 seq stream보다 15배 빠른 결과가 나왔다. 왜일까?

</br>

Effective Java의 공저자인 Angelika Langer에 따르면, 40년이상 된 for-loop에 비해 10년도 채 안된 stream은 최적화 부분에서 아직 부족하기 때문이라고 한다.
그렇다면, 속도 측면에서 무조건 for문을 써야 하는 것일까?

또 다른 테스트를 보자

<br>

#### 테스트2

ArrayList에 500,000개의 Wrapper type Integer 를 저장하고, 저장된 수 중 가장 큰 수를 찾는 함수를 만들어 테스트했다.

| 종류       | 시간   |
| ---------- | ------ |
| for-loop   | 6.55ms |
| seq stream | 8.33ms |

1.27배의 속도 차이밖에 나지 않는다. 왜일까?

</br>

wrapper type 자료형의 경우에는 stack에서 직접 접근가능했던 기존 primitive type 자료형과 달리 stack에 저장된 변수에서 다시 heap 영역을 참조하는 과정을 거쳐야 하므로 이 과정에서 iteration cost가 높아지고 이에 컴파일러의 최적화 이점이 사라지기 때문이라고 한다.

위 두 테스트는 모두 순회비용이 계산비용보다 높은 것들이었다. 그렇다면 계산비용이 더 높은 경우에는 어떨까?

<br>

#### 테스트3

Array에 10,000개의 primitive type int 를, ArrayList에 10,000개의 wrapper type Integer를 저장하고, 배열을 순회하면서 각각의 원소에 SlowSin() 함수를 계산했다. SlowSin이란 파라미터로 넘겨지는 메소드에 대해 사인함수 값을 계산하고, 이에 대한 테일러 급수를 계산하는 함수라고 한다.

| 종류                          | 시간    |
| ----------------------------- | ------- |
| int[] for-loop                | 11.72ms |
| int[] seq stream              | 11.85ms |
| ArrayList<Integer> for-loop   | 11.84ms |
| ArrayList<Integer> seq stream | 11.85ms |

사실상 차이가 없다고 봐도 무방할 정도이다. 이에 우리는 순회비용과 계산비용의 합이 충분히 크다면 for-loop대신 stream 사용해도 무방하지만, 해당 구문에 대규모의 트래픽이 예상된다면 순회비용과 계산비용을 잘 계산해서 둘 중 어떤걸 적용할지 고려해보면 될 것 같다.

<br>

### 🔸 JAVA에서 배열을 List로 변환하는 방법

출처 : https://hianna.tistory.com/551

JAVA에서 배열을 리스트로 변환하는 방법은 3가지가 있다.<br>

1. Arrays.asList(E[] array)
2. new ArrayList<>(Arrays.asList(E[] array))
3. Collectors.toList()

#### 1. Arrays.asList()

파라미터에 배열을 넣어주면 되는데, 해당 메소드는 fixed-size의 원 배열의 list view를 리턴한다. 즉, Dynamic Array가 아닌 Array라는 뜻이고 변환된 List도 정해진 size 이상으로 값 변경이 불가능 하다는 것을 뜻한다. 게다가 얕은복사가 되었기 때문에 동기화 문제도 있다.

#### 2. new ArrayList<>(Arrays.asList)

ArrayList 객체를 생성해 깊은복사를 하는 방법이다. 동기화 문제도 없고, 본래의 Dynamic Array처럼 기존에 정해진 size를 초과해 값을 추가하더라도 resize 과정을 거쳐 예외가 발생하지 않는다.

#### 3. Collectors.toList()

JAVA 8버전이후부터는 Stream의 최종연산인 collect의 파라미터로 Collectors.toList()를 전달해 배열을 List로 변환할 수 있다.
