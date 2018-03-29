---
layout: post
title: python 네트워크
categories: [Python]
---


# Lambda Functions
파이썬에서 Lambda는 한줄짜리, 버려지는 함수를 만들기 위한 문법입니다. def...():와 들여쓰기 블록을 쓰지 않아도 됩니다.

```python
>>> myfn = lambda x: x+1
>>> myfn(2)
3
>>> myfn(5)
6
>>> adder = lambda x,y: x + y
>>> adder(3, 2)
5
```
우리의 경우 우리는 이 함수를 사용하여 다른 방법으로 즉시 실행될 코드를 인수로 전달할 수있는 함수로 변환하고 나중에 실행하고 여러 번 실행할 수 있습니다.
```python
>>> def addthree(x):
...     return x + 3
...f
>>> addthree(2)
5
>>> myfn = lambda: addthree(2) # note addthere is not called immediately here
>>> myfn
<function <lambda> at 0x7fec9a056bf8>
>>> myfn()
5
```