# 스택, 큐, 덱: 어디에서 넣고 꺼낼까?

## 1. 면접 핵심 Q&A

### Q1. 스택과 큐의 차이는 무엇인가요?

**스택은 나중에 넣은 원소를 먼저 꺼내는 LIFO, 큐는 먼저 넣은 원소를 먼저 꺼내는 FIFO 구조입니다.** `A`, `B`, `C` 순서로 넣으면 스택에서는 `C`, `B`, `A`, 큐에서는 `A`, `B`, `C` 순서로 나옵니다. 스택은 최근 작업부터 되돌릴 때, 큐는 들어온 순서대로 처리 대상을 꺼낼 때 사용합니다.

### Q2. 덱은 무엇이며, 스택·큐와 어떤 관계인가요?

**덱은 앞과 뒤 양쪽에서 삽입·삭제할 수 있는 자료구조입니다.** 한쪽 끝에서만 넣고 꺼내면 스택이고, 뒤로 넣고 앞으로 꺼내면 큐입니다. 덱 자체가 LIFO나 FIFO 하나를 강제하는 것은 아니며, 어떤 연산을 사용하는지가 순서를 결정합니다. [Java Deque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html)

### Q3. 스택·큐는 배열이나 연결 리스트와 다른 종류의 개념인가요?

**스택·큐는 사용할 연산과 순서를 정하고, 배열·연결 리스트는 이를 구현하는 저장 구조가 될 수 있습니다.** 예를 들어 스택의 꼭대기를 동적 배열의 끝으로 잡거나 연결 리스트의 맨 앞으로 잡을 수 있습니다. 이렇게 내부 구현보다 제공하는 동작에 초점을 맞춘 정의를 **추상 자료형(ADT)**이라고 합니다.

### Q4. 배열로 만든 큐는 앞에서 꺼낼 때마다 원소를 옮겨야 하나요?

**그렇지 않습니다. 맨 앞 위치를 따로 기록하면 원소를 옮기지 않고 그 위치만 갱신할 수 있습니다.** 원형 큐는 인덱스가 배열 끝에 도달하면 처음으로 돌아가 앞에서 비워 둔 칸도 재사용합니다. 다만 비었거나 가득 찬 상태를 구분해야 하며, 가득 찼을 때 거절할지 확장할지도 정해야 합니다.

### Q5. 스택·큐·덱의 삽입·삭제는 모두 O(1)인가요?

**입출력 규칙만으로 복잡도가 정해지지는 않고 구현을 봐야 합니다.** 원소 이동이 없는 원형 배열의 양 끝 조작은 `O(1)`이지만, 동적 배열을 확장하는 한 번은 `O(N)`일 수 있습니다. Java `ArrayDeque`의 양 끝 추가는 여러 번의 비용을 나눠 보면 분할 상환 `O(1)`이고, 특정 값을 찾아 삭제하는 연산은 최악 `O(N)`입니다. [Java ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html)

### Q6. Java에서 스택으로 Stack보다 ArrayDeque를 먼저 고려하는 이유는 무엇인가요?

**공식 문서가 `Stack`보다 `Deque`와 그 구현체 사용을 권장하기 때문입니다.** `Stack`은 `Vector`를 상속해 인덱스 삽입 같은 스택 밖의 기능도 노출하고, 동기화된 연산을 포함합니다. 별도 동기화가 필요 없는 상황에서는 `ArrayDeque`를 고려할 수 있지만, 여러 스레드가 함께 수정한다면 동시성 요구에 맞는 구현이나 동기화가 필요합니다. [Java Stack 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Stack.html), [Java ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html)

<details>
<summary>심화 Q&A: 큐가 FIFO이면 작업 완료 순서도 같나요?</summary>

FIFO는 **큐에서 꺼내는 순서**에 대한 규칙입니다. 여러 소비자가 작업을 병렬로 처리하면 먼저 꺼낸 작업이 오래 걸려 나중에 끝날 수 있습니다. 완료 순서까지 필요하다면 순차 처리나 결과 순서 조정 같은 별도 설계가 필요합니다.

