---
layout: post
title: Django Rest 프레임워크 튜토리얼3
categories: [Python]
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