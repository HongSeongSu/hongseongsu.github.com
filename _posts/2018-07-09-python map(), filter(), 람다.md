---
layout: post
title: Python map(), filter(), lambda()에 대해
categories: [Python]
---

# python map(), filter()

* map(함수, 리스트)
	- 리스트의 각요소에 함수를 적용해서 새로운 배열 생성

* filter(함수, 리스트)
	- 리스트의 각 요소중에 함수에 맞는 것만 모아 추출

<br>

## map()

```python
list_a = [1, 2, 3, 4, 5, 6, 7, 8, 9]

def power(x):
    return x * x

list_b = map(power, list_a)
print(list_b)
print(list(list_b))
```

실행하면 다음과 같이 나온다

```bash
<map object at 0x7f17a724a1d0>
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

<br>

## filter()

```python
list_a = [1, 2, 3, 4, 5, 6, 7, 8, 9]

def power(x):
    return x * x

def under_3(x):
    return x < 3

list_b = map(power, list_a)
list_c = filter(under_3, list_a)
print(list_c)
print(list(list_c))
```

실행하면 다음과 같이

```bash
<filter object at 0x7f8a42b93470>
[1, 2]
```

<br>

## lambda()

위와 같이 `map()`, `filter()`함수를 쓸때마다 함수를 따로 정의하다보면 코드가 많아졌을때 정의된 함수가 무엇을 하는지 알기 위해서 스크롤을 올려 확인하거나 검색기능을 이용해 찾아야하는 번거로움이 있습니다.
그리고 위와 같이 간다한 함수인데 두줄이나 사용하는것은 귀찮은일입니다.

그래서 사용하는 것이 `lambda()` 함수입니다.

```python
list_a = [1, 2, 3, 4, 5, 6, 7, 8, 9]

list_b = map(lambda x : x * x, list_a)
list_c = filter(lambda x : x < 3, list_a)
```