또한 Java의 `Queue` 인터페이스가 모든 구현에 FIFO를 강제하는 것은 아닙니다. 예를 들어 우선순위 큐는 우선순위에 따라 원소를 꺼냅니다. 이 문서의 기본 큐 설명은 **FIFO 큐**를 대상으로 합니다. [Java Queue 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Queue.html)

</details>


---


## 2. 실무 사례

> 아래는 원리를 설명하는 **학습용 가상 사례**입니다. 특정 서비스의 장애나 사용자의 프로젝트 경험을 뜻하지 않습니다.

### 2-1. 최근 편집부터 취소하려면?

**상황:** 편집기에서 작업 `A`, `B`, `C`를 실행한 뒤 실행 취소를 두 번 누릅니다. 사용자는 가장 최근 작업인 `C`, 그다음 `B`가 취소되기를 기대합니다.

**이유:** 나중 작업이 앞선 작업의 결과에 이어서 수행됐으므로 최근 작업부터 되돌릴 필요가 있습니다. 실행한 작업의 취소 정보를 스택에 쌓으면 필요한 순서대로 꺼낼 수 있습니다.

```text
실행 순서: A → B → C

취소 정보: [A, B, C] ← 꼭대기
첫 취소:   [A, B]    → C의 취소 정보를 꺼내 적용
다음 취소: [A]       → B의 취소 정보를 꺼내 적용
```

**판단 기준:** 스택은 취소할 대상을 고르는 순서를 제공합니다. 실제 복원에는 변경 전 값이나 반대 동작 같은 정보도 저장해야 합니다. 다시 실행 기능까지 필요하면 취소한 작업을 별도 스택에 보관하는 방식으로 확장할 수 있습니다.

### 2-2. 알림 발송이 느려지면 작업 큐에는 무슨 일이 생길까?

**상황:** 요청을 받는 쪽은 알림 작업을 큐에 넣고, 작업을 처리하는 쪽은 큐에서 꺼내 외부 API를 호출합니다. 각각을 **생산자**, **소비자**라고 합니다. 큐는 두 쪽의 일시적인 속도 차이를 흡수하는 대기 공간, 즉 **버퍼** 역할을 합니다.

```text
생산자                         소비자 전체
초당 5개 제출 → [대기 작업 큐] → 초당 2개 처리
                    ↑
            같은 속도가 유지되면
            대기 작업은 초당 3개 증가
```

**원인:** 빈 큐에서 시작하고 위 속도가 계속 유지되며 취소·유실이 없다고 가정하면, 10초 동안 들어온 50개 중 20개를 처리해 30개가 남습니다. 큐는 작업을 기다리게 할 뿐 소비자의 처리 능력을 늘리지 않습니다. 이런 상태가 지속되면 대기 시간과 메모리 사용량이 늘어납니다.

**개선 또는 판단 기준:** 큐의 최대 작업 수를 정하고, 가득 찼을 때 생산자를 기다리게 할지, 새 작업을 거절할지 결정합니다. 소비 상태에 맞춰 생산 속도를 늦추는 제어를 **배압(backpressure)**이라고 합니다. 허용 대기 시간, 작업 한 개의 메모리, 처리 속도를 함께 보고 용량을 정해야 하며, 작업 수 제한만으로 전체 서버의 메모리 문제가 해결되지는 않습니다.

