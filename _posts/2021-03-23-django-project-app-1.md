---
title: "[Django] 시작하기 - project 및 app 생성(1)"
layout: post
date: '2021-03-23 23:52:22'
author: Edward Park
categories:
- Django
tags:
- Django
cover: "/assets/instacode.png"
---

## What is Django
Django는 Python을 기반으로 하는 웹개발 프레임웨크이다.
## Why use Django
- 개발 속도가 빠르다.
- 디버깅이 수월하며, 비교적 쉽다.

## 시작하기
### 환경 세팅
1. 가상환경 생성<br>
Django에서는 1개의 프로젝트당 1개의 가상환경을 설정하는것을 지향한다.
```Shell
$ python -m venv venv
```
2. 가상환경 실행
```Shell
$ source venv/bin/activate  # window에서는 source venv/Scripts/activate 명령어를 통해 실행가능하다.
```
3. Django 설치(pip 사용)
```Shell
$ pip install django
$ pip freeze > requirement.txt  #  라이브러리 환경을 저장, 배포시 활용
```

### 프로젝트 생성
```Shell
$ django-admin startproject project_name .  # project_name은 프로젝트 이름으로 임의로 설정가능하다
```

### 앱 생성
하나의 프로젝트는 여러개의 앱(각각의 기능을 가진 것)으로 구성된다.
```Shell
$ python manage.py startapp app_name  # app_name은 앱 이름으로 임의로 설정가능하다.
```
만들어진 폴더의 구조는 아래와 같다.
<img src="/blog/post_images/django1_1.png" title="django">
그리고 앱을 생성할 때 마다 project_name/settings.py 에서 INSTALLED_APPS에 앱 이름을 추가해 줘야 해당앱을 인식할 수 있다.
<img src="/blog/post_images/django1_2.png" title="django">
