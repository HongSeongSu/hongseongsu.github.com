---
layout: post
title: Django Rest 프레임워크 튜토리얼4
categories: [Python]
---
# 튜토리얼4: Authentication & Permissions

현재 API에는 코드 snippet을 편집하거나 삭제할 수있는 사람에 대한 제한이 없습니다. 다음 사항을 확인하기 위해 8좀 더 발전된 행동을 원합니다.
* 코드 snippet은 항상 작성자와 연결됩니다.
* 인증된 사용자만 snippet을 만들수 있습니다.
* snippet 작성자 만 업데이트 또는 삭제할 수 있습니다.
* 인증되지 않은 요청에는 완전한 읽기 전용 액세스 권한이 있어야합니다.

### 모델에 정보 추가

우리는 `Snippet` 모델 클래스에 몇 가지 변경을 가할 것입니다. 먼저 두 개의 필드를 추가해 보겠습니다. 이 필드 중 하나는 코드 snippet을 만든 사용자를 나타내는 데 사용됩니다. 다른 필드는 코드의 강조 표시된 HTML 표현을 저장하는 데 사용됩니다.

`models.py`의 `Snippet` 모델에 다음 두 필드를 추가하십시오.

```python
owner = models.ForeignKey('auth.User', related_name='snippets', on_delete=models.CASCADE)
highlighted = models.TextField()
```

또한 모델이 저장 될 때 `pygments` 코드 강조 라이브러리를 사용하여 강조 표시된 필드가 채워지도록해야합니다.

추가 import

```python
from pygments.lexers import get_lexer_by_name
from pygments.formatters.html import HtmlFormatter
from pygments import highlight
```

이제 모델 클래스에 `.save()` 메서드를 추가 할 수 있습니다.

```python
def save(self, *args, **kwargs):
    """
    Use the `pygments` library to create a highlighted HTML
    representation of the code snippet.
    """
    lexer = get_lexer_by_name(self.language)
    linenos = 'table' if self.linenos else False
    options = {'title': self.title} if self.title else {}
    formatter = HtmlFormatter(style=self.style, linenos=linenos,
                              full=True, **options)
    self.highlighted = highlight(self.code, lexer, formatter)
    super(Snippet, self).save(*args, **kwargs)
```

모든 작업이 완료되면 데이터베이스 테이블을 업데이트해야합니다. 일반적으로 데이터베이스 마이그레이션을 만들려고하는데, 이 튜토리얼에서는 데이터베이스를 삭제하고 다시 시작해보겠습니다.

```bash
rm -f db.sqlite3
rm -r snippets/migrations
python manage.py makemigrations snippets
python manage.py migrate
```

API 테스트를 위해 몇 명의 사용자를 생성 할 수도 있습니다. 이를 수행하는 가장 빠른 방법은 createsuperuser 명령을 사용하는 것입니다.

```bash
python manage.py createsuperuser
```

### 유저 모델에 엔드포인트 추가

이제 일부 사용자가 작업 할 수있게되었으므로 해당 사용자의 표현을 API에 추가하는 것이 좋습니다. 새로운 시리얼 라이저를 만드는 것은 쉽습니다. `serializers.py`에 추가

```python
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ('id', 'username', 'snippets')
```

`'snippets'`는 User 모델에서 역방향 관계이므로 `ModelSerializer` 클래스를 사용할 때 기본적으로 포함되지 않으므로 명시 적 필드를 추가해야했습니다.

우리는 또한 `views.py`에 몇가지 보기를 추가 할 것입니다. 사용자 표현에 대해 읽기전용 보기만 사용하고자하므로 `ListAPIView` 및 `RetrieveAPIView` 제네릭 클래스 기반 뷰를 사용합니다.

