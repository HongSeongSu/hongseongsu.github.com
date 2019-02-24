---
layout: post
title: Python 리스트 정렬(list sort)
categories: [Python, computerscience]
---

# list 정렬

* sorted()
    * 내장함수(built-in function)
	* 정렬된 새로운 리스트를 얻어냄

* sort()
	* 리스트의 메소드
	* 해당 리스트를 정렬함

```python
a = [3, 1, 2, 4]

print(a.sort())
print(sorted(a))

None
[1, 2, 3, 4]
```

<br>

#### 정렬의 순서를 반대로

```python
reverse=True
L2 = sorted(L, reverse=True)
L.sort(reverse=True)
```

문자열로 이루어진 리스트의 경우 정렬순서는 사전순서

문자열 길이 순서로 정렬하려면 정렬에 이용하는 키를 지정

```python
L = ['abcd', 'xyz', 'spam']
sorted(L, key=lambda x: len(x))
['xyz', 'abcd', 'spam']

L = ['spam', 'xyz', 'abcd']
sorted(L, key=lambda x: len(x))
['xyz', 'spam', 'abcd']
```

<br>

#### 정렬 키를 지정하는 또다른 예

```python
L = [{'name':'John', 'scroe':83}, {'name':'Paul', 'score':92}]

L.sort(key=lambda x: x['score'], reverse=True)
```

점수가 높은 순으로 정렬이 된다.

<br>
<br>

# 탐색알고리즘

## 선형탐색(Linear Search

리스트의 길이에 비례하는 시간 소요: O(n)
최악의 경우 모든 원소를 다 비교해 보아야 한다.

```python
def linear_search(L, x):
    i = 0
    while i < len(L) and L[i] != x:
        i += 1
    if i < len(L):
        return i
    else:
        return -1
```

## 이진탐색(Binary Search)

탐색하려는 리스트가 이미 정렬되어 있는 경우에만 적용가능
크기순으로 정렬되어 있다는 성질을 이용

한번 비교가 일어날 때마다 리스트를 반씩 줄임(divide & conquer): O(log n)

```python
def solution(L, x):
    answer = 0
    lower = 0
    upper = len(L) - 1
    while lower <= upper:
        middle = (lower +  upper) // 2
        if L[middle] == x:
            answer = middle
            return answer
        elif L[middle] < x:
            lower = L[middle] + 1
        elif L[middle] > x:
            upper = L[middle] - 1
    return -1
```
