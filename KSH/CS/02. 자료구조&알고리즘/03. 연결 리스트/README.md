# 연결 리스트: 노드를 따라 찾고 연결을 바꾸기

## 1. 면접 핵심 Q&A

### Q1. 연결 리스트는 배열과 어떻게 다른가요?

**배열은 원소를 담는 칸이 나란히 놓이지만, 연결 리스트는 각 노드가 다음 노드를 가리켜 순서를 만듭니다.** 노드는 데이터와 연결에 필요한 참조를 담습니다. 따라서 앞 노드에서 다음 노드로 이동할 수 있지만, 배열처럼 인덱스만으로 임의의 노드를 바로 찾을 수는 없습니다.

### Q2. 단일 연결 리스트와 이중 연결 리스트는 무엇이 다른가요?

**단일 연결 리스트의 노드는 `next`를, 이중 연결 리스트의 노드는 `prev`와 `next`를 가집니다.** 이중 연결 리스트에서는 앞뒤로 이동하고, 이미 찾은 노드의 양옆 연결을 바꾸기 쉽습니다. 대신 노드마다 이전 노드를 가리키는 참조를 더 저장하고, 연결을 바꿀 때 양쪽을 맞춰야 합니다. Java의 `LinkedList`는 이중 연결 리스트입니다. [Java LinkedList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html)

### Q3. 연결 리스트의 중간 삽입·삭제는 왜 `O(1)`이라고 하나요?

**위치를 이미 알고 있다면 주변 노드의 참조 몇 개만 바꾸기 때문입니다.** 예를 들어 `A → B` 사이에 `X`를 넣을 때 `X`가 `B`를, `A`가 `X`를 가리키게 합니다. 다만 *몇 번째 위치*라는 인덱스만 받았다면 먼저 그 위치까지 이동해야 하므로 전체는 `O(N)`입니다.

### Q4. 인덱스 조회와 순회 비용은 어떻게 다른가요?

**인덱스로 한 노드를 찾을 때는 앞이나 뒤에서 참조를 따라가야 하므로 최악 `O(N)`이고, 현재 노드에서 다음 노드로 가는 일은 `O(1)`입니다.** 따라서 전체를 한 번 순회하면 `O(N)`이지만, 인덱스 `0`부터 `N−1`까지 매번 다시 조회하면 `O(N²)`이 될 수 있습니다. Java `LinkedList`는 인덱스에 가까운 양 끝 중 한쪽에서 탐색합니다. [Java LinkedList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html)

### Q5. Java에서 큐가 필요하면 `LinkedList`를 쓰면 되나요?

**양 끝에서 넣고 꺼내는 일반적인 큐라면 `ArrayDeque`를 먼저 고려할 수 있습니다.** 배열을 재사용하며 노드를 원소마다 만들지 않고, Java 문서도 큐로 사용할 때 대체로 `LinkedList`보다 빠를 것으로 설명합니다. `LinkedList`는 `List`의 인덱스·반복자 기능까지 필요할 때 검토할 수 있습니다. `ArrayDeque`는 `null`을 허용하지 않습니다. `LinkedList`는 허용하지만 큐의 `poll()`이 빈 큐를 `null`로 표시하므로 큐 원소로 `null`을 쓰는 것은 피합니다. 두 구현 모두 별도 동기화 없이 여러 스레드에서 함께 수정할 수 없습니다. [Java ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html), [Java LinkedList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html), [Java Queue 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Queue.html)

<details>
<summary>심화 Q&A: 단일 연결 리스트에서 노드만 알면 삭제도 O(1)인가요?</summary>

보통은 **삭제할 노드의 이전 노드**도 알아야 앞 노드의 `next`를 바꿀 수 있습니다. `A → B → C`에서 `B`만 가리키고 있다면 `A`의 위치를 바로 알 수 없습니다. 반면 이중 연결 리스트의 `B`에는 `prev`가 있으므로, `B`를 이미 가리키고 있다면 양옆 노드의 연결을 일정한 횟수로 바꿀 수 있습니다. 맨 앞·맨 뒤에서는 `head`·`tail`도 적절히 갱신해야 합니다.

</details>

---

## 2. 실무 사례

### `List`로 받은 처리 대상을 인덱스로 반복 조회한다면?

> 원리를 설명하는 **학습용 가상 사례**입니다. 특정 서비스의 장애나 측정 결과가 아닙니다.

