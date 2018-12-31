---
layout: post
title: 알고리즘 - 빈도순 정렬
categories: [algorism, python]
---

# 파이썬으로 푸는 빈도순 정렬

## 문제 설명

이 문제에서는 1 개 이상 99 개 이하의 자연수가 주어집니다. 이 자연수들을 빈도순으로 오름차순으로 정렬하고,
빈도가 동일할 때는 내림차순으로 정렬하여 출력하는 코드를 구현하세요.

### INPUT 예시

3, 8, 8, 3, 2, 8, 1, 2, 4, 56

### OUTPUT 예시

56, 4, 1, 3, 3, 2, 2, 8, 8, 8

### OUTPUT 설명

주어진 INPUT 에 등장하는 숫자와 그 빈도는 아래와 같습니다.
3 2 회
8 3 회
2 2 회
1 1 회
4 1 회
56 1 회

빈도가 1 회로 가장 작은 1, 4, 56 을 내림차순으로 정렬하면 다음과 같습니다.

56, 4, 1

다음으로 빈도가 2 회인 3, 2 를 내림차순으로 정렬하면 다음과 같습니다.
3, 2

마지막으로 빈도가 3 회인 숫자는 8 하나 뿐입니다.
이를 모두 연결하면 아래와 같은 OUTPUT 을 얻게 됩니다.

56, 4, 1, 3, 3, 2, 2, 8, 8, 8

<br>
<br>

## 문제풀이

1. Couter를 사용해 입력된 값들을 딕셔너리로 만든다.(num: count, 숫자: 빈도수)
2. 딕셔너리 값을 빈도수로 오름차순으로 정렬하고 빈도수가 같을 경우 숫자의 내림차순으로 정렬한다.(count, -num)
3. 정렬된 결과물에서 빈도수를 고려하여 리스트를 만든다.

```python
from collections import Counter


def sort_by_frequency(n):
    most_n = Counter(n)
    most_n_list = sorted((count, -num) for num, count in most_n.items())
    result = []

    for count, num in most_n_list:
        for i in range(count):
            result.append(-num)

    return result


print(sort_by_frequency([3, 8, 8, 3, 2, 8, 1, 2, 4, 56]))
```