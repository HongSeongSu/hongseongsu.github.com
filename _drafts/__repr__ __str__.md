---
layout: post
title: python ___rep___과 ___str___의 차이점?
categories: [Python]
---

>[https://stackoverflow.com/questions/1436703/difference-between-str-and-repr-in-python](http://)의 내용을 해석한것입니다.

<br>

* 기본구현은 쓸모가 없다(그렇지 않을 수도 있는 것을 생각하기는 어렵지만)
* ```__repr__```의 목표는 모호하지 않아야하는 것입니다.
* ```__str__```의 목표는 읽기 쉬운것 입니다.
* Container's ```__str__``` uses contained objects ```__repr__```

<br>

# 기본 구현은 쓸모가 없다

파이썬의 기본값은 꽤 유용하기 때문에 이것은 놀랍습니다. 그러나 이 경우 ```__repr__```에 대한 기본값은 다음과 같이 작동합니다.

```python
return "%s(%r)" % (self.__class__, self.__dict__)
```

이것은 매우 위험하다(예를들어, 객체가 서로를 참조하면 무한재귀를 하기 쉽습니다.). So Python cops out. true라는 하나의 기본값이 있습니다. ```__repr__```은 정의되고 ```__str__```은 아닌 경우 객체는 ```__str__ == __repr__```처럼 동작합니다.

이것은 간단히 말해서, 구현하는 거의 모든 객체에는 객체를 이해하는데 사용할 수 있는 기능적인 `__repr__`이 있어야 합니다. `__str__`구현은 선택 사항입니다. 예를들어 `pretty print`기능이 필요할 때 사용하십시오.

# ```__repr__```의 목표는 모호하지 않아야하는 것입니다

나는 디버거를 믿지 않는다. 디버거를 사용하는 방법을 알지도 못하고, 진지하게 사용하지도 않습니다. 