**상황:** 백엔드에서 전달받은 처리 대상 목록을 순서대로 방문합니다. Java 메서드는 매개변수를 `List`로 받으므로 호출자가 `ArrayList`와 `LinkedList` 중 어느 구현을 전달했는지에 따라 인덱스 조회 비용이 달라집니다. Python 예시에서는 양 끝 접근에 적합한 `deque`를 인덱스로 순회했을 때의 같은 문제를 봅니다. 두 자료구조의 내부 구현이 같다는 뜻은 아닙니다. [Java RandomAccess 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/RandomAccess.html), [Python deque 문서](https://docs.python.org/3/library/collections.html#collections.deque)

**Java**

```java
List<String> targets = new LinkedList<>(List.of("A", "B", "C", "D"));
for (int i = 0; i < targets.size(); i++) {
    System.out.println(targets.get(i));
}
```

**Python**

```python
from collections import deque

targets = deque(["A", "B", "C", "D"])
for i in range(len(targets)):
    print(targets[i])
```

**원인:** Java `LinkedList.get(i)`는 요청할 때마다 가까운 끝에서 다시 노드를 찾습니다. 예를 들어 원소 8개를 차례로 조회할 때 `next` 또는 `prev`를 따라가는 횟수는 인덱스별로 `0, 1, 2, 3, 3, 2, 1, 0`번, 합계 **12번**입니다. 원소 수가 `N`으로 커지면 이런 탐색의 합계가 `N²`에 비례해 늘어납니다. 여기서 센 것은 **노드 사이의 이동 횟수**이며 출력 시간은 제외했습니다. Python `deque`도 양 끝의 인덱스 접근은 빠르지만 가운데로 갈수록 느려집니다. [OpenJDK LinkedList 구현](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/LinkedList.java), [Python deque 문서](https://docs.python.org/3/library/collections.html#collections.deque)

**개선:** 순서대로 읽기만 한다면 현재 위치에서 다음 원소로 이동하는 반복자를 사용합니다. 한 원소를 한 번씩 방문하므로 순회는 `O(N)`입니다.

**Java**

```java
List<String> targets = new LinkedList<>(List.of("A", "B", "C", "D"));
for (String target : targets) {
    System.out.println(target);
}
```

**Python**

```python
from collections import deque

targets = deque(["A", "B", "C", "D"])
for target in targets:
    print(target)
```

**판단 기준:** 인덱스로 자주 조회한다면 Java `ArrayList`나 Python `list`가 알맞습니다. 양 끝에서 넣고 꺼내는 작업이 주목적이면 Java `ArrayDeque`나 Python `deque`를 고려합니다. Java에서 `LinkedList`를 순회하며 요소를 삭제해야 한다면 반복자의 `remove()`를 사용해 현재 위치에서 삭제할 수 있습니다. [Java ArrayList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html), [Java ListIterator 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ListIterator.html)

---

## 3. Deep Dive: 노드는 어떻게 찾고 연결할까?

### 3-1. 떨어진 노드를 어떻게 한 줄로 만들까?

배열은 같은 크기의 저장 칸이 순서대로 놓여 인덱스로 칸을 찾습니다. 연결 리스트에서는 **첫 노드를 가리키는 `head`**부터 시작해 각 노드의 `next`를 따라갑니다. 여기서 참조는 다음 노드를 찾아갈 수 있는 정보이며, Java·Python의 참조를 화면에 보이는 메모리 주소 숫자와 동일시할 필요는 없습니다.

```text
head
 │
 ▼
[A | next] ──→ [B | next] ──→ [C | null]
   노드 A          노드 B          노드 C

논리적인 순서는 A → B → C입니다.
노드 객체들이 메모리에서 이 순서로 나란히 있다는 뜻은 아닙니다.
```

다음은 원소 세 개를 직접 연결하고 모두 출력하는 작은 코드입니다. Java의 `Node`와 Python의 `Node` 모두 데이터와 다음 노드의 참조를 담습니다.

**Java**

```java
class Node {
    String value;
    Node next;

    Node(String value) {
        this.value = value;
    }
}

Node a = new Node("A");
Node b = new Node("B");
Node c = new Node("C");
a.next = b;
b.next = c;
Node head = a;

for (Node current = head; current != null; current = current.next) {
    System.out.println(current.value);
}
```

**Python**

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

a = Node("A")
b = Node("B")
c = Node("C")
a.next = b
b.next = c
head = a

current = head
while current is not None:
    print(current.value)
    current = current.next
```

**작업 횟수:** 노드의 값을 방문하는 일을 한 번으로 세면 `A`, `B`, `C`에서 **3번** 방문합니다. `C`를 찾을 때는 `A.next`, `B.next`를 따라 **2번 이동**합니다. 노드가 `N`개라면 전체 순회는 `N`번 방문해 `O(N)`이고, 한 인덱스에 해당하는 노드를 처음부터 찾는 일도 최악 `O(N)`입니다. 노드 `N`개를 저장하는 공간은 `O(N)`, 순회 중 `current`가 추가로 쓰는 공간은 `O(1)`입니다.

**면접에서 정리하면:** “연결 리스트는 `head`에서 노드의 `next`를 따라 순서를 찾습니다. 그래서 전체 순회는 `O(N)`이고, 인덱스로 특정 노드를 찾는 데도 최악 `O(N)`이 듭니다.”

---

### 3-2. 단일 연결 리스트 중간에 `X`를 넣거나 빼려면?

앞에서 만든 `A → B → C`에서 **`A`를 이미 가리키고 있다**고 합시다. `A` 뒤에 `X`를 넣을 때는 먼저 `X.next`를 기존 `B`에 연결하고, 그다음 `A.next`를 `X`로 바꿉니다. 순서를 반대로 하면 `B`로 가는 기존 연결을 잃을 수 있습니다.

```text
삽입 전:  A ──→ B ──→ C

1단계:   X ──→ B             X.next = A.next
         A ──→ B ──→ C

2단계:   A ──→ X ──→ B ──→ C  A.next = X

삭제 후:  A ─────────→ B ──→ C  A.next = A.next.next
```

**Java — 앞 코드의 `a`, `b`, `c`를 사용**

```java
Node x = new Node("X");
x.next = a.next;
a.next = x;             // A → X → B → C

a.next = a.next.next;   // A → B → C: X를 목록에서 분리
```

**Python — 앞 코드의 `a`, `b`, `c`를 사용**

```python
x = Node("X")
x.next = a.next
a.next = x             # A → X → B → C

a.next = a.next.next   # A → B → C: X를 목록에서 분리
```

**작업 횟수:** `A`를 이미 알고 있을 때 삽입은 참조 대입 **2번**, `A` 다음 노드 삭제는 참조 대입 **1번**입니다. 원소 수와 관계없이 일정하므로 이 **연결 변경**은 `O(1)`입니다. 그러나 “인덱스 5만 뒤에 삽입”처럼 `A`의 위치를 먼저 찾아야 한다면 탐색 `O(N)`이 더해져 전체는 `O(N)`입니다. 위 코드의 `x` 변수는 분리된 노드를 계속 가리키므로, 목록에서 분리했다고 객체가 즉시 사라지는 것은 아닙니다.

**면접에서 정리하면:** “단일 연결 리스트는 이전 노드를 이미 알면 다음 연결을 바꿔 `O(1)`에 삽입·삭제할 수 있습니다. 위치를 찾아야 하면 탐색 때문에 전체는 `O(N)`입니다.”

---

### 3-3. 앞 노드로도 돌아가려면?

단일 연결 리스트의 `B`는 자신 앞에 누가 있는지 기록하지 않습니다. **이중 연결 리스트**는 노드에 `prev`를 더해 앞 노드로도 이동합니다. Java의 `LinkedList`가 이 구조를 사용하며, 인덱스 조회 시 `head`와 `tail` 중 가까운 쪽에서 출발합니다. [Java LinkedList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html), [OpenJDK LinkedList 구현](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/LinkedList.java)

```text
삽입 전:  A ⇄ B ⇄ C

X를 A와 B 사이에 연결:
         X.prev = A    X.next = B
         A.next = X    B.prev = X

삽입 후:  A ⇄ X ⇄ B ⇄ C

X를 목록에서 분리:
         A.next = B    B.prev = A

삭제 후:  A ⇄ B ⇄ C
```

아래 코드는 가운데 두 노드 `A`, `B`를 **이미 알고 있는 상황**을 보여줍니다. `DNode`는 양쪽 참조를 가집니다.

**Java**

```java
class DNode {
    String value;
    DNode prev;
    DNode next;

    DNode(String value) {
        this.value = value;
    }
}

DNode a = new DNode("A");
DNode b = new DNode("B");
a.next = b;
b.prev = a;

DNode x = new DNode("X");
x.prev = a;
x.next = b;
a.next = x;
b.prev = x;  // A ⇄ X ⇄ B

a.next = b;
b.prev = a;  // A ⇄ B
```

**Python**

```python
class DNode:
    def __init__(self, value):
        self.value = value
        self.prev = None
        self.next = None

a = DNode("A")
b = DNode("B")
a.next = b
b.prev = a

x = DNode("X")
x.prev = a
x.next = b
a.next = x
b.prev = x  # A ⇄ X ⇄ B

a.next = b
b.prev = a  # A ⇄ B
```

**작업 횟수:** 두 노드 사이 삽입은 참조 대입 **4번**, 목록에서 `X`를 분리할 때 양옆 연결 변경은 **2번**입니다. 위치를 이미 알면 둘 다 `O(1)`이지만, 인덱스로 위치를 찾아야 하면 전체는 `O(N)`입니다. 첫 노드와 마지막 노드를 바꿀 때는 없는 이웃 대신 `head`나 `tail`을 갱신합니다. `X` 자체를 다른 곳에서 계속 참조한다면 필요에 따라 `X.prev`와 `X.next`도 끊어야 합니다.

**면접에서 정리하면:** “이중 연결 리스트는 `prev` 덕분에 양방향으로 이동하고, 이미 찾은 노드의 양옆 연결을 `O(1)`에 바꿉니다. 대신 노드마다 참조 하나를 더 저장하며, 인덱스 탐색은 여전히 `O(N)`입니다.”

---

### 3-4. 어떤 작업에 어떤 자료구조가 맞을까?

지금까지 본 비용은 **위치를 찾는 작업**과 **찾은 뒤 값을 옮기거나 연결을 바꾸는 작업**을 나눈 결과입니다. 이를 함께 보면 다음처럼 정리할 수 있습니다. 단일 연결 리스트의 끝 추가 비용은 `tail`을 별도로 보관하는지에 따라 달라집니다.

| 작업 | 동적 배열 (`ArrayList`) | 단일 연결 리스트 | 이중 연결 리스트 (`LinkedList`) |
|---|---|---|---|
| 인덱스로 조회 | `O(1)` | `O(N)` | `O(N)`; 양 끝 중 가까운 쪽부터 탐색 |
| 전체 순회 | `O(N)` | `O(N)` | `O(N)` |
| 맨 뒤에 추가 | 분할 상환 `O(1)` | `tail`이 있으면 `O(1)`, 없으면 `O(N)` | `tail`이 있으면 `O(1)` |
| 이미 찾은 중간 위치에 삽입·삭제 | 뒤 원소 이동으로 최악 `O(N)` | 이전 노드를 알면 연결 변경 `O(1)` | 관련 노드를 알면 연결 변경 `O(1)` |
| 인덱스로 중간 위치를 찾아 삽입·삭제 | 뒤 원소 이동으로 최악 `O(N)` | 탐색 때문에 `O(N)` | 탐색 때문에 `O(N)` |

연결 리스트는 노드마다 데이터 외의 참조가 필요하고, 다음 노드를 읽으려면 현재 노드의 참조를 먼저 확인해야 합니다. 반면 동적 배열은 저장 칸이 나란히 있어 순차 접근에서 공간 지역성을 활용하기 쉽습니다. **같은 빅오라도 실제 실행 시간은 객체 배치·입력·구현에 따라 달라지므로 측정 없이 우열을 단정하지 않습니다.** [Java ArrayList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html), [Java LinkedList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html)

<details>
<summary>이해 보충: Java의 LinkedList에서 노드를 직접 다룰 수 있을까?</summary>

표준 `LinkedList`의 내부 `Node`는 공개되지 않습니다. `list.add(index, value)`나 `list.remove(index)`는 먼저 인덱스에 해당하는 노드를 찾으므로 중간 위치에서 `O(N)`이 듭니다. 이미 순회 중이라면 `ListIterator`의 `add()`와 `remove()`로 현재 위치를 활용할 수 있습니다. 단, `listIterator(index)`를 중간 인덱스에서 새로 만들 때도 그 위치까지 이동해야 합니다. [OpenJDK LinkedList 구현](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/LinkedList.java), [Java ListIterator 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ListIterator.html)

</details>

<details>
<summary>심화: 객체 크기와 캐시 효율을 숫자로 비교할 수 있을까?</summary>

노드마다 추가 참조와 객체 관리 비용이 든다는 방향은 설명할 수 있습니다. 그러나 **노드당 바이트 수**는 JVM의 참조 크기·객체 헤더·정렬·버전·옵션에 따라 달라지고, 동적 배열에도 배열 자체와 남는 용량의 비용이 있습니다. 특정 조건 없이 “노드 24바이트 대 배열 칸 4바이트”를 모든 Java 실행 환경에 적용할 수는 없습니다.

노드가 반드시 서로 멀리 떨어져 있거나 매번 캐시 미스가 나는 것도 아닙니다. 다만 다음 노드의 위치를 알려면 현재 노드의 `next`를 읽어야 하므로, 연속된 저장 칸을 읽는 배열보다 메모리 접근의 공간 지역성을 활용하기 어려울 수 있습니다. [앞서 배운 캐시와 공간 지역성](<../04. 메모리 계층 구조와 캐시 메모리/README.md>)을 연결해 이해하면 됩니다.

</details>
