---
layout: post
title: Django Rest 프레임워크 튜토리얼1
categories: [Django]
---
# 튜토리얼1: Serialization

### 개발환경 설정

```bash
pip install django
pip install djangorestframework
```

```bash
django-admin startproject tutorial
cd tutorial

python manage.py startapp snippets
```
<br>
`INSTALLED_APPS`에 새로운 `snippets`app과 `rest_framework`app을 추가해야합니다. `tutorial/settings.py` 파일을 수정합니다.

```python
INSTALLED_APPS = (
	...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
)
```


### model 만들기

`snippets/models.py`에서 작업
```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)r
```

### Serializer class 만들기
웹 API를 시작하기 위해 가장 먼저해야 할 일은 snippet 인스턴스를 `json`과 같은 표현으로 직렬화 및 비 직렬화하는 방법을 제공하는 것입니다. 우리는 Django의 형식과 매우 유사한 serializer를 선언하여이 작업을 수행 할 수 있습니다. `snippets`디렉토리에 `serializers.py`라는 이름의 파일을 만들고 다음을 추가하십시오.

```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        Update and return an existing `Snippet` instance, given the validated data.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```
serializer 클래스의 첫 번째 부분은 serialize/deserialize 된 필드를 정의합니다. `create()` 및 `update()` 메서드는 `serializer.save()`를 호출 할 때 본격적인 인스턴스가 생성되거나 수정되는 방법을 정의합니다.

serializer 클래스는 Django `Form`클래스와 매우 유사하며 `required`, `max_length` 및 `default`와 같은 다양한 필드에 비슷한 유효성 검사 플래그를 포함합니다.

필드 플래그는 특정 상황 (예 : HTML로 렌더링 할 때)에서 직렬기를 표시하는 방법을 제어 할 수도 있습니다. 위의 `{'base_template': 'textarea.html'}`플래그는 Django `Form`클래스에서 `widget = widgets.Textarea`를 사용하는 것과 같습니다. 이 방법은 탐색 가능한 API를 표시하는 방법을 제어하는데 특히 유용합니다.

### serializer 사용해보기

```bash
python manage.py shell
```

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

snippet = Snippet(code='foo = "bar"\n')
snippet.save()

snippet = Snippet(code='print "hello, world"\n')
snippet.save()
```

이제 우리는 놀 수있는 snippet 인스턴스를 얻었습니다. 이러한 인스턴스 중 하나를 serialize하는 방법에 대해 살펴 보겠습니다.

```python
serializer = SnippetSerializer(snippet)
serializer.data
# {'id': 2, 'title': u'', 'code': u'print "hello, world"\n', 'linenos': False, 'language': u'python', 'style': u'friendly'}
```

이 시점에서 모델 인스턴스를 Python 기본 데이터 유형으로 변환했습니다. 직렬화 프로세스를 마무리하기 위해 데이터를 `json`으로 렌더링합니다.

```python
content = JSONRenderer().render(serializer.data)
content
# '{"id": 2, "title": "", "code": "print \\"hello, world\\"\\n", "linenos": false, "language": "python", "style": "friendly"}'
```

Deserializer 도 비슷합니다. First we parse a stream into Python native datatypes...

```python
from django.utils.six import BytesIO

stream = BytesIO(content)
data = JSONParser().parse(stream)
```

...then we restore those native datatypes into a fully populated object instance.

```python
serializer = SnippetSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# OrderedDict([('title', ''), ('code', 'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])
serializer.save()
# <Snippet: Snippet object>
```

Notice how similar the API is to working with forms. The similarity should become even more apparent when we start writing views that use our serializer.

모델 인스턴스 대신 querysets을 직렬화할 수도 있습니다. 그렇게 하기 위해 간단히 `many=True`플래그를 추가합니다.

```python
serializer = SnippetSerializer(Snippet.objects.all(), many=True)
serializer.data
# [OrderedDict([('id', 1), ('title', u''), ('code', u'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 2), ('title', u''), ('code', u'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 3), ('title', u''), ('code', u'print "hello, world"'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]
```

<br>

### ModelSerializers를 사용하기

`SnippetSerializer`클래스는 `Snippet`모델에도 포함된 많은 정보를 반복합니다. 코드를 좀더 간결하게 할 수 있다면 좋을것입니다.

Django가 `Form`클래스와 `ModelForm`클래스를 제공하는 것과 같은 방식으로 REST 프레임 워크는 `Serializer`클래스와 `ModelSerializer`클래스를 모두 포함합니다.

`ModelSerializer`클래스를 사용하여 serializer를 리팩터링하는 방법을 살펴 보겠습니다. `snippets/serializers.py` 파일을 다시 열고 `SnippetSerializer`클래스를 다음과 같이 바꿉니다.

```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```

serializer가 가지고있는 좋은 속성 중 하나는 serializer 인스턴스의 모든 필드를 표현할 수 있다는 것입니다. Django 쉘을 `python manage.py shell`로 열고 다음을 시도하십시오.

```python
from snippets.serializers import SnippetSerializer
serializer = SnippetSerializer()
print(repr(serializer))
# SnippetSerializer():
#    id = IntegerField(label='ID', read_only=True)
#    title = CharField(allow_blank=True, max_length=100, required=False)
#    code = CharField(style={'base_template': 'textarea.html'})
#    linenos = BooleanField(required=False)
#    language = ChoiceField(choices=[('Clipper', 'FoxPro'), ('Cucumber', 'Gherkin'), ('RobotFramework', 'RobotFramework'), ('abap', 'ABAP'), ('ada', 'Ada')...
#    style = ChoiceField(choices=[('autumn', 'autumn'), ('borland', 'borland'), ('bw', 'bw'), ('colorful', 'colorful')...
```

`ModelSerializer`클래스는 특히 마술 같은 것을하지 않는다는 것을 기억하는 것이 중요합니다. 간단히 말해서 serializer 클래스를 만드는 shortcut입니다.
* An automatically determined set of fields.
* `create()` 및 `update()` 메서드에 대한 간단한 기본 구현입니다.

<br>

### Writing regular Django views using our Serializer

새로운 Serializer 클래스를 사용하여 API 뷰를 작성하는 방법을 살펴 보자. 잠시 동안 우리는 REST 프레임 워크의 다른 기능들을 사용하지 않을 것이고, 우리는보기를 일반적인 Django views를 쓴다.

`snippets/views.py`을 수정한다.

```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
```

```python
@csrf_exempt
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)
```

CSRF 토큰을 가지지 않을 클라이언트에서 이 view로 POST를 할 수 있기를 원한다면 view를 `csrf_exempt`로 표시해야합니다.이것은 당신이 일반적으로하고 싶어하는 것이 아니며, REST 프레임 워크 뷰는 실제로 이것보다 더 현명한 동작을 사용하지만, 지금 우리의 목적을 위해 할 것입니다.

```python
@csrf_exempt
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```

마지막으로 이러한 view를 연결해야합니다. `snippets/urls.py` 파일을 만듭니다.

```python
from django.conf.urls import url, include

urlpatterns = [
    url(r'^', include('snippets.urls')),
]
```

<br>

### Testing our first attempt at a Web API

Django 개발 서버를 실행한다

```bash
python manage.py runserver

Validating models...

0 errors found
Django version 1.11, using settings 'tutorial.settings'
Development server is running at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

다른 터미널에서 서버를 테스트 할 수 있다.

API를 `curl`나 `httpie`로 테스트 할거다. Httpie는 Python으로 작성된 사용자 친화적 인 http 클라이언트다.

```bash
pip install httpie
```

```bash
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
