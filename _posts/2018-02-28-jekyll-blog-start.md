---
layout: post
title: Jekyll로 블로그 시작하기
category: post
categories: [Web]
---

{: img-center: style="text-align: center;"}
![](https://jekyllrb.com/img/logo-2x.png)
{: img-center}

# 1. Jekyll 이란

jekyll은 정적 사이트 생성기로 다양한 포맷의 원본 텍스트 파일을 템플릿 디렉토리로부터 읽어서 Markdown변환기와 Liquid 렌더러를 통해 가공하여, 웹서버에 곧바로 게시할 수 있는, 완성된 정적 웹사이트를 만들어냅니다.

또한 Github Pages의 내부엔진이기도 합니다. jekyll을 사용하면 자신의 프로젝트 페이지나 블로그, 웹사이트를 무료로 Github에 호스팅 할 수 있습니다.

<br>
<br>

## 2. Jekyll 요구사항

* OS(Linux or macOS)
* Ruby(`ruby -v`로 확인)
* RubyGems(`gem -v`로 확인)

<br>
<br>

## 3. Jekyll 설치

준비가 됐으면 Jekyll을 설치하면 된다.

```bash
gem install jekyll
```

`jekyll`이 설치가 되면
`blogname` 으로 블로그를 만듭니다.

```bash
jekyll new blogname
cd blogname
jekyll serve
```

`127.0.0.1:4000`에 들어가면 블로그를 볼 수 있다.

<br>

### Jekyll Directory structure

jekyll의 기본적인 구조는 다음과 같다.

```bash
.
├── _config.yml
├── _data
|   └── members.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.md
|   └── on-simplicity-in-technology.md
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
|   └── 2009-04-26-barcamp-boston-4-roundup.md
├── _sass
|   ├── _base.scss
|   └── _layout.scss
├── _site
├── .jekyll-metadata
└── index.html # can also be an 'index.md' with valid YAML Frontmatter
```

> Jekyll 3.2 부터는 `minima`라는 테마가 기본테마가 적용되고 `_layouts`, `_includes`, '_sass'는 theme-gem에 저장되어있습니다.
> `bundle show minima`로 테마파일이 컴퓨터에 저장되는 곳을 보여줍니다.
> >[minima thema](https://github.com/jekyll/minima) github에 가서 코드를 볼수도 있습니다.

<br>

### GITHUB PAGE로 배포하기

`github`에서 새 `repository`를 `자기아이디.github.com`으로 만듭니다. 만든후에 자신의 blog폴더에서 아래와 같은 작업을 합니다.

```bash
git init
git add .
git commit -m "blog start"
git remote add origin https://github.com/자기아이디/자기아이디.github.com.git
git push -u origin master
```

`자기아이디.github.com`으로 들어가면 블로그를 볼 수가 있습니다.
> 페이지가 구성되는데 시간이 좀 걸릴수도 있습니다. 문제가 있어서 page build가 실패했다면 github에 설정한 이메일로 메일이 옵니다.