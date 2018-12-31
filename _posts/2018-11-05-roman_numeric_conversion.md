---
layout: post
title: 알고리즘 - 로마 숫자 변환
categories: [algorism, python]
---

# 파이썬으로 푸는 로마 숫자 변환

## 문제 설명

이 문제에서는 1 이상 100 이하의 자연수 1 개가 주어집니다. 이 자연수를 로마 숫자로 변환하여 출력하는 코드를
구현하세요.

## INPUT 예시

85

## OUTPUT 예시

LXXXV

## OUTPUT 설명

로마 숫자에서 85 에 해당되는 문자열은 LXXXV 이므로 이를 출력합니다.

## 문제풀이

1. 100보다 작은 수에 대해서 입력된 숫자들을 로마숫자로 변환
2. 100, 90, 50, 40, 10, 9, 5, 4, 1 의 순서대로 숫자와 로마숫자 리스트를 만든다.
3. 입력된 숫자를 숫자리스트의 값들로 100부터 나누어 몫을 구한다.
4. 구한 몫은 해당 값의 로마숫자가 표시되는 횟수이다.
5. 몫을 해당 로마숫자에 곱해 result 문자열에 더한다.
6. 로마숫자가 더해진 만큼을 입력된 숫자에서 빼준다.

```python
def roman_numeric_conversion(n):
    if n > 100:
        raise ValueError("Value must be less than 100.")

    num = (100, 90, 50, 40, 10, 9, 5, 4, 1)
    roman_num = ('C', 'XC', 'L', 'XL', 'X', 'IX', 'V', 'IV', 'I')

    result = ""
    for i in range(len(num)):
        count = int(n / num[i])
        result += roman_num[i] * count
        n -= num[i] * count
    return result


print(roman_numeric_conversion(85))

# for i in range(1, 101):
#     print(i, roman_numeric_conversion(i))
```