Java `ArrayBlockingQueue`와 Python `queue.Queue(maxsize=...)`는 스레드 사이에서 사용할 수 있는 용량 제한 큐입니다. 반면 Java `ArrayDeque`는 별도 동기화 없이 여러 스레드가 함께 수정하는 작업 대기열로 사용하면 안 됩니다. [Java ArrayBlockingQueue 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ArrayBlockingQueue.html), [Python queue 문서](https://docs.python.org/3/library/queue.html), [Java ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html)

<details>
<summary>심화: Java 스레드 풀과 CallerRunsPolicy에서는 무엇을 더 확인할까?</summary>

`Executors.newFixedThreadPool(...)`은 고정된 수의 작업 스레드와 용량 제한 없는 작업 큐를 사용합니다. 작업 스레드 수가 고정됐다고 대기 작업 수도 제한되는 것은 아닙니다. 유입이 처리를 계속 앞서면 메모리 고갈 위험이 있습니다. [Executors 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Executors.html#newFixedThreadPool(int))

`ThreadPoolExecutor`에 제한 큐를 지정하면 일반적으로 **핵심 스레드 수까지 생성 → 큐에 적재 → 큐가 꽉 차면 최대 스레드 수까지 추가 생성 → 더 받을 수 없으면 거절 정책 적용** 순서로 동작합니다. 따라서 큐가 꽉 찼다는 이유만으로 즉시 `CallerRunsPolicy`가 실행되는 것은 아닙니다.

`CallerRunsPolicy`는 포화 상태에서 작업을 제출한 스레드가 직접 작업을 수행하게 합니다. 제출 속도를 늦출 수 있지만, 웹 요청 스레드가 호출했다면 그 스레드가 외부 API 처리까지 맡아 응답이 늦어질 수 있습니다. 실행기가 종료된 상태에서는 작업을 버린다는 점도 확인해야 합니다. [ThreadPoolExecutor 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)

Python의 `queue.Queue`는 큐이며 스레드 풀 자체가 아닙니다. 생산자의 `put` 대기나 시간 제한, 소비자의 처리 방식을 조합해서 제어하므로 Java의 거절 정책과 일대일로 대응시키지 않습니다. [Python queue 문서](https://docs.python.org/3/library/queue.html)

</details>


---


## 3. Deep Dive: 입출력 규칙과 저장 구조를 연결하기

### 3-1. 같은 원소도 꺼내는 규칙에 따라 달라진다

`A`, `B`, `C`를 저장했다고 해 봅시다. 최근 작업을 되돌릴 때는 `C`가 먼저 필요하지만, 접수된 순서대로 처리할 때는 `A`가 먼저 필요합니다. 그래서 필요한 순서를 연산 규칙으로 정합니다.

- **스택(Stack):** 같은 끝에서 넣고 꺼냅니다. 그 끝을 **꼭대기(top)** 라고 합니다.
- **큐(Queue):** 뒤(rear)로 넣고 앞(front)으로 꺼냅니다.
- **덱(Deque):** 앞과 뒤 양쪽에서 넣고 꺼낼 수 있습니다.

```text
스택:   [A, B, C] ← 이 끝에서 삽입·삭제       꺼내기: C → B → A
큐:  ← [A, B, C] ← 뒤에서 삽입               꺼내기: A → B → C
       앞에서 삭제
덱:  ↔ [A, B, C] ↔                          양 끝 모두 삽입·삭제
```

위 그림은 **원소의 논리적인 순서**를 보여주며 메모리 배치를 뜻하지 않습니다. 같은 스택도 배열의 끝에서 넣고 꺼내거나, 연결 리스트의 맨 앞에서 연결을 바꿔 구현할 수 있습니다. **입출력 규칙과 실제 저장 방식은 구분해서 생각해야 합니다.**

---

### 3-2. 스택: 가장 최근에 넣은 원소부터 꺼내기

이번 예시에서는 Java와 Python 모두 **오른쪽 끝을 꼭대기**로 사용합니다. 하나를 꺼내면 바로 그 아래 원소가 다음 꼭대기가 됩니다.

**Java**

```java
java.util.Deque<String> stack = new java.util.ArrayDeque<>();
for (String value : new String[]{"A", "B", "C"}) {
    stack.addLast(value);
}
System.out.println(stack.peekLast()); // C: 보기만 하고 남겨 둠
while (!stack.isEmpty()) {
    System.out.println(stack.removeLast()); // C, B, A 순서
}
```

**Python**

```python
from collections import deque

stack = deque()
for value in ["A", "B", "C"]:
    stack.append(value)
print(stack[-1])  # C: 보기만 하고 남겨 둠
while stack:
    print(stack.pop())  # C, B, A 순서
```

삽입 반복문은 매번 오른쪽 끝에 원소 하나를 추가합니다. 삭제 반복문의 첫 회에는 `C`를 꺼내 `[A, B]`가 되고, 다음에는 `B`, 마지막에는 `A`를 꺼냅니다. 꼭대기를 보는 `peek`는 삭제와 다릅니다.

-  **작업 횟수:** <br> 
삽입·삭제 연산의 호출을 각각 한 번으로 세면 원소 3개는 삽입 3번과 삭제 3번, 총 6번입니다. 8개면 16번, `N`개면 `2N`번입니다. 꼭대기 확인 1번과 출력 시간은 이 집계에서 제외했습니다.

- **복잡도:** <br>
위 구현의 양 끝 연산 비용을 고려하면 전체 처리 시간은 **`O(N)`** 입니다. <u> 모두 넣은 직후 원소 `N`개를 동시에 보관하므로 자료구조에 필요한 공간은 `O(N)`입니다.</u> [Java ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html), [Python deque 문서](https://docs.python.org/3/library/collections.html#collections.deque)

**면접에서 정리하면:** “스택은 같은 끝에서 넣고 꺼내므로 마지막에 넣은 원소가 먼저 나옵니다. 원소를 한 번씩 넣고 꺼내면 전체 연산 수는 입력 크기에 비례합니다.”

<details>
<summary>이해 보충: Java의 push·pop과 Python의 list를 사용하면?</summary>

Java `Deque.push()`는 `addFirst()`, `pop()`은 `removeFirst()`, `peek()`는 `peekFirst()`에 대응합니다. 즉 **왼쪽 끝을 꼭대기로 사용하는 방식**입니다. 위 예시처럼 오른쪽 끝을 써도 스택이지만, `addLast()`로 넣은 뒤 `pop()`으로 꺼내면 반대쪽 끝에서 나오므로 LIFO가 되지 않습니다. [Java Deque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html)

Python에서는 `list`의 `append()`와 `pop()`으로도 스택을 구현할 수 있습니다. 이 문서에서는 스택·큐·덱의 입출력 위치를 같은 자료구조로 비교하기 위해 `deque`를 사용합니다. [Python 자료구조 튜토리얼](https://docs.python.org/3/tutorial/datastructures.html#using-lists-as-stacks)

</details>

---

### 3-3. 큐: 먼저 들어온 원소부터 꺼내기

이번에는 같은 순서로 넣되, **꺼내는 쪽을 왼쪽 끝으로 바꿉니다.** 스택과 입력은 같지만 결과가 달라집니다.

**Java**

```java
java.util.Deque<String> queue = new java.util.ArrayDeque<>();
for (String value : new String[]{"A", "B", "C"}) {
    queue.addLast(value);
}
while (!queue.isEmpty()) {
    System.out.println(queue.removeFirst()); // A, B, C 순서
}
```

**Python**

```python
from collections import deque

queue = deque()
for value in ["A", "B", "C"]:
    queue.append(value)
while queue:
    print(queue.popleft())  # A, B, C 순서
```

삭제 반복문의 첫 회에는 `A`를 꺼내 `[B, C]`가 되고, 다음에는 `B`, 마지막에는 `C`를 꺼냅니다. 먼저 들어온 원소가 앞에 남아 있으므로 먼저 나갑니다.

- **작업 횟수와 복잡도:** <br>
원소 3개는 삽입 3번과 삭제 3번, 8개는 총 16번입니다. `N`개를 넣고 모두 꺼내는 작업은 `2N`번이며, 위 구현에서는 전체 `O(N)` 시간과 최대 `O(N)` 저장 공간이 필요합니다. 출력 시간은 제외합니다. [Java ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html), [Python deque 문서](https://docs.python.org/3/library/collections.html#collections.deque)

**면접에서 정리하면:** “FIFO 큐는 뒤로 넣고 앞으로 꺼내 접수 순서를 유지합니다. 다만 배열 기반 목록의 첫 원소를 매번 지우는 방식은 이동 비용이 있으므로, 큐에 맞는 구현을 선택해야 합니다.”

그렇다면 Java `ArrayDeque`는 배열을 쓰면서도 어떻게 앞의 원소를 빠르게 꺼낼까?

---

### 3-4. 원형 큐: 원소 대신 인덱스를 움직이기

배열의 첫 칸이 언제나 큐의 맨 앞이어야 한다면 `A`를 꺼낸 뒤 `B`, `C`를 앞으로 옮겨야 합니다. 하지만 **현재 맨 앞이 몇 번 칸인지** 기록하면 옮길 필요가 없습니다. 그 위치를 `head`, 다음에 넣을 위치를 `tail`이라고 하겠습니다.

여기서는 **용량 4칸, 현재 원소 수 `size`를 별도로 저장하는 고정 크기 큐**를 가정합니다. 표준 라이브러리 구현을 그대로 옮긴 것이 아니라 원리를 보여주는 예시입니다. `·`는 빈칸입니다.

```text
                           인덱스:  0   1   2   3

① A, B, C가 들어 있음             [A] [B] [C] [·]
   head=0, tail=3, size=3           논리적 순서: A → B → C

② A를 꺼내고 head를 이동          [·] [B] [C] [·]
   head=1, tail=3, size=2           논리적 순서: B → C
   B와 C는 원래 칸에 그대로 있음

③ D를 3번 칸에 넣음               [·] [B] [C] [D]
   head=1, tail=0, size=3           논리적 순서: B → C → D
   tail은 배열 끝을 지나 0으로 돌아옴

④ E를 비어 있던 0번 칸에 넣음     [E] [B] [C] [D]
   head=1, tail=1, size=4           논리적 순서: B → C → D → E
   가득 찼으므로 다음 삽입은 거절해야 함
```

마지막 상태에서 배열을 0번부터 읽으면 `E, B, C, D`지만, 큐의 앞은 `head=1`입니다. 따라서 **1 → 2 → 3 → 0번 칸**으로 읽어야 FIFO 순서인 `B, C, D, E`가 됩니다. 물리적으로 배열이 둥글어지는 것이 아니라, 인덱스를 끝에서 처음으로 돌려 **논리적으로 순환**시키는 구조입니다.

다음 코드는 위의 ①에서 출발해 ④까지 실행합니다. 초기 상태에서 꺼낼 원소가 있고, 추가할 두 원소를 담을 공간이 있음을 알고 있으므로 빈 상태·가득 찬 상태의 검사 코드는 생략했습니다. 일반적인 큐 구현에서는 삭제 전에 `size == 0`, 삽입 전에 `size == 용량`을 검사해야 합니다.

**Java**

```java
String[] buffer = {"A", "B", "C", null};
int head = 0, tail = 3, size = 3;

String removed = buffer[head];
buffer[head] = null;
head = (head + 1) % buffer.length;
size--;
System.out.println(removed); // A

for (String value : new String[]{"D", "E"}) {
    buffer[tail] = value;
    tail = (tail + 1) % buffer.length;
    size++;
}
for (int i = 0; i < size; i++) {
    System.out.println(buffer[(head + i) % buffer.length]); // B, C, D, E
}
```

**Python**

```python
buffer = ["A", "B", "C", None]
head, tail, size = 0, 3, 3

removed = buffer[head]
buffer[head] = None
head = (head + 1) % len(buffer)
size -= 1
print(removed)  # A

for value in ["D", "E"]:
    buffer[tail] = value
    tail = (tail + 1) % len(buffer)
    size += 1
for i in range(size):
    print(buffer[(head + i) % len(buffer)])  # B, C, D, E
```

- **작업 횟수:** <br>
삭제 한 번은 원소 읽기, 칸 비우기, `head` 갱신, `size` 갱신을 각각 한 번 수행합니다. 삽입 한 번은 원소 저장, `tail` 갱신, `size` 갱신을 각각 한 번 수행합니다. 나머지 원소를 옮기는 횟수는 모두 **0번**이며, 저장된 원소가 3개에서 더 많아져도 이 단계 수는 늘지 않습니다.

- **복잡도:** <br>
인덱스 계산을 상수 시간으로 보는 일반적인 분석에서 삽입·삭제 한 번은 `O(1)`입니다. 마지막 출력 반복문은 별도 작업으로, 원소 4개를 한 번씩 읽으며 일반적으로 `N`개면 `O(N)`입니다. 용량이 `C`인 큐는 원소 칸 `C`개와 인덱스·개수 변수 몇 개를 유지하므로 저장 공간은 `O(C)`입니다.

이 예시에서는 **빈 큐도 가득 찬 큐도 `head == tail`일 수 있습니다.** `size == 0`이면 비었고 `size == 4`이면 가득 찬 것으로 구분합니다. 가득 찼는데 계속 쓰면 아직 꺼내지 않은 원소를 덮어쓰므로, 일반적인 FIFO 큐에서는 검사 후 거절·대기·확장 중 정한 정책을 적용해야 합니다.

**면접에서 정리하면:** “원형 큐는 맨 앞과 다음 삽입 위치를 인덱스로 관리하고, 배열 끝에서 처음으로 돌아가 빈칸을 재사용합니다. 원소를 밀지 않으므로 확장이 없는 삽입·삭제는 일정한 작업만 필요합니다.”

<details>
<summary>이해 보충: 원형 큐가 가득 차면 어떻게 확장할까?</summary>

위 ④ 상태에서 새 원소 `F`까지 받아야 한다고 해 봅시다. 용량 제한을 유지한다면 거절하거나 공간이 생기기를 기다려야 합니다. 반면 동적 원형 배열은 더 큰 배열로 옮길 수 있습니다.

```text
기존 4칸: [E] [B] [C] [D]       head=1, size=4
읽는 순서:  1 → 2 → 3 → 0
                  ↓ 논리적 순서대로 4개 복사
새   8칸: [B] [C] [D] [E] [·] [·] [·] [·]
          head=0, tail=4

F 추가:   [B] [C] [D] [E] [F] [·] [·] [·]
          head=0, tail=5, size=5
```

확장 때 원소 `N`개를 복사하면 그 삽입 한 번은 `O(N)`입니다. 그러나 용량을 일정 비율로 늘리는 구현은 확장이 매번 일어나지 않아, 여러 번의 추가에 든 비용을 나누면 추가 한 번당 **분할 상환 `O(1)`**로 분석할 수 있습니다. 앞의 [동적 배열 확장 설명](<../07. 배열과 동적 배열/README.md>)과 같은 원리입니다.

위 그림은 이해를 위한 **2배 확장 예시**이며 Java `ArrayDeque`의 실제 복사·확장 코드를 그대로 나타낸 것은 아닙니다.

</details>

<details>
<summary>심화: Java ArrayDeque와 Python deque의 내부도 같을까?</summary>

Java `ArrayDeque`는 원형 동적 배열을 사용합니다. 다만 **배열 크기가 항상 2의 거듭제곱이라는 설명은 버전에 관계없이 적용할 수 없습니다.** OpenJDK 21 구현은 기본 생성 시 내부 배열을 17칸으로 만들고, 인덱스를 증가시킨 뒤 배열 길이에 도달하면 0으로 되돌립니다. 확장 비율도 항상 2배가 아닙니다. [OpenJDK 21 ArrayDeque 구현](https://github.com/openjdk/jdk21u/blob/master/src/java.base/share/classes/java/util/ArrayDeque.java)

또한 이 구현은 정상 상태에서 빈칸을 하나 남겨 두며, `head == tail`은 빈 상태를 나타냅니다. 삽입 중 일시적으로 두 인덱스가 같아지면 즉시 확장합니다. 본문의 `size`를 따로 저장하는 큐와 **빈 상태·가득 찬 상태를 구별하는 방식이 다릅니다.**

CPython 3.13의 `deque`는 여러 원소를 담는 고정 크기 **블록들을 양방향으로 연결**합니다. 원소 하나마다 노드를 만드는 일반적인 연결 리스트나, 하나의 큰 원형 배열과는 다릅니다. 양 끝 조작이 가능하다는 공통점과 내부 저장 방식이 같다는 주장은 구분해야 합니다. [CPython 3.13 deque 구현](https://github.com/python/cpython/blob/3.13/Modules/_collectionsmodule.c)

</details>

---

### 3-5. 덱: 양 끝을 모두 사용할 수 있다면?

스택과 큐의 예시에서는 사용할 입출력 위치를 정해 두었습니다. 덱은 양쪽 끝을 모두 열어 두므로, 앞뒤 어느 쪽에 넣고 어느 쪽에서 꺼낼지 연산마다 선택할 수 있습니다.

**Java**

```java
java.util.Deque<String> deque = new java.util.ArrayDeque<>();
deque.addLast("B");     // [B]
deque.addFirst("A");    // [A, B]
deque.addLast("C");     // [A, B, C]
System.out.println(deque.removeFirst()); // A → [B, C]
System.out.println(deque.removeLast());  // C → [B]
```

**Python**

```python
from collections import deque

items = deque()
items.append("B")       # [B]
items.appendleft("A")   # [A, B]
items.append("C")       # [A, B, C]
print(items.popleft())  # A → [B, C]
print(items.pop())      # C → [B]
```

`B`를 먼저 넣고 그 앞에 `A`, 뒤에 `C`를 붙였습니다. 이후 앞에서 `A`, 뒤에서 `C`를 꺼내 `B`만 남습니다. **덱은 중간 위치를 자유롭게 조작하는 규칙이 아니라 양 끝을 조작하는 규칙**이라는 점에 주목하면 됩니다.

**작업 횟수와 복잡도:** 삽입 3번과 삭제 2번, 총 5번의 양 끝 연산을 수행하고 최대 3개를 보관합니다. Java `ArrayDeque`에서는 이런 연산을 총 `M`번 수행할 때 확장 비용을 포함해 분할 상환 관점에서 `O(M)`으로 볼 수 있습니다. 저장 공간은 동시에 보관하는 원소 수와 확보한 용량에 따라 결정됩니다. [Java ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html)

**면접에서 정리하면:** “덱은 양 끝 삽입·삭제를 지원합니다. 같은 끝을 사용하면 스택, 반대쪽 끝을 사용하면 큐로 활용할 수 있습니다.”

앞에서 사용한 연산을 비교하면 다음과 같습니다. 모든 예시는 왼쪽을 앞, 오른쪽을 뒤로 봅니다.

| 동작 | Java `Deque` | Python `collections.deque` |
|---|---|---|
| 앞에 넣기 | `addFirst(x)` | `appendleft(x)` |
| 뒤에 넣기 | `addLast(x)` | `append(x)` |
| 앞에서 꺼내기 | `removeFirst()` | `popleft()` |
| 뒤에서 꺼내기 | `removeLast()` | `pop()` |
| 앞 원소 보기 | `peekFirst()` | `d[0]` |
| 뒤 원소 보기 | `peekLast()` | `d[-1]` |

빈 자료구조에 대한 동작은 다릅니다. Java의 `removeFirst()`·`removeLast()`는 예외를 던지고, `pollFirst()`·`pollLast()`는 `null`을 반환합니다. `peekFirst()`·`peekLast()`도 비었으면 `null`입니다. Python의 위 삭제·인덱스 조회는 비었으면 `IndexError`가 발생합니다. [Java Deque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html), [Python deque 문서](https://docs.python.org/3/library/collections.html#collections.deque)

<details>
<summary>이해 보충: 스레드 사이의 작업 큐도 deque로 만들면 될까?</summary>

Java `ArrayDeque`는 스레드 안전성을 제공하지 않습니다. Python `deque`의 양 끝 추가·삭제는 스레드 안전한 연산으로 문서화되어 있지만, **비었는지 확인한 뒤 꺼내는 두 동작 전체**까지 하나의 원자적 연산이 되는 것은 아닙니다. 확인 직후 다른 스레드가 마지막 원소를 꺼낼 수 있습니다. 본문의 예시는 단일 스레드에서 실행합니다. [Java ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html), [Python deque 문서](https://docs.python.org/3/library/collections.html#collections.deque)

여러 스레드가 작업을 주고받으며 빈 큐나 가득 찬 큐에서 기다려야 한다면 Java의 `BlockingQueue` 구현, Python의 `queue.Queue`처럼 해당 기능을 갖춘 자료구조를 사용합니다. 기다리는 시간은 내부 원소 조작의 시간 복잡도와 별개입니다. [Java ArrayBlockingQueue 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ArrayBlockingQueue.html), [Python queue 문서](https://docs.python.org/3/library/queue.html)

Python의 `deque(maxlen=...)`는 작업 큐의 배압과 다릅니다. 가득 찬 상태에서 한쪽에 `append`하면 반대쪽 원소를 버리므로, 반드시 처리해야 하는 작업을 보존할 목적이라면 이 차이를 주의해야 합니다. [Python deque 문서](https://docs.python.org/3/library/collections.html#collections.deque)

</details>

<details>
<summary>심화: 콜 스택과 재귀는 자료구조 스택과 어떻게 연결될까?</summary>

일반적인 중첩 함수 호출에서는 호출한 함수가 호출받은 함수의 완료를 기다립니다. `main → order → pay` 순서로 호출하면 가장 안쪽의 `pay`가 끝난 뒤 `order`, 그다음 `main`으로 돌아갑니다. 진행 중인 호출 정보를 나중에 쌓은 것부터 꺼내므로 LIFO 구조와 연결됩니다.

```text
호출: main → order → pay       정상 복귀: pay → order → main

          꼭대기
         [ pay   ] ← 현재 호출의 실행 정보
         [ order ] ← pay가 끝나면 이어서 실행
         [ main  ] ← order가 끝나면 이어서 실행
```

메서드 호출마다 지역 변수와 실행 중 필요한 정보를 담는 **스택 프레임**이 만들어집니다. JVM 프레임에는 지역 변수, 피연산자 스택 등의 정보가 포함됩니다. 하지만 JVM 스택을 특정 CPU의 `RSP` 레지스터나 고정된 메모리 성장 방향과 동일시해서는 안 됩니다. 명세는 JVM 스택 메모리가 연속적일 필요도 없다고 설명합니다. [JVM 스택·프레임 명세](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html#jvms-2.5.2)

호출 깊이가 실행 환경에서 허용하는 한계를 넘으면 `StackOverflowError`가 발생할 수 있습니다. 한계는 스택 설정과 프레임 크기 등에 따라 달라지므로 특정 재귀 횟수로 고정할 수 없습니다. 또한 `O(1)`인 조작이라는 설명은 **실행 비용이 0클럭**이라는 뜻이 아닙니다.

깊은 탐색을 반복문과 명시적 스택으로 바꾸면 그 탐색이 호출 프레임을 계속 쌓는 문제를 줄일 수 있습니다. 그러나 대기 중인 탐색 대상을 저장할 메모리는 여전히 필요합니다. `A → B → A` 같은 순환이 있다면 방문 여부 검사 등도 필요하며, 반복문으로 바꾸는 것만으로 종료나 메모리 안전이 보장되지는 않습니다.

</details>

<details>
<summary>심화: 덱은 작업 훔치기와 슬라이딩 윈도우에도 어떻게 쓰일까?</summary>

**작업 훔치기(work-stealing)**에서는 작업이 없는 스레드가 다른 스레드의 대기 작업을 가져옵니다. 덱의 한쪽 끝은 소유 스레드가 사용하고 반대쪽 끝에서는 다른 스레드가 가져가도록 설계할 수 있습니다. Java `ForkJoinPool`은 작업 훔치기를 사용하지만, 일반 `ArrayDeque`를 여러 스레드에 공유하는 것만으로 구현되는 것은 아닙니다. [ForkJoinPool 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html)

**슬라이딩 윈도우**는 연속된 일정 범위를 한 칸씩 옮기며 계산하는 방식입니다. 범위의 최댓값을 찾는 알고리즘에서는 덱 앞에서 범위를 벗어난 후보를 제거하고, 뒤에서 새 값보다 작아 더 이상 필요 없는 후보를 제거할 수 있습니다. 양 끝 삭제가 각각 다른 역할을 맡는 사례이며, 자세한 알고리즘은 별도 주제로 다룹니다.

</details>

