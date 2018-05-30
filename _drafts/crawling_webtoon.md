```
```

# 크롤링을 위한 준비물

```bash
pip install beatifulsoup4

pip install requests

pip install lxml
```

# 페이지 크롤링 해보기

`requests`을 import해서 페이지를 가져온다.

```python
import rquests

r = requests.get('http://comic.naver.com/webtoon/list.nhn?titleId=690502')
print(r)
```

```bash
python crawl.py
<Response [200]>
```

url 파라미터 추가해서 출력

```python
import requests

r = requests.get('http://comic.naver.com/webtoon/list.nhn?titleId=690502')
print(r)

print(r.url)
```

```bash
python crawl.py
<Response [200]>
http://comic.naver.com/webtoon/list.nhn?titleId=690502
```