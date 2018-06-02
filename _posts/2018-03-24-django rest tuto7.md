---
layout: post
title: Django Rest 프레임워크 튜토리얼7
categories: [Python]
---
# 튜토리얼7: Schemas & client libraries

스키마는 사용 가능한 API 엔드포인트, 해당 URL 및 지원하는 조작을 설명하는 machine-readable 문서입니다.

스키마는 자동 생성 문서에 유용한 도구 일 수 있으며 API와 상호 작용할 수있는 동적 클라이언트 라이브러리를 구동하는 데 사용될 수도 있습니다.

### Core API

스키마 지원을 제공하기 위해 REST 프레임워크는 Core API를 사용합니다.

Core API는 API를 설명하기위한 문서 사양입니다. 사용 가능한 엔드 포인트와 API가 노출 할 수있는 가능한 상호 작용의 내부 표현 형식을 제공하는 데 사용됩니다. 서버 측 또는 클라이언트 측 중 하나를 사용할 수 있습니다.

서버 측에서 사용할 경우 Core API를 사용하면 API가 다양한 스키마 또는 하이퍼 미디어 형식으로의 렌더링을 지원할 수 있습니다.

클라이언트 측에서 사용되면 코어 API는 지원되는 스키마 또는 하이퍼 미디어 형식을 노출하는 모든 API와 상호 작용할 수있는 동적으로 구동되는 클라이언트 라이브러리를 허용합니다.

### schema 추가

REST 프레임 워크는 명시 적으로 정의 된 스키마 뷰 또는 자동 생성 된 스키마를 지원합니다. 우리는 뷰 세트와 라우터를 사용하기 때문에 단순히 자동 스키마 생성을 사용할 수 있습니다.

API 스키마를 포함하려면 `coreapi` python 패키지를 설치해야합니다.

```bash
$ pip install coreapi
```

URL 구성에 자동 생성 된 스키마 뷰를 포함하여 API에 대한 스키마를 포함 할 수 있습니다.

```python
from rest_framework.schemas import get_schema_view

schema_view = get_schema_view(title='Pastebin API')

urlpatterns = [
    url(r'^schema/$', schema_view),
    ...
]
```

브라우저에서 `/schema/` endpoint를 방문하면 `corejson` 표현이 옵션으로 사용 가능해집니다.

`Accept` 헤더에 원하는 내용 유형을 지정하여 명령 줄에서 스키마를 요청할 수도 있습니다.

```bash
$ http http://127.0.0.1:8000/schema/ Accept:application/coreapi+json
HTTP/1.0 200 OK
Allow: GET, HEAD, OPTIONS
Content-Type: application/coreapi+json

{
    "_meta": {
        "title": "Pastebin API"
    },
    "_type": "document",
    ...
```

### Using a command line client

API가 스키마 엔트포인트를 노출하므로 동적 클라이언트 라이브러리를 사용하여 API와 상호 작용할 수 있습니다. 이를 증명하기 위해 Core API 명령 행 클라이언트를 사용 해보자.

커맨드 라인 클라이언트는 `coreapi-cli` 패키지로 제공됩니다.

```bash
$ pip install coreapi-cli
```

```bash
$ coreapi
Usage: coreapi [OPTIONS] COMMAND [ARGS]...

  Command line client for interacting with CoreAPI services.

  Visit http://www.coreapi.org for more information.

Options:
  --version  Display the package version number.
  --help     Show this message and exit.

Commands:
...
```

먼저 커맨드 라인 클라이언트를 사용하여 API 스키마를 로드합니다.

```bash
$ coreapi get http://127.0.0.1:8000/schema/
<Pastebin API "http://127.0.0.1:8000/schema/">
    snippets: {
        highlight(id)
        list()
        read(id)
    }
    users: {
        list()
        read(id)
    }
```

아직 인증되지 않았으므로 API에 대한 사용 권한을 설정 한 방법에 따라 읽기 전용 엔드포인트만 볼 수 있습니다.

명령 줄 클라이언트를 사용하여 기존 snippet을 나열 해 봅시다.

```bash
$ coreapi action snippets list
[
    {
        "url": "http://127.0.0.1:8000/snippets/1/",
        "id": 1,
        "highlight": "http://127.0.0.1:8000/snippets/1/highlight/",
        "owner": "lucy",
        "title": "Example",
        "code": "print('hello, world!')",
        "linenos": true,
        "language": "python",
        "style": "friendly"
    },
    ...
```

일부 API 엔드포인트에는 명명 된 매개 변수가 필요합니다. 예를 들어 특정 snippet의 하이라이트 HTML을 되돌리려면 ID를 제공해야합니다.

```bash
$ coreapi action snippets highlight --param id=1
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

<html>
<head>
  <title>Example</title>
  ...
```

### Authenticating our client

snippet을 생성, 편집 및 삭제하려면 올바른 사용자로 인증해야합니다. 이 경우 기본 인증만 사용합니다.

아래의 `<username>` 및 `<password>`를 실제 사용자 이름과 암호로 바꾸십시오.

```bash
$ coreapi credentials add 127.0.0.1 <username>:<password> --auth basic
Added credentials
127.0.0.1 "Basic <...>"
```

이제 스키마를 다시 가져 오면 사용할 수있는 모든 상호 작용을 볼 수 있습니다.

```bash
$ coreapi reload
Pastebin API "http://127.0.0.1:8000/schema/">
    snippets: {
        create(code, [title], [linenos], [language], [style])
        delete(id)
        highlight(id)
        list()
        partial_update(id, [title], [code], [linenos], [language], [style])
        read(id)
        update(id, code, [title], [linenos], [language], [style])
    }
    users: {
        list()
        read(id)
    }
```

그리고 snippet을 삭제하려면 다음과 같이하십시오.

```bash
$ coreapi action snippets delete --param id=7
```

개발자는 명령 줄 클라이언트뿐 아니라 클라이언트 라이브러리를 사용하여 API와 상호 작용할 수도 있습니다. Python 클라이언트 라이브러리가 이들 중 첫 번째로 사용 가능하며 곧 Javascript 클라이언트 라이브러리가 출시 될 예정입니다.

