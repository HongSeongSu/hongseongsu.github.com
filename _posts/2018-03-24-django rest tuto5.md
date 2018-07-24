---
layout: post
title: Django Rest 프레임워크 튜토리얼5
categories: [Django]
---
# 튜토리얼5: Relationships & Hyperlinked APIs

현재 API 내의 관계는 기본 키를 사용하여 표현됩니다. 이번 튜토리얼에서는 관계에 하이퍼 링크를 사용하여 API의 응집력과 발견 가능성을 향상시킬 것입니다.

### API의 루트에 대한 엔드포인트 만들기

지금은 'snippets'과 'users'에 대한 엔드포인트가 있지만 API에 대한 싱글 엔트리 포인트가 없습니다. 하나를 만들기 위해, 우리는 정규 함수 기반 뷰와 앞서 소개 한 `@api_view` 데코레이터를 사용합니다. `snippets/views.py`에 추가

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.reverse import reverse


@api_view(['GET'])
def api_root(request, format=None):
    return Response({
        'users': reverse('user-list', request=request, format=format),
        'snippets': reverse('snippet-list', request=request, format=format)
    })
```

여기서 두가지를 알아야합니다. 먼저 정규화 된 URL을 반환하기 위해 REST 프레임 워크의 `reverse` 함수를 사용합니다. 두번째로, URL 패턴은 나중에 `snippets/urls.py`에서 선언할 convenience 이름으로 식별됩니다.

### 하이라이트 된 snippet의 엔드포인트 만들기

pastebin API에서 아직 발견되지 않은 또 다른 명백한 점은 엔드포인트를 하이라이트하는 코드입니다.

다른 모든 API 엔드 포인트와 달리 JSON을 사용하지 않고 대신 HTML 표현 만 제공합니다. REST 프레임워크는 두 가지 스타일의 HTML 렌더러를 제공합니다. 하나는 템플릿을 사용하여 렌더링 된 HTML을 처리하고 다른 하나는 사전 렌더링 된 HTML을 처리하는 것입니다. 두 번째 렌더러는이 엔드포인트에 사용하려는 렌더러입니다.

코드 하이라이트 뷰를 생성 할 때 고려해야 할 또 다른 사항은 우리가 사용할 수있는 기존의 구체적인 일반 뷰가 없다는 것입니다. 우리는 객체 인스턴스가 아니라 객체 인스턴스의 속성을 리턴합니다.

구체적인 제네릭 뷰를 사용하는 대신 인스턴스를 나타내는 데 기본 클래스를 사용하고 자체 `.get()` 메서드를 만듭니다. `snippets/views.py`에 추가

```python
from rest_framework import renderers
from rest_framework.response import Response

class SnippetHighlight(generics.GenericAPIView):
    queryset = Snippet.objects.all()
    renderer_classes = (renderers.StaticHTMLRenderer,)

    def get(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)
```

평소처럼 우리가 생성 한 새로운 뷰를 URLconf에 추가해야합니다. `snippets/urls.py`에 새 API 루트에 대한 URL 패턴을 추가합니다.

```python
url(r'^$', views.api_root),
```

그리고 나서 snippet 하이라이트에 대한 URL 패턴을 추가하십시오.

```python
url(r'^snippets/(?P<pk>[0-9]+)/highlight/$', views.SnippetHighlight.as_view()),
```

### API 하이퍼링크하기

엔터티 간의 관계를 다루는 것은 웹 API 디자인의 더욱 어려운 측면 중 하나입니다. 관계를 나타 내기 위해 선택할 수있는 여러 가지 방법이 있습니다.
* 기본 키 사용.
* 엔티티 간의 하이퍼 링크 사용.
* 관련 엔티티에 고유 한 식별 슬러그 필드 사용.
* 관련 엔터티의 기본 문자열 표현 사용.
* 부모 표현 안에 관련 엔티티를 중첩합니다.
* 다른 사용자 정의 표현.

REST 프레임워크는 이러한 모든 스타일을 지원하며 정방향 또는 역방향 관계에 적용하거나 generic foreign key와 같은 사용자 지정 관리자에 적용 할 수 있습니다.

이 경우 엔티티간에 하이퍼링크 된 스타일을 사용하고 싶습니다. 이렇게하기 위해 우리는 기존 `ModelSerializer` 대신 `HyperlinkedModelSerializer`를 확장하기 위해 serializer를 수정합니다.

`HyperlinkedModelSerializer`에는 `ModelSerializer`와 다음과 같은 차이가 있습니다.
* `id` 필드는 기본적으로 포함되지 않습니다.
* `HyperlinkedIdentityField`를 사용하여 `URL` 필드를 포함합니다.
* 관계는 `PrimaryKeyRelatedField` 대신 `HyperlinkedRelatedField`를 사용합니다.

하이퍼링크를 사용하기 위해 기존의 serializer를 쉽게 다시 작성할 수 있습니다. `snippets/serializers.py`에 다음을 추가합니다.

```python
class SnippetSerializer(serializers.HyperlinkedModelSerializer):
    owner = serializers.ReadOnlyField(source='owner.username')
    highlight = serializers.HyperlinkedIdentityField(view_name='snippet-highlight', format='html')

    class Meta:
        model = Snippet
        fields = ('url', 'id', 'highlight', 'owner',
                  'title', 'code', 'linenos', 'language', 'style')


