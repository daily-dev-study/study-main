## 1. 면접 핵심 Q&A

### Q1. 배열과 동적 배열은 어떻게 다른가요?

**배열은 생성한 뒤 길이가 고정되고, 동적 배열은 원소가 늘어나면 내부 저장 공간을 확장합니다.** Java `int[]`는 길이 3으로 만들면 네 번째 칸을 붙일 수 없습니다. Java `ArrayList`와 Python `list`는 사용 중인 칸과 확보한 칸을 구분해 관리하며, 자리가 부족하면 더 큰 공간을 마련합니다. [Java 배열 명세](https://docs.oracle.com/javase/specs/jls/se21/html/jls-10.html), [ArrayList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html), [CPython 리스트 구현](https://github.com/python/cpython/blob/main/Include/cpython/listobject.h)

### Q2. 배열은 왜 인덱스로 바로 찾을 수 있나요?

**같은 크기의 저장 칸이 순서대로 놓여 있어 시작 위치와 인덱스로 목표 칸을 찾기 때문입니다.** 앞의 원소들을 차례로 방문할 필요가 없으므로 인덱스 접근은 `O(1)`입니다. 다만 `O(1)`은 접근 시간이 0이라는 뜻도, 모든 접근이 똑같은 속도라는 뜻도 아닙니다.

### Q3. 동적 배열의 맨 뒤 추가는 왜 분할 상환 O(1)인가요?

**빈 칸이 있으면 바로 넣고, 꽉 찼을 때만 확장하기 때문입니다.** 확장하는 한 번은 기존 `N`개를 옮겨 `O(N)`이 될 수 있습니다. 하지만 여러 번의 끝 추가에 드는 전체 이동을 나누어 보면 추가 한 번당 분할 상환 비용은 `O(1)`입니다. [ArrayList 성능 설명](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html), [CPython 리스트 확장 구현](https://github.com/python/cpython/blob/main/Objects/listobject.c)

### Q4. 중간 삽입과 삭제는 왜 O(N)인가요?

**순서를 유지하려면 그 위치 뒤의 원소들을 한 칸씩 옮겨야 하기 때문입니다.** 맨 앞에 추가하면 기존 원소 `N`개가 밀리고, 맨 앞에서 삭제하면 `N−1`개가 당겨집니다. 끝에 가까울수록 옮길 원소는 줄어듭니다. [ArrayList의 삽입·삭제](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html), [Python 리스트를 큐로 쓸 때의 비용](https://docs.python.org/3/tutorial/datastructures.html#using-lists-as-queues)

### Q5. 동적 배열에서는 객체 자체가 연속해서 놓이나요?

**객체를 담는 Java `ArrayList`와 CPython `list`에서는 원소를 가리키는 참조들이 저장 공간에 나란히 놓입니다.** 참조 대상 객체 자체가 모두 옆에 있다는 뜻은 아닙니다. 반면 Java `int[]`에는 정수 값이 배열의 원소 칸에 직접 담깁니다. [CPython 리스트 구조](https://github.com/python/cpython/blob/main/Include/cpython/listobject.h)

<details>
<summary>심화 Q&A: 연결 리스트라면 중간 삽입이 더 빠를까요?</summary>

이미 삽입할 노드를 가리키고 있다면 연결만 바꾸는 작업은 일정한 비용으로 볼 수 있습니다. 그러나 **5만 번째 인덱스**를 받아 그 자리를 찾아야 한다면 노드를 따라 이동해야 하므로 탐색에 `O(N)`이 듭니다. 같은 `O(N)`이라도 실제 속도는 객체 배치와 캐시, 이동 비용에 따라 달라서 측정 없이 어느 쪽이 빠르다고 단정할 수 없습니다. [Java LinkedList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html)

</details>


---


## 2. 실무 사례

### 처리 대기 목록의 첫 항목을 계속 꺼낸다면?

> 원리를 설명하는 **학습용 가상 사례**입니다. 특정 서비스에서 측정한 장애 사례가 아닙니다.

**상황:** 외부 API에서 받아온 작업을 순서대로 처리합니다. 아래 코드에서는 목록의 첫 작업을 꺼낸 뒤 다음 작업을 다시 첫 칸으로 당깁니다.

**Java**

```java
List<String> jobs = new ArrayList<>(List.of("A", "B", "C", "D"));
while (!jobs.isEmpty()) {
    String job = jobs.remove(0);
    process(job);
}
```

**Python**

```python
jobs = ["A", "B", "C", "D"]
while jobs:
    job = jobs.pop(0)
    process(job)
```

**원인:** `A`를 빼면 `B·C·D` 세 칸을 옮깁니다. 이어 `B`를 빼면 두 칸, `C`를 빼면 한 칸을 옮겨 **총 3 + 2 + 1 = 6칸**입니다. 작업이 `N`개면 이동은 `(N−1) + … + 1 = N(N−1)/2`칸으로 늘어 `O(N²)`입니다. 여기서 센 것은 *원소 또는 참조의 이동 횟수*이며 `process`의 비용은 제외했습니다. [ArrayList 삭제 동작](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html), [Python 리스트 큐 설명](https://docs.python.org/3/tutorial/datastructures.html#using-lists-as-queues)

```text
처음       [A][B][C][D]
A 꺼낸 뒤  [B][C][D]     ← 뒤의 3칸 이동
B 꺼낸 뒤  [C][D]        ← 뒤의 2칸 이동
C 꺼낸 뒤  [D]           ← 뒤의 1칸 이동
                              합계 6칸 이동
```

**개선:** 앞에서 꺼내는 일이 주된 작업이면 Java `ArrayDeque`, Python `collections.deque`를 사용합니다. 양쪽 끝을 효율적으로 다루도록 만든 자료구조입니다.

**Java**

```java
Deque<String> jobs = new ArrayDeque<>(List.of("A", "B", "C", "D"));
while (!jobs.isEmpty()) {
    String job = jobs.removeFirst();
    process(job);
}
```

**Python**

```python
from collections import deque

jobs = deque(["A", "B", "C", "D"])
while jobs:
    job = jobs.popleft()
    process(job)
```

각 작업을 한 번씩 꺼내 처리하므로 큐 연산의 합계는 `O(N)`입니다. Java `ArrayDeque`의 끝 연산은 확장을 포함해 **분할 상환 `O(1)`**, Python `deque`의 양끝 연산은 약 `O(1)`입니다. `process` 자체의 비용과 입력을 큐에 담는 비용은 별도로 더해야 합니다. [ArrayDeque 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html), [Python deque 문서](https://docs.python.org/3/library/collections.html#collections.deque)

**판단 기준:** 인덱스로 자주 조회하고 끝에 추가한다면 동적 배열이 잘 맞습니다. 앞에서 계속 꺼내야 한다면 큐를 사용합니다. 작업 수가 작고 드물다면 단순한 목록으로도 충분할 수 있습니다.

<details>
<summary>이해 보충: 추가할 개수를 미리 안다면?</summary>

Java `ArrayList`에는 초기 용량을 지정하는 생성자가 있습니다. 아래처럼 3건을 넣을 것을 알고 있다면 적어도 3칸을 확보해 추가 중 확장 횟수를 줄일 수 있습니다. 이것은 **확장의 상수 비용을 줄이는 방법**이며, 끝 추가의 분할 상환 `O(1)`을 다른 복잡도로 바꾸지는 않습니다. [ArrayList의 초기 용량 생성자](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html)

**Java**

```java
List<String> source = List.of("A", "B", "C");
List<String> jobs = new ArrayList<>(source.size());
for (String job : source) {
    jobs.add(job);
}
```

**Python**

```python
source = ["A", "B", "C"]
jobs = []
for job in source:
    jobs.append(job)
```

Python `list`에는 Java `ArrayList`처럼 용량을 지정하는 공개 생성자 인자가 없습니다. 위 코드는 같은 원소를 순서대로 추가하는 예시이며, Python에서 용량을 미리 지정했다는 뜻은 아닙니다. 입력을 통째로 복사하면 Java에서는 `new ArrayList<>(source)`, Python에서는 `list(source)`처럼 작성할 수 있습니다.

</details>


---


## 3. Deep Dive: 가까이 둔 칸은 어떻게 늘어나고 움직일까?

### 3-1. 세 번째 원소를 찾을 때 앞의 두 개를 읽어야 할까?

Java 배열 `int[]`를 `[10, 20, 30]`이라고 그려 봅시다. 같은 타입의 원소 칸이 순서대로 놓이므로 `values[2]`의 위치는 시작 위치에서 두 칸 떨어져 있습니다. `values[0]`, `values[1]`을 방문해 찾지 않습니다.

```text
인덱스                 0       1       2
Java int[] 원소 칸    [ 10 ] [ 20 ] [ 30 ]
                                      ↑ values[2]: 바로 세 번째 칸

Java ArrayList<Integer> / CPython list의 저장 칸
                      [ 참조 A ][ 참조 B ][ 참조 C ]
                         │         │         │
                         ↓         ↓         ↓
                      객체(10)  객체(20)  객체(30)

저장 칸의 참조는 나란하지만, 아래 객체들이 나란히 놓인다는 뜻은 아닙니다.
```

동적 배열도 인덱스 접근 방식은 같습니다. 아래 두 예시는 모두 마지막 값 `30`을 읽습니다.

**Java**

```java
List<Integer> values = new ArrayList<>(List.of(10, 20, 30));
int selected = values.get(2); // 30
```

**Python**

```python
values = [10, 20, 30]
selected = values[2]  # 30
```

인덱스 2에 해당하는 **저장 칸 조회 한 번**을 작업으로 세면 원소가 3개든 3만 개든 앞의 원소를 훑지 않으므로 `O(1)`입니다. 다만 Java `int[]`의 칸에서는 정수 값을 직접 읽고, `ArrayList<Integer>`와 CPython `list`의 칸에서는 정수 객체를 가리키는 참조를 읽습니다.

#### 왜 값을 직접 넣지 않고 참조를 저장할까?

**참조는 객체의 위치를 찾아갈 수 있는 정보**입니다. 주소가 적힌 쪽지를 보고 해당 장소로 찾아가는 것처럼, 리스트 칸의 참조를 읽고 그 참조가 가리키는 객체에 접근합니다. Java의 참조를 실제 메모리 주소 숫자 그 자체라고 단정할 필요는 없습니다.

```text
정수 값을 사용하는 과정

int[]                  배열 칸 [20] → 값 사용
ArrayList / CPython list  칸 [참조] → 정수 객체 [20] → 값 사용
```

Java `int[]`는 같은 크기의 정수만 담으므로 값을 직접 저장하기 좋습니다. 하지만 리스트는 문자열이나 사용자 정보처럼 다양한 객체도 다뤄야 합니다. 예를 들어 문자열의 문자들까지 각 칸에 모두 넣으려면, 문자열 길이가 제각각이라 칸마다 필요한 공간이 달라집니다.

그래서 **일정한 크기의 참조를 배열에 나란히 저장하고, 실제 객체는 따로 둡니다.** 다음 그림에서 문자열은 실제 메모리 구조를 단순화해 내용으로 표시했습니다.

```text
저장 칸:    [참조 A][참조 B][참조 C]   ← 같은 크기의 참조 칸

참조 A ──→ 문자열 객체 "안녕"
참조 B ──→ 문자열 객체 "반갑습니다"
참조 C ──→ 문자열 객체 "A"
```

이렇게 하면 객체의 내용이 달라도 **인덱스로 참조 칸을 바로 찾을 수 있습니다.** 또 리스트가 커지거나 원소의 순서를 바꿀 때, 객체 전체를 복사하지 않고 참조만 옮길 수 있습니다. 대신 실제 객체의 값을 사용하려면 참조를 따라가는 메모리 접근 비용이 추가될 수 있습니다. 앞의 원소들을 순서대로 찾는 과정은 없으므로 인덱스 접근의 `O(1)`은 유지됩니다. [CPython 리스트의 저장 구조](https://docs.python.org/3/faq/design.html#how-are-lists-implemented-in-cpython)

`ArrayList<Integer>`도 객체 참조를 담는 `ArrayList`의 공통 구조를 사용합니다. Java의 제네릭 타입 인자에는 `int` 같은 원시 타입을 쓸 수 없어, 정수를 객체로 표현한 `Integer`를 사용합니다. CPython `list` 역시 정수를 포함한 원소들을 객체 참조로 저장합니다. [Java 제네릭의 타입 제한](https://docs.oracle.com/javase/tutorial/java/generics/restrictions.html)

**동적 배열이라고 반드시 참조를 저장해야 하는 것은 아닙니다.** 정수 값을 직접 담는 배열을 내부에 두고, 꽉 찼을 때 더 큰 배열로 바꾸는 동적 배열도 만들 수 있습니다. 크기를 늘리는 방식과 각 칸에 값·참조 중 무엇을 저장하는지는 별개의 선택입니다.

<details>
<summary>이해 보충: Python과 CPython은 어떻게 다를까?</summary>

**Python은 프로그래밍 언어이고, CPython은 Python 코드를 실제로 실행해 주는 대표적인 구현체입니다.** 여기서 구현체란 언어의 문법과 동작 규칙에 맞춰 만들어진 실행 프로그램을 뜻합니다. CPython은 주로 C로 작성되었으며, python.org에서 배포하는 Python이 이에 해당합니다. [Python 공식 용어집](https://docs.python.org/3/glossary.html#term-CPython), [Python 구현체 소개](https://docs.python.org/3/reference/introduction.html#alternate-implementations)

```text
Python 언어의 규칙에 맞춰 작성한 코드
                  │
                  ▼
          CPython 같은 구현체
                  │
                  ▼
               실행 결과
```

Python을 실행하는 구현체에는 PyPy 등도 있습니다. 같은 언어의 규칙을 따르더라도 메모리에 자료구조를 배치하고 실행하는 방식은 달라질 수 있습니다.

그래서 이 문서는 리스트의 사용법을 말할 때는 **Python의 list**, 내부 저장 방식을 설명할 때는 **CPython의 list**라고 표현합니다. “리스트에서 인덱스로 원소를 읽는다”는 사용법과 “참조 배열을 내부에 둔다”는 구현 설명을 구분하는 것입니다.

</details>

<details>
<summary>이해 보충: 주소 계산과 캐시를 연결하면?</summary>

같은 크기의 원소가 연속된 배열이라면 개념적으로 `원소 위치 = 시작 위치 + 인덱스 × 원소 크기`입니다. Java 배열에는 범위 검사도 있으며, 이 식을 특정 CPU 명령어나 고정된 클럭 수로 단정할 수는 없습니다. [Java 배열 접근 명세](https://docs.oracle.com/javase/specs/jls/se21/html/jls-10.html)

인접한 원소를 차례로 읽으면 [앞서 배운 공간 지역성](<../04. 메모리 계층 구조와 캐시 메모리/README.md>)을 활용하기 쉽습니다. 다만 다음 원소가 반드시 캐시에 있거나 모든 객체가 같은 캐시 라인에 놓인다는 뜻은 아닙니다. 객체 참조 배열에서는 나란한 것이 참조이며, 참조 대상 객체의 배치는 별개입니다.

</details>

---

### 3-2. 맨 뒤에 새 원소를 넣을 자리가 없다면?

Java 배열은 길이가 정해져 있어 같은 배열에 칸을 덧붙일 수 없습니다. 반면 동적 배열은 **크기(size)** 와 **용량(capacity)** 을 따로 관리합니다. 크기는 실제 원소 수이고, 용량은 현재 확보한 칸 수입니다.

**Java**

```java
List<String> letters = new ArrayList<>();
letters.add("A");
letters.add("B");
letters.add("C");
```

**Python**

```python
letters = []
letters.append("A")
letters.append("B")
letters.append("C")
```

설명을 위해 처음 용량이 **2칸**이고, 가득 차면 **4칸**으로 늘리는 동적 배열을 상상해 봅시다. 이는 동작을 세기 위한 모형이며 위 Java·Python 코드의 실제 용량을 말하지 않습니다.

```text
추가 전: 크기 2, 용량 2
기존 공간          [ A ][ B ]             ← 빈 칸 없음
새 공간 확보       [   ][   ][   ][   ]   ← 용량 4
A·B 이동           [ A ][ B ][   ][   ]   ← 기존 원소 2개 이동
C 쓰기             [ A ][ B ][ C ][   ]   ← 크기 3, 빈 칸 1
```

1. `A`, `B`를 넣습니다. 원소 쓰기 두 번으로 크기 2, 용량 2가 됩니다.
2. `C`를 넣으려면 자리가 없습니다. 더 큰 저장 공간을 확보하고 `A`, `B`를 옮긴 다음 `C`를 씁니다.
3. 이제 크기 3, 용량 4입니다. 다음 추가에는 남은 칸을 사용할 수 있습니다.

이 예시에서 **원소 쓰기 3번 + 기존 원소 이동 2번 = 5번**입니다. 확장이 일어난 세 번째 추가만 보면 이동 2번이 붙어 `O(N)`이지만, 앞뒤 추가까지 함께 보면 매번 옮기지는 않았습니다. Java `ArrayList`는 내부 배열을 확장하며, CPython `list`는 확보해 둔 참조 칸을 관리하고 재할당 시 제자리 확장이 가능할 수도 있습니다. 둘 다 **확장 때 기존 참조들을 옮겨야 할 수 있다**는 점이 핵심입니다. [ArrayList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html), [CPython 리스트 구현](https://github.com/python/cpython/blob/main/Objects/listobject.c)

그렇다면 왜 한 칸씩만 늘리지 않을까요? 매번 꽉 찰 때 한 칸만 늘리면 `N`개를 끝에 추가하는 동안 복사 횟수가 `0 + 1 + … + (N−1) = N(N−1)/2`가 됩니다. **`O(N²)`의 이동**이 생기는 방식입니다.

#### 1부터 8까지 추가하면서 비용을 나누어 보자

**분할 상환은 가끔 비싸게 드는 작업까지 여러 연산의 비용을 모두 합쳐, 연산 한 번당 비용으로 나누어 보는 방법**입니다. 두 배씩 확장하면 왜 도움이 되는지, 원소 8개를 넣으며 직접 세어 봅시다.

아래 두 코드는 반복할 때마다 숫자를 하나씩 맨 뒤에 추가합니다.

**Java**

```java
List<Integer> values = new ArrayList<>();
for (int value = 1; value <= 8; value++) {
    values.add(value);
}
```

**Python**

```python
values = []
for value in range(1, 9):
    values.append(value)
```

이번 계산에서는 **빈 배열의 초기 용량이 1칸이고, 꽉 차면 두 배로 늘리는 학습용 모형**을 사용합니다. 위 코드의 실제 내부 용량을 추적한 결과는 아닙니다. 작업은 **기존 원소의 참조 하나를 복사하거나 새 참조 하나를 쓰는 것**을 각각 1번으로 세며, 실제 CPU 명령어 수나 메모리 할당 시간은 세지 않습니다.

먼저 `1`, `2`, `3`을 추가하며 용량이 `1 → 2 → 4`로 커졌다고 합시다. `4`를 넣을 때는 빈 칸이 있으므로 쓰기 한 번이면 됩니다. 그림의 숫자는 해당 정수 객체를 가리키는 참조를 간단히 표시한 것입니다.

```text
4 추가: 빈 칸이 있는 경우

추가 전  [1][2][3][ ]
추가 후  [1][2][3][4]

작업: 새 참조 쓰기 1번
```

그런데 다음 숫자 `5`를 넣으려면 공간이 부족합니다. 더 큰 배열로 기존 참조를 옮겨야 하므로 이번 추가에는 작업이 더 듭니다.

```text
5 추가: 확장이 필요한 경우

기존 공간     [1][2][3][4]                 ← 꽉 참
새 공간 확보  [ ][ ][ ][ ][ ][ ][ ][ ]
기존 참조 복사 [1][2][3][4][ ][ ][ ][ ]    ← 복사 4번
5 추가        [1][2][3][4][5][ ][ ][ ]     ← 쓰기 1번

작업: 복사 4번 + 쓰기 1번 = 5번
```

**한 번 확장하면서 빈 칸을 함께 확보했기 때문에, 그다음 `6`, `7`, `8`을 넣을 때는 각각 쓰기 1번이면 됩니다.** 1부터 8까지의 작업을 빠짐없이 적으면 다음과 같습니다.

| 추가하는 값 | 용량 변화 | 기존 참조 복사 | 새 참조 쓰기 | 이번 작업 합계 |
|---|---|---:|---:|---:|
| 1 | 1 유지 | 0 | 1 | 1 |
| 2 | 1 → 2 | 1 | 1 | 2 |
| 3 | 2 → 4 | 2 | 1 | 3 |
| 4 | 4 유지 | 0 | 1 | 1 |
| 5 | 4 → 8 | 4 | 1 | 5 |
| 6 | 8 유지 | 0 | 1 | 1 |
| 7 | 8 유지 | 0 | 1 | 1 |
| 8 | 8 유지 | 0 | 1 | 1 |
| **합계** | | **7** | **8** | **15** |

8번 추가하는 동안 작업은 총 15번입니다. 이를 추가 횟수로 나누면 `15 ÷ 8 = 1.875`번입니다. **이것은 여러 추가의 비용을 나누어 계산한 값**입니다. `5`를 추가할 때 실제로 작업을 5번 했다는 사실은 그대로입니다.

#### 원소 수를 늘려도 추가 한 번당 비용이 계속 커질까?

같은 모형에서 16개, 32개까지 넣어 보면 확장 시 복사 횟수는 각각 `1 + 2 + 4 + 8`, `1 + 2 + 4 + 8 + 16`입니다. 원소 수가 늘어날수록 확장 한 번의 복사량도 커지지만, 다음 확장까지 사용할 빈 칸도 많아집니다.

| 추가한 원소 수 N | 기존 참조 복사 | 새 참조 쓰기 | 전체 작업 | 추가 한 번당 나눈 비용 |
|---:|---:|---:|---:|---:|
| 8 | 7 | 8 | 15 | 1.875 |
| 16 | 15 | 16 | 31 | 약 1.94 |
| 32 | 31 | 32 | 63 | 약 1.97 |

이처럼 `N`이 2의 거듭제곱이면 전체 작업은 `(N−1) + N = 2N−1`번입니다. 그 사이에서 멈춰도 이 모형의 전체 작업은 `3N` 미만이어서, `N`의 일정한 배수 이내에 머뭅니다. 따라서 **N번 추가하는 전체 비용은 `O(N)`이고, 추가 횟수 N으로 나누면 분할 상환 `O(1)`**입니다. 실제 작업 횟수를 N번으로 바꾼 것이 아니라, 작업량이 N에 비례해 늘어난다는 뜻입니다.

```text
한 번의 추가를 따로 보면
├─ 빈 칸 있음 → 참조 하나 쓰기              → O(1)
└─ 꽉 참     → 기존 N개 복사 + 하나 쓰기   → O(N)

빈 배열부터 N번 추가하는 전체 과정을 보면
전체 비용 O(N) ÷ 추가 N번 → 한 번당 분할 상환 O(1)
```

실제 Java·Python의 성장 규칙이 정확히 두 배일 필요는 없으며, 위 횟수는 분할 상환의 원리를 이해하기 위한 모형입니다. [ArrayList의 분할 상환 비용](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html)

> **면접에서 정리하면:** “동적 배열은 확장할 때 한 번에 `O(N)`이 들 수 있습니다. 하지만 여유 공간을 함께 확보하므로 매번 복사하지 않고, `N`번 끝에 추가하는 전체 비용은 `O(N)`입니다. 따라서 추가 한 번의 분할 상환 비용은 `O(1)`입니다.”

공간은 현재 원소뿐 아니라 여유 칸도 차지하므로 `O(N)`입니다. 확장 과정에서는 기존 저장 공간과 새 공간이 잠시 함께 존재할 수 있습니다. 한 번의 추가에 필요한 작업량과 여러 번의 추가를 나눈 비용은 구분해서 말합니다.

### 분할 상환 O(1)과 평균 시간 복잡도 O(1)은 어떻게 다를까?

둘 다 비용을 나누어 보니 같은 말처럼 느껴질 수 있습니다. **분할 상환은 여러 연산을 이어서 수행한 총비용을 보고, 평균 시간 복잡도는 가능한 입력의 비용에 입력 확률을 반영합니다.** 작은 예시로 나누는 대상이 어떻게 다른지 살펴봅시다.

**1) 분할 상환: 연속된 추가 8번의 총비용을 나눈다**

앞의 동적 배열에서는 실제로 `1`부터 `8`까지 차례로 추가했습니다. 처음 용량이 1이고 꽉 차면 두 배로 늘어나는 규칙에 따라, 추가별 작업 횟수가 정해집니다.

```text
추가하는 값     1   2   3   4   5   6   7   8
이번 작업 수   1   2   3   1   5   1   1   1
              └──────── 합계 15번 ─────────┘

연속된 추가 8번의 총비용 ÷ 추가 횟수 8
= 15 ÷ 8 = 한 번당 1.875번
```

확장할 때 많이 일한 덕분에 여유 칸이 생겨 다음 추가들이 저렴해집니다. **비싼 추가가 나올 확률을 가정하지 않아도**, 확장 규칙으로 전체 비용을 계산할 수 있습니다. 앞에서 확인했듯 빈 배열부터 `N`번 끝에 추가하는 전체 비용이 `O(N)` 이내이므로, 추가 한 번의 분할 상환 비용은 `O(1)`입니다.

**2) 평균: 한 번 실행할 때 가능한 입력들의 비용을 확률에 따라 나눈다**

비교를 위한 학습용 규칙을 만들어 봅시다. 입력 번호가 `1`이면 `N`번 처리하고, 나머지 번호이면 한 번만 처리합니다. **1부터 N까지의 번호가 같은 확률로 들어온다고 가정**합니다.

**Java**

```java
int n = 8;
int ticket = 1; // 입력 예시: 1부터 n까지의 번호 중 하나

int repeats = (ticket == 1) ? n : 1;
for (int i = 0; i < repeats; i++) {
    System.out.println("처리");
}
```

**Python**

```python
n = 8
ticket = 1  # 입력 예시: 1부터 n까지의 번호 중 하나

repeats = n if ticket == 1 else 1
for i in range(repeats):
    print("처리")
```

위 코드의 `ticket = 1`은 한 입력을 보여주는 예시이며, 코드는 번호를 무작위로 뽑지 않습니다. 평균을 구할 때는 `ticket`에 1부터 8까지 각각 들어오는 경우를 생각합니다. 출력 한 번을 일정한 비용의 작업으로 세면, 번호 1에는 8번, 나머지 번호에는 각각 1번이 필요합니다.

```text
가능한 입력 번호   1   2   3   4   5   6   7   8
그 입력의 작업 수  8   1   1   1   1   1   1   1
각 입력의 확률    모두 1/8

한 번 실행할 때의 평균 작업 수
= 8 × 1/8 + 1 × 7/8
= 15 ÷ 8 = 1.875번
```

여기서는 **실제로 8번 실행했다는 뜻이 아닙니다.** 한 번 실행할 때 어떤 입력이 들어올지 모르므로, 각 비용에 그 입력이 나올 확률을 곱해 더했습니다. 이렇게 계산한 값을 **기댓값**이라고 합니다.

`N`을 늘려도 번호 1이 들어올 확률은 `1/N`, 나머지가 들어올 확률은 `(N−1)/N`입니다.

```text
평균 작업 수 = N번 × 1/N + 1번 × (N−1)/N
             = 2 − 1/N
```

평균 작업 수가 2 미만으로 유지되므로, **이 입력 분포에서는 평균 시간 복잡도가 `O(1)`**입니다. 다만 번호 1이 계속 들어오면 매번 `N`번 일합니다. 번호 1만 들어오는 분포라면 평균도 `O(N)`으로 달라집니다.

| 비교 항목 | 분할 상환 O(1): 동적 배열 예시 | 평균 O(1): 입력 번호 예시 |
|---|---|---|
| 무엇을 함께 보나요? | 연속해서 수행하는 여러 추가 | 한 번 실행할 때 들어올 수 있는 여러 입력 |
| 어떻게 계산하나요? | 전체 추가 비용 ÷ 추가 횟수 | 각 입력의 비용 × 입력 확률을 모두 더함 |
| 확률 가정이 필요한가요? | 이 확장 규칙의 분석에는 필요 없음 | 번호가 같은 확률로 들어온다는 가정 필요 |
| 비싼 작업은 어떻게 반영하나요? | 확장 이후의 저렴한 추가까지 함께 합산 | 비싼 입력이 나올 확률을 반영 |
| 주의할 점은 무엇인가요? | 개별 추가는 여전히 O(N)일 수 있음 | 입력 분포가 바뀌면 평균도 달라질 수 있음 |

두 예시 모두 `15 ÷ 8`이 나왔지만, 첫 번째는 **이어지는 연산들의 비용**, 두 번째는 **가능한 입력들의 확률에 따른 비용**을 계산한 것입니다. 분할 상환 분석도 몇 번 실행해서 얻은 측정 평균만으로 결론 내리는 것은 아닙니다. 동적 배열에서는 확장 규칙을 통해 `N`번 추가의 전체 비용이 `O(N)` 이내임을 확인했습니다.

> **면접에서 정리하면:** “평균 시간 복잡도는 입력 분포를 가정해 비용의 기댓값을 구합니다. 분할 상환 분석은 여러 연산의 총비용을 연산 횟수에 나누어 보는 방식이며, 동적 배열의 끝 추가는 입력 확률을 가정하지 않아도 분할 상환 `O(1)`입니다.”

---

### 3-3. 가운데에 넣거나 빼면 어떤 칸이 움직일까?

`[A, B, C, D]`의 인덱스 1에 `X`를 넣어 봅시다. `B`, `C`, `D`를 각각 오른쪽으로 한 칸 옮긴 뒤 빈 인덱스 1에 `X`를 씁니다.

```text
인덱스             0    1    2    3    4
삽입 전           [A]  [B]  [C]  [D]  [ ]
뒤에서부터 이동   [A]  [ ]  [B]  [C]  [D]   ← B·C·D, 3칸 이동
X 쓰기            [A]  [X]  [B]  [C]  [D]
```

**Java**

```java
List<String> letters = new ArrayList<>(List.of("A", "B", "C", "D"));
letters.add(1, "X"); // [A, X, B, C, D]
```

**Python**

```python
letters = ["A", "B", "C", "D"]
letters.insert(1, "X")  # [A, X, B, C, D]
```

이때 **원소 칸의 이동**을 한 번의 작업으로 세면 3번 이동하고 1번 씁니다. 길이 `N`인 목록의 인덱스 `i`에 넣을 때 뒤쪽 `N−i`개를 옮기므로, 맨 앞이나 가운데 삽입은 최악 `O(N)`입니다. 용량이 부족하면 확장에 따른 이동도 추가될 수 있지만 전체 증가율은 여전히 `O(N)`입니다.

삭제도 같은 이유로 뒤의 원소를 왼쪽으로 당깁니다. `[A, B, C, D]`에서 인덱스 1의 `B`를 빼면 `C`, `D` 두 칸을 이동합니다. 일반적으로 인덱스 `i`를 지우면 `N−i−1`개를 옮깁니다. 옮기는 것은 Java `ArrayList`와 CPython `list`에서 **객체 전체가 아닌 저장 칸의 참조**입니다. [ArrayList 삽입·삭제](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html), [CPython 삽입 구현](https://github.com/python/cpython/blob/main/Objects/listobject.c)

```text
인덱스             0    1    2    3
삭제 전           [A]  [B]  [C]  [D]
B 삭제 후         [A]  [C]  [D]  [ ]   ← C·D, 2칸 이동
```

**면접에서 정리하면:** 배열은 연속된 원소 칸 덕분에 인덱스로 `O(1)`에 접근합니다. 동적 배열의 끝 추가는 확장 때문에 한 번은 `O(N)`일 수 있지만 여러 번에 나누면 분할 상환 `O(1)`이고, 중간 삽입·삭제는 뒤 원소를 옮겨 `O(N)`입니다.

<details>
<summary>심화: Java의 Arrays.asList와 subList도 동적 배열일까?</summary>

`Arrays.asList(array)`는 원본 배열을 바탕으로 한 **길이 고정 리스트**를 돌려줍니다. 원소 교체는 원본 배열에도 보이지만 크기를 바꾸는 `add`·`remove`는 지원하지 않습니다. `List.of(...)`는 크기 변경뿐 아니라 원소 교체도 지원하지 않는 수정 불가 리스트입니다. [Arrays.asList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html#asList(T...)), [List.of 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#of(E...))

`list.subList(from, to)`는 새 독립 저장 공간이 아닌 **원본의 일부를 보는 뷰**입니다. 원본의 일부만 오래 보관하려는 경우에는 원본 전체를 계속 참조할 수 있으므로, 독립된 목록이 필요하면 `new ArrayList<>(list.subList(from, to))`처럼 복사합니다. 원본을 별도로 구조 변경한 뒤 뷰를 사용할 때의 동작은 명세상 정의되지 않습니다. [List.subList 문서](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#subList(int,int))

</details>
