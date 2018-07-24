---
layout: post
title: Django Rest 프레임워크 튜토리얼2
categories: [Django]
---
# 튜토리얼2: Requests and Response

이제 REST 프레임 워크의 핵심을 다루기 시작합니다. 몇 가지 필수 빌딩 블록을 소개합시다

### Request objects

REST 프레임워크는 일반 `HttpRequest`를 확장하고보다 유연한 요청 구문 분석을 제공하는 `Request`객체를 도입합니다. `Request`객체의 핵심 기능은 `request.POST`와 비슷한 `request.data`속성이지만 웹 API 작업에 더 유용합니다.

```python
request.POST  # form 데이터만 처리합니다. 'POST' method에서만 작동합니다.
request.data  # 임의이 데이터를 처리합니다. 'POST', 'PUT' 그리고 'PATCH' method에서 작동합니다.
```

### Response objects

REST 프레임워크는 렌더링되지 않은 컨텐트를 취하고 컨텐트 협상을 사용하여 올바른 컨텐트 유형을 결정하여 클라이언트에 반환하는 `TemplateResponse` 유형인 `Response` 객체도 도입합니다.

```python
return Response(data)  # 클라이언트가 요청한대로 콘텐츠 형식을 렌더링합니다.
```

### Status codes

view에서 숫자 HTTP 상태 코드를 사용한다고해서 분명히 읽을 수있는 것은 아니며, 오류 코드가 잘못되었을 때 쉽게 알 수 없습니다. REST 프레임 워크는 `status` 모듈의 `HTTP_400_BAD_REQUEST`와 같은 각 상태 코드에 대해보다 명확한 식별자를 제공합니다. 숫자 식별자를 사용하는 대신에 이것을 사용하는 것이 좋습니다.

### Wrapping API views

REST 프레임 워크는 API보기를 작성하는 데 사용할 수있는 두 개의 래퍼를 제공합니다.
1. 함수 기반 view 작업을 위한 `@api_view` 데코레이터.
2. 클래스 기반 view 작업을 위한 `APIView` 클래스입니다.

이러한 래퍼는보기에서 `Request` 인스턴스를 수신하고 `Response` 객체에 컨텍스트를 추가하여 컨텐트 협상을 수행 할 수 있도록 몇 가지 기능을 제공합니다.

래퍼는 적절할 경우 `405 Method Not Allowed` 응답을 반환하고 조작 된 입력이있는 `request.data`에 액세스 할 때 발생하는 `ParseError`예외를 처리하는 등의 동작도 제공합니다.

### Pulling it all together

우리는 더 이상 views.py에서 JSONResponse 클래스가 필요하지 않으므로 삭제하십시오. 끝나면 우리의 view를 약간 리팩토링 할 수 있습니다.

```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


@api_view(['GET', 'POST'])
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

우리의 인스턴스 뷰는 앞의 예제보다 향상되었습니다. 이제는 좀 더 간결 해졌으며 이제 Forms API를 사용하는 경우와 매우 유사합니다. 우리는 또한 응답의 의미를보다 분명하게하는 지명 된 상태 코드를 사용하고 있습니다.


```python
@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

이것은 모두 매우 익숙해야합니다 - 일반 Django보기로 작업하는 것과 많이 다르지 않습니다.

우리는 더 이상 특정 콘텐츠 유형에 대한 요청이나 응답을 명시 적으로 묶어 두지 않습니다. `request.data`는 들어오는 `json`요청을 처리 할 수 있지만 다른 형식도 처리 할 수 있습니다.찬가지로 데이터로 응답 객체를 반환하지만 REST 프레임워크가 응답을 올바른 콘텐츠 유형으로 렌더링하도록 허용합니다.

### Adding optional format suffixes to our URLs

두 가지 view 모두에 `format`키워드 인수를 추가하는 것으로 시작하십시오.

```python
def snippet_list(request, format=None):
```

```python
def snippet_detail(request, pk, format=None):
```

이제 `snippets/urls.py` 파일을 약간 업데이트하여 기존 URL 외에도 `format_suffix_patterns` 세트를 추가하십시오.

```python
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)$', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

이러한 추가 URL 패턴을 반드시 추가 할 필요는 없지만 특정 형식을 간단하고 명확하게 참조 할 수 있습니다.

### How's it looking?


```python
http http://127.0.0.1:8000/snippets/

HTTP/1.1 200 OK
...
[
  {
    "id": 1,
    "title": "",
    "code": "foo = \"bar\"\n",
    "linenos": false,
    "language": "python",
    "style": "friendly"
  },
  {
    "id": 2,
    "title": "",
    "code": "print \"hello, world\"\n",
    "linenos": false,
    "language": "python",
    "style": "friendly"
  }
]
```

Accept 헤더를 사용하여 응답의 형식을 제어 할 수 있습니다.

```python
http http://127.0.0.1:8000/snippets/ Accept:application/json  # Request JSON
http http://127.0.0.1:8000/snippets/ Accept:text/html         # Request HTML
```

또는 format suffix를 추가하여

```python
http http://127.0.0.1:8000/snippets.json  # JSON suffix
http http://127.0.0.1:8000/snippets.api   # Browsable API suffix
```

마찬가지로 우리는 `Content-Type` 헤더를 사용하여 우리가 보내는 요청의 형식을 제어 할 수 있습니다.

```python
# POST using form data
http --form POST http://127.0.0.1:8000/snippets/ code="print 123"

{
  "id": 3,
  "title": "",
  "code": "print 123",
  "linenos": false,
  "language": "python",
  "style": "friendly"
}

# POST using JSON
http --json POST http://127.0.0.1:8000/snippets/ code="print 456"

{
    "id": 4,
    "title": "",
    "code": "print 456",
    "linenos": false,
    "language": "python",
    "style": "friendly"
}
```

위의 `http`요청에 `--debug`스위치를 추가하면 요청 헤더에서 요청 유형을 볼 수 있습니다.