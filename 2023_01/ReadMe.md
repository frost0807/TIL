# 2023_01

<br>

## 목차

### [JAVA에서 Stream과 for문의 속도 차이](#arrow_forward-java에서-stream과-for문의-속도-차이)

### [JAVA에서 배열을 List로 변환하는 방법](#arrow_forward-java에서-배열을-list로-변환하는-방법)

### [ConcurrentHashMap이란?](#arrow_forward-concurrenthashmap이란)

<br>

---

## 1월 7일

### :arrow_forward: JAVA에서 Stream과 for문의 속도 차이

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

### :arrow_forward: JAVA에서 배열을 List로 변환하는 방법

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

<br>

## 1월 11일

### :arrow_forward: ConcurrentHashMap이란?

출처:https://pplenty.tistory.com/17, https://devlog-wjdrbs96.tistory.com/269

Java의 key-value형 자료구조는 대표적으로 Hashtable과 HashMap이 있다. Hashtable의 경우에는 메소드 전체에 synchronized 키워드가 존재해 Multi-Thread 환경에서 Thread-safe 하지만 객체마다 Lock을 하나씩 가지고 있어서 살짝 느리다는 단점이 있다.
HashMap의 경우에는 synchronized 키워드가 존재하지 않아 Map 인터페이스를 구현한 클래스 중에서 성능이 제일 좋지만 Multi-Thread 환경에서 사용할 수 없다.

이에 Multi-Thread 환경에서 Hashtable을 대체하기 위해 나온게 ConcurrentHashMap이다.

```
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {

    public V get(Object key) {}

    public boolean containsKey(Object key) { }

    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
}
```

그렇다면 ConcurrentHashMap이 동기화를 처리하는 방식은 어떻게 될까?

2가지 경우로 나눌 수 있는데, 버킷이 비어있는 경우는 해당 버킷에 Lock을 걸지 않고 새로운 노드를 버킷에 삽입한다. 그리고 버킷에 이미 노드가 있는 경우는 synchronized를 이용해 Lock을 걸고 작업을 한다.

#### ConcurrentHashMap의 생성자

ConcurrentHashMap 생성자의 파라미터는 3가지가 있다.

```
public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
```

- initalCapacity : 초기 용량을 결정한다. 구현은 지정된 부하 계수가 주어지면 이 많은 요소를 수용하기 위해 내부 크기 조정을 수행한다.
- loadFactor : HashMap 에서 사용되는 부하계수와 동일하지만, ConcurrentHashMap에서는 초기 테이블의 크기를 설정하기 위한 용도로만 쓰인다. HashMap에서는 이 값에 따라서 table이 resize되는 시점이 결정되지만, ConcurrentHashMap에서는 항상 0.75로 동작한다.
- concurrencyLevel : 초기 테이블 크기를 결정하는데 힌트로 사용된다.

#### 가변 배열 리사이징

HashMap의 경우 resize를 할 때 새로운 배열을 만들어 copy 하는 방식이지만, CocurrentHashMap은 기존 배열에서 새로운 배열로 버킷을 하나씩 이동하는 방식이다. resize는 doubling으로 진행된다.

#### 1.8 버전에서의 변경점

JAVA 1.5버전에 추가된 ConcurrentHashMap은 추가될 당시에는 영역(기본 16개)을 구분하여 영역별로 Lock을 거는 방식이었지만, 1.8버전에서 영역별이 아닌 버킷단위로 Lock을 거는 방식으로 바뀌었다.