class UserSerializer(serializers.HyperlinkedModelSerializer):
    snippets = serializers.HyperlinkedRelatedField(many=True, view_name='snippet-detail', read_only=True)

    class Meta:
        model = User
        fields = ('url', 'id', 'username', 'snippets')
```

새로운 'highlight'필드도 추가했습니다. 이 필드는 `'snippet-detail'`URL 패턴 대신 `'snippet-highlight'`url 패턴을 가리키는 점을 제외하면 `url` 필드와 동일한 유형입니다.

`'.json'`과 같은 접미사 형식의 URL을 포함했기 때문에 `highlight` 필드에 접미사가있는 형식의 접미어 하이퍼 링크에 `'.html'`접미어를 사용해야한다는 것을 나타내야합니다.

### URL 패턴 이름 지정하기

하이퍼링크 된 API를 사용하려면 URL 패턴의 이름을 지정해야합니다. 이름을 지정해야하는 URL 패턴을 살펴 보겠습니다.
* API의 루트는 `'user-list'`및 `'snippet-list'`를 참조합니다.
* snippet serializer에는 `'snippet-highlight'`를 참조하는 입력란이 있습니다.
* 우리의 사용자 시리얼 라이저는 `'snippet-detail'`을 참조하는 필드를 포함합니다.
* snippet과 사용자 시리얼라이저에는 기본적으로 `'{model_name} -detail'`을 참조하는 `'url'`필드가 포함되며, 이 경우 `'snippet-detail'`과 `'user-detail'`이됩니다.

URLconf에 모든 이름을 추가 한 후 최종 `snippets/urls.py` 파일은 다음과 같아야합니다.

```python
from django.conf.urls import url, include
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

# API endpoints
urlpatterns = format_suffix_patterns([
    url(r'^$', views.api_root),
    url(r'^snippets/$',
        views.SnippetList.as_view(),
        name='snippet-list'),
    url(r'^snippets/(?P<pk>[0-9]+)/$',
        views.SnippetDetail.as_view(),
        name='snippet-detail'),
    url(r'^snippets/(?P<pk>[0-9]+)/highlight/$',
        views.SnippetHighlight.as_view(),
        name='snippet-highlight'),
    url(r'^users/$',
        views.UserList.as_view(),
        name='user-list'),
    url(r'^users/(?P<pk>[0-9]+)/$',
        views.UserDetail.as_view(),
        name='user-detail')
])
```

### Adding pagination

사용자 및 코드 snippet에 대한 목록보기는 많은 인스턴스를 반환 할 수 있으므로 결과에 페이지 매김을하고 API 클라이언트가 각 페이지를 단계별로 실행할 수 있도록해야합니다.

`tutorial/settings.py` 파일을 약간 수정하여 페이지 스타일을 사용하도록 기본 목록 스타일을 변경할 수 있습니다. 다음 설정을 추가하십시오.

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```

REST 프레임워크의 설정은 모두 `REST_FRAMEWORK`라는 이름의 단일 딕셔너리 설정에 네임 스페이스로 지정되어 있으므로 다른 프로젝트 설정과 잘 구분됩니다.

필요하다면 페이지네이션 스타일을 커스터마이징 할 수도 있지만, 이 경우 우리는 기본값을 씁니다.

