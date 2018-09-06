---
layout: post
title: Python으로 구현한 큐(Queue))
categories: [Python, computerscience]
---

# 큐(Queue)

자료(data element)를 보관할 수 있는 선형구조.
단, 넣을 때는 한쪽 끝에서 밀어 넣어야 하고(인큐enqueue 연산) 꺼낼 때는 반대 쪽에서 뽑아 꺼내야하는(디큐dequeue 연산) 제약이 있다.

선입선출(FIFO - First in First Out)특징을 가지는 선형자료 구조

<br>
<br>

## 큐의 추상적 자료구조 구현

<br>

### 1. 배열(array)를 이용해 구현

Python 리스트와 메서드들을 이용

<br>

### 2. 연결리스트(linked list)를 이용하여 구현

구현해 놓은 양방향 연결 리스트 이용

<br>

### 3. 연산의 정의

* size() - 현재 큐에 들어 있는 데이터 원소의 수를 구함
* isEmpty() - 현재 큐가 비어 있는지를 판단
* enqueue(x) - 데이터 원소 x를 큐에 추가
* dequeue() - 큐의 맨앞에 저장된 데이터 원소를 제거(또는 반환)
* peek() - 큐의 맨앞에 저장된 데이터 원소를 반환(제거하지 않음)

<br>

### 4. 배열로 구현한 큐

```python
class ArrayQueue:

	# 빈큐를 초기화
    def __init__(self):
        self.data = []

    # 큐의 크기를 리턴
    def size(self):
        return len(self.data)

	# 큐가 비어있는지 확인
    def isEmpty(self):
        return self.size() == 0

	# 데이터 원소를 추가
    def enqueue(self, item)
        self.data.append(item)

    # 데이터 원소를 삭제(리턴)
    def dequeue(self):
        return self.data.pop(0)

	# 큐의 맨앞에 원소 반환
    def peek(self):
        return self.data[0]
```

<br>

### 5. 배열로 구현한 큐의 연산 복잡도

| 연산 | 복잡도 |
|--------|--------|
| size() | O(1) |
| isEmpty() | O(1) |
| enqueue() | O(1) |
| dequeue() | O(n) |
| peek() | O(1) |

