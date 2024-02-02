---
title: "jekyll minimal mistakes 기반으로 블로그 만들기"
excerpt: "jekyll minimal mistakes category 적용"
categories:
  - IT
tags:
  - 블로그
  - jekyll
  - github
date: 2024-02-02
sidebar:
  nav: "sidebar-category"
---

Github를 통해 블로그를 만들기로 결심했다.
어떤 테마를 적용하는게 좋을까 하다 심플하고 fork 수가 많은 minimal mistakes 를 적용하기로 했다.

맘에 드는 테마의 github를 방문해서 Fork 하면 된다.

이후 해당 repository에 들어가서 setting에 Repository name 변경해주면 된다.

![img1.png](..%2Fassets%2Fimg%2Fimg1.png)


이러면 기본적인건 끝

블로그의 이름을 바꾸고 싶거나, author 등을 변경하고 싶으면 _config.yml 에서 바꾸면 된다.


무튼 minimal mistakes 를 적용하고 그다음엔 카테고리를 만들고 싶었다.
블로그가 너~무 심플해서 상단에도 왼쪽사이드에도 카테고리를 넣고 싶었다.


1. _pages 안에 category-archive.md 만들기

```
---
title: "Category"
layout: categories
permalink: /categories/
author_profile: true
sidebar_main: true
sidebar:
  nav: "sidebar-category" --> 이부분 중요!
---
```

2. 카테고리로 표현하고 싶은 거를 _pages 내에 md파일 만들기

나 같은 경우는 it 라는 카테고리를 만들었고, 아래와 같이 category-it.md 파일을 만들었다.

![img2.png](..%2Fassets%2Fimg%2Fimg2.png)

category-it.md 파일내용
```
---
title: "소소한 잇"
permalink: /categories/it/
layout: category
author_profile: true
taxonomy: IT
sidebar_main: true
sidebar:
  nav: "sidebar-category"
---

```

3. _config.yml 부분 수정
```angular2html
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
      mathjax: true
      sidebar_main: true
      sidebar:
        nav: "sidebar-category"

```

4. 게시물내에 Front Matter에 아래 내용 추가
```angular2html
sidebar:
  nav: "sidebar-category"
```


카테고리가 만들어진 블로그가 지금의 블로그다.
아직도 손볼것이 많다.

그리고 하루에 하나 뭐든 좋으니 게시물을 쓸까한다. 

