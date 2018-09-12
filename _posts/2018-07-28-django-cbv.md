---
layout: post
title: Django CBV(Class Based View)
categories: [Python, Django]
---

# Django Cbv

## View?

* 뷰의 정체는 호출가능한 객체(Callable Object)
	* 첫번째 인자로 HttpRequests 인스턴스를 받는다
	* 리턴값으로 HttpResponse 인스턴스를 리턴해야한다.

```python
# myapp/views.py
def about(request):
    return HttpResponse('안녕하세요')
```

```python
# myapp/urls.py
from . import views

urlpatterns = [
    url(r'^about/$', views.about, name='about'),
]
```

## 함수 기반 뷰/클래스 기반 뷰

* 함수 기반 뷰(Function Based View)
	* 함수로 구현한 뷰
* 클래스 기반 뷰(Class Based View)
	* 클래스로 함수를 구현한 뷰

```python
# 함수 기반 뷰
def about(request):
    return HttpResponse('안녕하세요')
```

```python
# 클래스 기반 뷰
class AboutView(object):
    @classmethod
    def as_view(cls, message)
        def view_fn(request):
            return HttpResponse(message)
        return view_fn

# 클래스 기반 뷰를 통해 만드러낸 함수
about = AboutView.as_view('안녕하세요')
```

## 클래스의 3가지 함수 형식

1. instance method: instance를 통한 호출. 첫번째 인자로 instance가 자동 지정(self에 대입)
2. class method: class를 통한 호출. 첫번째 인자가 class로 자동지정(cls에 대입)
3. static method: class를 통한 호출. 자동으로 지정되는 인자가 없음. 활용은 class method와 동일

```python
class Person(object):
    def __init__(self, name, country):
        self.name = name
        self.country = country

    def say_hello(self):
        return '나는 {}이고, {}에서 왔어' .format(self.name, self.country)

    @staticmethod
    def new_korean1(name):
        return Person(name, 'Korea')
    
    @classmethod
    def new_korean2(cls, name):
        return cls(name, 'Korea')
```

# Class Based View

CBV는 FBV를 만들어주는 클래스
* as_view 클래스 함수를 통해 뷰 함수를 생성

장고 기본 CBV 패키지 위치 : `django.views.generic`
* CBV는 generic으로 쓸 뷰

