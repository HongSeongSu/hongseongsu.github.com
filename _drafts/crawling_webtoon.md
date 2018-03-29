

`requests`을 import해서 페이지를 가져온다.

```python
import rquests

r = requests.get('http://comic.naver.com/webtoon/list.nhn?titleId=690502')
print(r)
```