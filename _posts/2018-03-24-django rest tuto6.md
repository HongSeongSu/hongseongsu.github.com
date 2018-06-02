---
layout: post
title: Django Rest 프레임워크 튜토리얼6
categories: [Python]
---
# 튜토리얼6: ViewSets & Routers

REST 프레임워크에는 개발자가 API의 상태 및 상호 작용을 모델링하는 데 집중하고 일반적인 규칙에 따라 URL 구성을 자동으로 처리 할 수 있도록 해주는 `ViewSets`을 다루는 추상화가 포함되어 있습니다.

`ViewSet` 클래스는 `read` 또는 `update`와 같은 작업을 제공하고 `get` 또는 `put`과 같은 메소드 처리기는 제공하지 않는다는 점을 제외하고는 `View` 클래스와 거의 동일합니다.

`ViewSet` 클래스는 마지막 순간에 뷰 집합으로 인스턴스화 될 때, 일반적으로 URL conf를 정의하는 복잡성을 처리하는 `Router` 클래스를 사용하여 메소드 핸들러 세트에만 바인딩됩니다.

### ViewSet을 사용하기위한 리팩토링

현재의 뷰 집합을 가져 와서 뷰 집합으로 리팩터링합니다.

먼저 `UserList` 및 `UserDetail` 뷰를 단일 `UserViewSet`으로 리팩토링합시다. 우리는 두개의 뷰를 제거하고 그것들을 하나의 클래스로 대체 할 수 있습니다.

```python
from rest_framework import viewsets

class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    This viewset automatically provides `list` and `detail` actions.
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

여기에서는 `ReadOnlyModelViewSet` 클래스를 사용하여 자동으로 기본 '읽기 전용'작업을 제공합니다. 우리는 여전히 일반 뷰를 사용할 때와 마찬가지로 `queryset` 및 `serializer_class` 특성을 정확하게 설정하지만 더 이상 두 개의 별도 클래스에 동일한 정보를 제공 할 필요가 없습니다.

다음으로 `SnippetList`, `SnippetDetail` 및 `SnippetHighlight`뷰 클래스를 대체 할 것입니다. 세가지 뷰를 제거하고 다시 단일 클래스로 대체 할 수 있습니다.

```python
from rest_framework.decorators import action
from rest_framework.response import Response

class SnippetViewSet(viewsets.ModelViewSet):
    """
    This viewset automatically provides `list`, `create`, `retrieve`,
    `update` and `destroy` actions.

    Additionally we also provide an extra `highlight` action.
    """
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,
                          IsOwnerOrReadOnly,)

    @action(detail=True, renderer_classes=[renderers.StaticHTMLRenderer])
    def highlight(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

이번에는 `ModelViewSet` 클래스를 사용하여 완전한 기본 읽기 및 쓰기 작업 세트를 가져 왔습니다.

`@action` 데코레이터를 사용하여 `highlight`라는 사용자 정의 액션을 만들었습니다. 이 데코레이터는 표준 `create/update/delete` 스타일에 맞지 않는 사용자 정의 엔드포인트를 추가하는데 사용될 수 있습니다.

`@action` 데코레이터를 사용하는 사용자 정의 액션은 기본적으로 `GET` 요청에 응답합니다. `POST` 요청에 응답한 작업을 원한다면 `methods` 인수를 사용할 수 있습니다.

사용자 지정 작업의 URL은 기본적으로 메서드 이름 자체에 따라 다릅니다. url을 구성하는 방법을 변경하려면 `url_path`를 데코레이터 키워드 인수로 포함 할 수 있습니다.

### Binding ViewSets to URLs explicitly

핸들러 메서드는 URLConf를 정의 할 때만 액션에 바인딩됩니다. 내부에서 진행중인 작업을 보려면 먼저 ViewSet에서 명시 적으로 일련의보기를 만듭니다.

`snippets/urls.py` 파일에서 우리는 `ViewSet` 클래스를 구체적인 뷰 세트에 바인딩합니다.

```python
from snippets.views import SnippetViewSet, UserViewSet, api_root
from rest_framework import renderers

snippet_list = SnippetViewSet.as_view({
    'get': 'list',
    'post': 'create'
})
snippet_detail = SnippetViewSet.as_view({
    'get': 'retrieve',
    'put': 'update',
    'patch': 'partial_update',
    'delete': 'destroy'
})
snippet_highlight = SnippetViewSet.as_view({
    'get': 'highlight'
}, renderer_classes=[renderers.StaticHTMLRenderer])
user_list = UserViewSet.as_view({
    'get': 'list'
})
user_detail = UserViewSet.as_view({
    'get': 'retrieve'
})
```

http 메소드를 각 뷰에 대해 필요한 액션에 바인딩하여 각 `ViewSet` 클래스에서 여러 뷰를 생성하는 방법에 주목하십시오.

이제 리소스를 concrete view로 묶었으므로 평소처럼 URL conf를 사용하여 뷰를 등록 할 수 있습니다.


```python
urlpatterns = format_suffix_patterns([
    url(r'^$', api_root),
    url(r'^snippets/$', snippet_list, name='snippet-list'),
    url(r'^snippets/(?P<pk>[0-9]+)/$', snippet_detail, name='snippet-detail'),
    url(r'^snippets/(?P<pk>[0-9]+)/highlight/$', snippet_highlight, name='snippet-highlight'),
    url(r'^users/$', user_list, name='user-list'),
    url(r'^users/(?P<pk>[0-9]+)/$', user_detail, name='user-detail')
])
```

### Using Routers

`View` 클래스보다는 `ViewSet` 클래스를 사용하기 때문에 실제로 URL을 직접 디자인 할 필요가 없습니다. 보기 및 URL에 자원을 연결하는 규칙은 Router 클래스를 사용하여 자동으로 처리 할 수 있습니다. 적절한 뷰 세트를 라우터에 등록하고 나머지는 그대로두면됩니다.

```python
from django.conf.urls import url, include
from rest_framework.routers import DefaultRouter
from snippets import views

# Create a router and register our viewsets with it.
router = DefaultRouter()
router.register(r'snippets', views.SnippetViewSet)
router.register(r'users', views.UserViewSet)

# The API URLs are now determined automatically by the router.
urlpatterns = [
    url(r'^', include(router.urls))
]
```

뷰 세트를 라우터에 등록하는 것은 url 패턴을 제공하는 것과 유사합니다. 뷰의 URL 접두어와 뷰 세트 자체라는 두 가지 인수가 포함됩니다.

우리가 사용하고있는 `DefaultRouter` 클래스는 자동으로 API 루트 뷰를 생성하기 때문에 view 모듈에서 `api_root` 메소드를 삭제할 수 있습니다.

### views와 viewsets의 절충(Trade-offs)

뷰 세트를 사용하면 정말 유용한 추상화가 될 수 있습니다. API를 통해 URL 규칙이 일관되게 유지되고 작성해야하는 코드의 양을 최소화하며 URL conf의 특성보다는 API가 제공하는 상호 작용 및 표현에 집중할 수 있습니다.

그렇다고 항상 올바른 접근 방식을 취하는 것은 아닙니다. 함수 기반 뷰 대신 클래스 기반 뷰를 사용할 때와 마찬가지로 고려해야 할 유사한 절충점이 있습니다. viewsets를 사용하는 것은 뷰를 개별적으로 작성하는 것보다 덜 명백합니다.