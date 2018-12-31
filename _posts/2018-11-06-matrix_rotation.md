---
layout: post
title: 알고리즘 - 행렬회전
categories: [algorism, python]
---

# 파이썬으로 푸는 행렬회전

## 문제 설명

이 문제에서는 행렬을 구성하는 16 개의 자연수(99 이하)와 회전 숫자를 지정하는 1 개의 자연수(99 이하)가 주어집니다.
16 개의 정수로 구성되는 4 x 4 행렬의 “외곽” 성분을 시계 방향으로 n 회 만큼 회전시키는 코드를
구현하고, 회전시킨 행렬의 성분을 출력하세요.
(INPUT 과 OUTPUT 의 행렬 성분의 순서는 1 행 1 열, 1 행 2 열, 1 행 3 열, 1 행 4 열, 2 행 1 열, 2 행 2 열, ... , 4 행
3 열, 4 행 4 열입니다. 아래의 예시를 참고하세요.)

## INPUT 예시

1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 2

## OUTPUT 예시

9, 5, 1, 2, 13, 6, 7, 3, 14, 10, 11, 4, 15, 16, 12, 8

## OUTPUT 설명

해당 INPUT 으로 구성되는 행렬은 아래와 같습니다.

|1|2|3|4|
|--|--|--|--|
|5|6|7|8|
|9|10|11|12|
|13|14|15|16|

이 행렬의 외곽 성분을 시계 방향으로 1 회 회전하면 아래와 같은 행렬을 얻을 수 있습니다.

|5|1|2|3|
|--|--|--|--|
|9|6|7|4|
|13|10|11|8|
|14|15|16|12|

이 행렬의 외곽 성분을 다시 한 번 시계 방향으로 회전하면 아래와 같은 행렬을 얻을 수 있습니다.

|9|5|1|2|
|--|--|--|--|
|13|6|7|3|
|14|10|11|4|
|15|16|12|8|

이제 이 행렬의 성분을 출력하면 아래의 OUTPUT 을 얻게 됩니다.

9, 5, 1, 2, 13, 6, 7, 3, 14, 10, 11, 4, 15, 16, 12, 8

## 문제풀이

1. 입력된 값에서 행렬인 숫자와 회전하는 횟수를 구한다.
2. 외곽 성분들의 좌표를 회전하는 순서대로 구해서 리스트에 저장한다.
3. 입력된 숫자들을 행렬로 변환한다.
4. 만든 행렬에서 회전하는 순서 좌표의 값들을 리스트로 가져온다.
5. 가져온 값들을 회전하는 횟수만큼 회전시킨다.
6. 회전시킨 값들을 행렬에 넣는다.
7. 행렬을 리스트로 변환한다.

```python
def matrix_rotation(nums):
    input_nums = nums[:-1]
    rotation_count = nums[-1]

    ns = int(len(input_nums) ** 0.5)
    rotation_pos = []

    for i in range(ns):
        rotation_pos.append((0, i))
    for i in range(ns - 2):
        rotation_pos.append((i + 1, ns - 1))
    for i in range(ns, 0, -1):
        rotation_pos.append((ns - 1, i - 1))
    for i in range(ns - 2, 0, -1):
        rotation_pos.append((i, 0))

    matrix = [[0 for col in range(ns)] for row in range(ns)]
    i = 0
    for x in range(ns):
        for y in range(ns):
            matrix[x][y] = input_nums[i]
            i += 1

    src_number = [matrix[x][y] for x, y in rotation_pos]
    dst_number = src_number[-rotation_count % len(rotation_pos):] + src_number[:-rotation_count % len(rotation_pos)]

    for i, pos in enumerate(rotation_pos):
        x, y = pos
        matrix[x][y] = dst_number[i]

    result = list()
    for row in matrix:
        result.extend(row)

    return result


print(matrix_rotation([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 2]))
```