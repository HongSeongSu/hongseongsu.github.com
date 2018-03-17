---
layout: post
title: jekyll로 블로그 시작하기
---

# jekyll 이란?
jekyll은 정적 사이트 생성기로 다양한 포맷의 원본 텍스트 파일을 템플릿 디렉토리로부터 읽어서 Markdown변환기와 Liquid 렌더러를 통해 가공하여, 웹서버에 곧바로 게시할 수 있는, 완성된 정적 웹사이트를 만들어냅니다.

또한 Github Pages의 내부엔진이기도 합니다. jekyll을 사용하면 자신의 프로젝트 페이지나 블로그, 웹사이트를 무료로 Github에 호스팅 할 수 있습니다.



### jekyll 요구사항

* Ruby
* RubyGems
* NodeJS

### jekyll 설치

준비가 됐으면 jekyll을 설치하면 된다.

```bash
$ gem install jekyll
```

`jekyll`이 설치가 되면
`blogname` 으로 블로그를 만듭니다.

```bash
$ jekyll new blogname
$ cd blogname
$ jekyll serve
```
`127.0.0.1:4000`에 들어가면 블로그를 볼 수 있다.


#### jekyll Directory structure

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

> jekyll 3.2 부터는 `minima`라는 테마가 기본테마가 적용되고 `_layouts`, `_includes`, '_sass'는 theme-gem에 저장되어있습니다.
> `bundle show minima`로 테마파일이 컴퓨터에 저장되는 곳을 보여줍니다.
> [minima thema](https://github.com/jekyll/minima) github에 가서 코드를 볼수도 있습니다.