```python
from django.contrib.auth.models import User


class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer


class UserDetail(generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

`UserSerializer` import

```python
from snippets.seralizers import UserSerializer
```

마지막으로 이러한 뷰를 URL conf에서 참조하여 API에 추가해야합니다. `urls.py`의 패턴에 다음을 추가하십시오.

```python
url(r'^users/$', views.UserList.as_view()),
url(r'^users/(?P<pk>[0-9]+)/$', views.UserDetail.as_view()),
```

### snippet을 유저와 연결

지금 코드 snippet을 만든 경우 snippet을 만든 사용자와 snippet 인스턴스를 연결할 방법이 없습니다. 사용자는 일련화 된 표현의 일부로 전송되지 않고 들어오는 요청의 특성입니다.

처리하는 방법은 snippet 뷰에서 `.perform_create()` 메서드를 재정의하여 인스턴스 저장을 관리하는 방법을 수정하고 들어오는 요청이나 요청 된 URL에 암시 적으로 포함 된 정보를 처리하는 것입니다.

`SnippetList` 뷰 클래스에서 다음 메서드를 추가하십시오.

```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```

serializer의 `create()` 메소드에는 요청의 유효성이 검사 된 데이터와 함께 추가 `'owner'`필드가 전달됩니다.

### Serializer 업데이트

이제 snippet은 snippet을 만든 사용자와 연결되어 있으므로 이를 반영하도록 `SnippetSerializer`를 업데이트합시다. `serializers.py`의 serializer 정의에 다음 필드를 추가합니다.

```python
owner = serializers.ReadOnlyField(source='owner.username')
```

> 내부 `Meta` 클래스의 필드 목록에 `'owner',`도 추가해야합니다.

이 분야는 꽤 흥미로운 일을하고 있습니다. `source` 인수는 필드를 채우는 데 사용되는 속성을 제어하며 직렬화 된 인스턴스의 모든 속성을 지정할 수 있습니다. 또한 위의 점으로 구분 된 표기법을 사용할 수 있습니다. 이 경우 Django의 템플릿 언어에서 사용되는 것과 비슷한 방식으로 주어진 속성을 트래버스합니다.

우리가 추가 한 필드는 형식화되지 않은 `ReadOnlyField` 클래스입니다. `CharField`, `BooleanField` 등의 다른 형식화 된 필드와 달리 형식화되지 않은 `ReadOnlyField`는 항상 읽기 전용이며 직렬화 된 표현에 사용되지만 실제로는 사용되지 않습니다. deserialize 될 때 모델 인스턴스를 업데이트하는 데 사용됩니다. 여기에서도 `CharField(read_only=True)`를 사용할 수 있습니다.

### 뷰에 필요한 권한 추가

코드 snippet이 사용자와 연결되어 있으므로 인증 된 사용자 만 코드 snippet을 생성, 업데이트 및 삭제할 수 있습니다.

REST 프레임워크에는 주어진 뷰에 액세스 할 수있는 사용자를 제한하는 데 사용할 수있는 많은 권한 클래스가 포함되어 있습니다. 이 경우 `IsAuthenticatedOrReadOnly`를 사용하면 인증 된 요청에 읽기-쓰기 액세스 권한이 부여되고 인증되지 않은 요청에는 읽기-전용 액세스 권한이 부여됩니다.

view에 다음을 import
```python
from rest_framework import permissions
```

`SnippetList`와 'SnippetDetail` 뷰 클래스에 다음 속성을 추가

```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
```

### Browsable API에 로그인 추가

현재 브라우저를 열고 탐색 가능한 API로 이동하면 더 이상 새 코드 snippet을 만들 수 없다는 것을 알 수 있습니다. 그렇게하기 위해 우리는 사용자로 로그인 할 수 있어야합니다.

프로젝트 레벨 `urls.py` 파일에서 URLconf를 편집하여 브라우저 API에 사용할 로그인보기를 추가 할 수 있습니다.

```python
from django.conf.urls import include
```

```python
urlpatterns += [
	url(r'^api-auth/', include('rest_framework.urls')),
]
```

패턴의 `r'^api-auth/'`부분은 실제로 사용하고자하는 URL이 될 수 있습니다.

이제 브라우저를 다시 열고 페이지를 새로 고침하면 페이지 오른쪽 상단에 '로그인'링크가 표시됩니다. 이전에 만든 사용자 중 하나로 로그인하면 코드 snippet을 다시 만들 수 있습니다.

몇 가지 코드 snippet을 만든 다음 '/users/' 엔드포인트로 이동하여 각 사용자의 'snippet'입력란에 각 사용자와 연결된 snippet ID 목록이 표시되는지 확인합니다.

### Object level permissions

snippets app에 `permissions.py`를 만듭니다.

```python
from rest_framework import permissions


class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    """

    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request,
        # so we'll always allow GET, HEAD or OPTIONS requests.
        if request.method in permissions.SAFE_METHODS:
            return True

        # Write permissions are only allowed to the owner of the snippet.
        return obj.owner == request.user
```

이제 `SnippetDetail` 뷰 클래스의 `permission_classes` 속성을 편집하여 snippet 인스턴스 엔드포인트에 해당 사용자 지정 권한을 추가 할 수 있습니다.

```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly,
                      IsOwnerOrReadOnly,)
```

`IsOwnerOrReadOnly`클래스를 import

```python
from snippets.permissions import IsOwnerOrReadOnly
```

이제 브라우저를 다시 열면 코드 snippet을 만든 사용자와 동일한 사용자로 로그인 한 경우 'DELETE'및 'PUT'작업 만 스 니펫 인스턴스 끝점에 나타납니다.

### API로 인증하기

이제는 API에 대한 권한 집합이 있으므로 snippet을 편집하려면 API에 대한 요청을 인증해야합니다. 우리는 어떤 **인증 클래스**도 설정하지 않았기 때문에 현재 `SessionAuthentication`과 `BasicAuthentication` 인 기본값이 적용됩니다.

웹 브라우저를 통해 API와 상호 작용할 때 우리는 로그인 할 수 있으며 브라우저 세션은 요청에 필요한 인증을 제공합니다.

프로그래밍 방식으로 API와 상호 작용할 경우 각 요청에 대해 인증 자격 증명을 명시 적으로 제공해야합니다.

인증없이 snippet을 만들려고하면 오류가 발생합니다.

```bash
http POST http://127.0.0.1:8000/snippets/ code="print 123"

{
    "detail": "Authentication credentials were not provided."
}
```

이전에 생성한 사용자 중 하나의 사용자 이름과 비밀번호를 포함시켜 요청을 성공적으로 처리 할 수 있습니다.

```bash
http -a admin:password123 POST http://127.0.0.1:8000/snippets/ code="print 789"

{
    "id": 1,
    "owner": "admin",
    "title": "foo",
    "code": "print 789",
    "linenos": false,
    "language": "python",
    "style": "friendly"
}
```

### 요약

우리는 이제 웹 API와 시스템 사용자 및 그들이 작성한 코드 snippet에 대한 사용 권한을 상당히 세분화했습니다.