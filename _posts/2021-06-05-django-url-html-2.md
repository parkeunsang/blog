---
title: "[Django] url과 html파일 매칭 시키기 (2)"
layout: post
date: '2021-06-06 00:29:58'
author: Edward Park
categories:
- Django
tags:
- Django
cover: "/assets/instacode.png"
---

## 개요
Django를 이용한 웹 앱 개발은 MVC 패턴을 따른다.<br>
- M(model): models.py, 어플리케이션에 필요한 DB 구조
- V(view): template/, 사용자에게 보여지는 부분(html file 등)
- C(controller): views.py, DB에 접근해 데이터를 조작+계산 과정을 거친 후 html file과 연결

이번 포스트에서는 Django를 이용해 url을 설계하고, 해당 url로 접근시 사전에 정의된 간단한 html파일을 반환하는 작업을 해볼 예정이다.<br>
과정은 크게  **url 설계 - C(controller)정의 - V(view)정의** 순서로 나눌 수 있다.<br>
## 사전 세팅
사전 환경 세팅은 [이전 포스트](https://parkeunsang.github.io/blog/django/2021/03/23/django-project-app-1.html) 참고<br>
이번 포스트에서 생성한 파일들을 포함하는 최종 폴더 구조는 아래와 같다.
<img src="/blog/post_images/django2_1.png" title="폴더구조">
또한 서버를 실행시키기 위해서는 프로젝트폴더(project_name), 앱 폴더(app_name), manage.py가 위치한 폴더에서 아래와 같은 명령어를 입력하면 된다. 
`$ python manage.py runserver`
**참고 : 이때 $는 terminal에서 명령어를 실행시킴을 의미하며 코드에 따로 $를 입력할 필요는 없다.** terminal환경은 git bash, vscode, pycharm 등에서 접속 가능하다.
## url 설계
`project_name/urls.py`
```Python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('app1/', include('app_name.urls')),
]
```
기본적으로 프로젝트를 시작하면 `admin` url은 자동으로 생성된다. 즉 `http://127.0.0.1:8000/admin` 에 접속하면 관리자 페이지가 나오게 된다.<br>
1개의 프로젝트에는 여러개의 **app**이 있고, 각각의 app마다 base url을 지정해 사용할 수 있다. 예를들어 `board`라는 앱과 `user` 라는 앱이 있고, board에는 글쓰기(create), 글 조회하기(detail)기능이 있으며 user에는 로그인(login), 회원가입(signup) 기능이 필요화다고 가정하자. 이럴경우 url은 다음과 같이 설계할 수 있을 것이다.<br>
- `http://127.0.0.1:8000/board/create`, `http://127.0.0.1:8000/board/detail` 
- `http://127.0.0.1:8000/user/login`, `http://127.0.0.1:8000/user/signup`
이때 board, user와 같이 앱 정보에 해당하는 url은 `project_name/urls.py`에서 정의하고, create, detail과 같이 앱의 기능에 해당하는 url은 `app_name/urls.py`에서 정의한다(이를 지키지않는다고 해서 앱이 작동하지 않는것은 아니다).<br>
그래서 위의 코드에서 `path('app1/', include('app_name.urls')),` 를 살펴보면, app1은  board, user와 같은 앱 이름에 해당하는 url이고, include안의 내용은 `app_name.urls`에 있는 내용(예를들어 create, detail과 같은것)들을 탑재하겠다는 뜻이다.<br>
app_name폴더안의 urls.py는 프로젝트생성시 **존재하지 않기**때문에 파일을 만든 후 내용을 채워넣어야한다. 여기서는 아주 기본적인 기능만 넣어보자.
`app_name.urls.py`
```Python
from django.urls import path
from . import views

urlpatterns = [
    path('index/', views.index),
]
```
이렇게 두개의 urls.py 파일을 만들면 `http://127.0.0.1:8000/app1/index/` 에 접근할 수 있을 것이다.<br>
두번째 urls.py에서 views를 import한다고 나와있는데, 아직 views안의 내용을 채워넣지 않았기 때문에 서버를 실행시킨 후 위 url에 접근하면 에러가 발생한다.
## controller(views.py)  정의
`app_name/views.py`
```Python
from django.shortcuts import render

# Create your views here.
def index(request):
    return render(request, 'index.html')
```
`app_name.urls.py` 에서 views.index를 호출하므로 index라는 함수를 만들어 준다. 이때 **request**는 필수로 받아야하는 인자이다.<br>
render부분에도 첫번째 인자로 request가 필수로 들어가야하며 두번째 인자인 `index.html`은 아직까지 생성하지않은 templates 폴더에 위치한 html 파일이다.

## view(template/) 정의
`app_name/templates/index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    index page
</body>
</html>
```
app_name폴더에서 templates라는 폴더를만들고, 안에 index.html파일을 생성해 간단한 내용을 채웠다. <br>
이제 서버를 실행시켜 url에 접근해 해당 html파일이 표시되는지 확인해보자.<br>
`python manage.py runserver` 명령어를 실행시키면 아래와 같이 서버가 실행되고있다는 표시를 확인할 수 있다.
<img src="/blog/post_images/django2_2.png" title="runserver">
그리고 해당 url로 접근하면 정의된 html이 나타남을 확인할 수 있을 것이다.
<img src="/blog/post_images/django2_3.png" title="chrome">
[전체 코드 참고(git)](https://github.com/parkeunsang/django_blog)
