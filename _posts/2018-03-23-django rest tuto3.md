---
layout: post
title: Django Rest 프레임워크 튜토리얼3
categories: [Django]
---
# 튜토리얼3: Class-based Views

함수 기반보기 대신 클래스 기반 view를 사용하여 API view를 작성할 수도 있습니다. 여기서 알 수 있듯이 코드는 일반적인 기능을 재사용 할 수있는 강력한 패턴이며 코드를 DRY 상태로 유지하는 데 도움이됩니다.

### 클래스 기반 뷰를 사용하여 API 다시 작성

먼저 루트 view를 클래스 기반 view로 다시 작성합니다. 이 모든 것이 관련된 것은 `views.py`의 리팩토링입니다.

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from django.http import Http404
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status


class SnippetList(APIView):
    """
    List all snippets, or create a new snippet.
    """
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

앞의 경우와 매우 비슷하게 보입니다. 하지만 서로 다른 HTTP 메소드를 더 잘 구분할 수 있습니다. 또한 `views.py`에서 인스턴스 view를 업데이트해야합니다.

```python
class SnippetDetail(APIView):
    """
    Retrieve, update or delete a snippet instance.
    """
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

우리는 클래스 기반 뷰를 사용하고 있기 때문에 `snippets/urls.py`를 약간 리팩토링해야합니다.

```python
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.SnippetList.as_view()),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

### mixins 사용하기

클래스 기반 뷰를 사용하여 얻은 큰 이점 중 하나는 재사용 가능한 비트를 쉽게 구성 할 수 있다는 것입니다.

지금까지 사용해 온 create/retrieve/update/delete 작업은 모델에서 지원하는 모든 API 지원 뷰와 매우 유사합니다. 이러한 공통된 동작은 REST 프레임워크의 mixin 클래스에서 구현됩니다.

mixin 클래스를 사용하여 뷰를 구성하는 방법을 살펴 보겠습니다. 여기에 우리의 `views.py` 모듈이 있습니다.

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics

class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

우리는 잠시 시간을내어 여기에서 무슨 일이 일어나고 있는지 정확하게 살펴볼 것입니다. `GenericAPIView`를 사용하고 `ListModelMixin` 및 `CreateModelMixin`을 추가하여 뷰를 작성합니다.

```python
class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

`GenericAPIView` 클래스를 사용하여 핵심 기능을 제공하고 `.retrieve()`, `.update()` 및 `.destroy()` 액션을 제공하기 위해 mixins를 추가합니다.

### generic class-based views 사용하기

mixin 클래스를 사용하여 이전보다 약간 적은 코드를 사용하기 위해 뷰를 다시 작성했지만 한 걸음 더 나아갈 수 있습니다. REST 프레임 워크는 이미 혼합 된 일반 뷰 집합을 제공하여 우리의 `views.py` 모듈을 더 많이 줄일 수 있습니다.

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import generics


class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```