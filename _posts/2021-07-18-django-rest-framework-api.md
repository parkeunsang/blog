---
title: "[Django] Rest Framework를 이용해 API 서버 만들기"
layout: post
date: '2021-07-18 16:50:56'
author: Edward Park
categories:
- Django
tags:
- Django
cover: "/assets/instacode.png"
---

## Abstract
앞선 포스트에서 봤듯이 Django는 Front-end와 Back-end를 한번에 개발할 수 있는 framework이다. 하지만 많은 경우 Django는 서버 **개발(Back-end)의** 용도로 많이 사용된다. Front-end는 React, Vue등 보다 효율적으로 개발할 수 있는 JS기반 framework가 많기 때문이다. 대표적인 예시로 Django를 이용해 Single Page Application(SPA)를 만드는 것은 매우 비효율적일 것이다.<br>
그래서 이번 포스트에서는 Django의 Rest Framework를 이용해 간단한 **API 서버(back-end)** 를 만들어 보겠다.
## API 서버란?
URL 등을 이용해 데이터를 제공하는 부분을 의미한다. 대표적으로 REST API 아키텍쳐가 있으며 GET, POST 등의 Method를 이용해 JSON 형식으로 데이터를 전달한다.<br>
**GET** method는 단순히 데이터를 요청하는 것에 해당하고, **POST**는 서버의 DB에 데이터를 추가하는 요청, **PUT, PATCH**는 DB의 데이터를 변경하는 요청, **DELETE**는 DB의 데이터를 삭제하는 요청에 사용된다.
## Django 서버 개발
### 1. 개발 환경 구성
**Packages 설치**<br>
다음의 명령어를 통해 API 서버 개발에 필요한 패키지를 설치할 수 있다.
```Shell
pip install djangorestframework
```
**환경 세팅**<br>
`project_name/settings.py`에서 INSTALLED_APPS에 rest framework를 추가해준다.
```Python
INSTALLED_APPS = [
    'rest_framework',
    ...
]
```
### 2. DB 설계
먼저 `app_name/models.py`에서 DB 모델을 설계한다.
```Python
from django.db import models

class Question(models.Model):
    name = models.CharField(max_length=5)
    age = models.IntegerField()
```
그리고 아래의 명령어를 통해 DB의 변경을 Django에 반영해주자.<br>
```Shell
python manage.py makemigrations
python manage.py migrate
```

### 3. Rest Framework 활용
app_name폴더에 `serializers.py`라는 파일을 만든 후 DB와 API Response를 연동한다.
```Python
from rest_framework import serializers
from .models import Question

class QuestionSerializer(serializers.ModelSerializer):
    class Meta:
        model = Question
        fields = '__all__'  
```

### 4. URL 설계
이제 어떤 url로 접근해야 response를 받을 수 있을지를 설계해보자.<br>
`project_name/urls.py`
```Python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('app1/', include('app_name.urls')),
]
```
`app_name/urls.py`
```Python
from django.urls import path
from . import views

urlpatterns = [
    path('index/', views.deal_question),
]
```
이제 `http://127.0.0.1:8000/app1/index`로 요청을 보내 API 응답을 확인할 수 있다.
### 5. API 설계
마지막으로 `app_name/views.py`에서 API로 제공할 데이터(Resource)와 응답 형태(method)를 설계해보자
```Python
from rest_framework.response import Response
from .models import Question
from .serializers import QuestionSerializer
from rest_framework.decorators import api_view

@api_view(['GET', 'POST'])
def deal_question(request):
    if request.method == 'POST':
        serializer = QuestionSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)  # 생성성공시 201
        return Response(serializer.errors, status=400)
    elif request.method == 'GET':
        questions = Question.objects.all()
        serializer = QuestionSerializer(questions, many=True) 
        print('----', serializer)
        return Response(serializer.data)
```
GET으로 요청을 보내면 DB예 저장된 데이터를 응답받고, data를 포함한 POST요청을 보내면 서버의 DB에 해당 데이터를 저장할 수 있다.

### 요청 보내기
먼저 POST요청을 보내 데이터를 저장해보자. postman, javascript, Python 등 다양한 방법을 통해 요청을 보낼 수 있으며 여기서는 Python 의 requests를 이용해 요청을 보내보겠다.
```Python
import requests
url = 'http://127.0.0.1:8000/app1/index/'
data = {'name': edward, 'age': 26}
result = requests.post(url, data=data)
print(result)  # Response [201] 프린트
print(result.text)  # {"id":1, "name": "edward', "age": 26} 프린트
```
그리고 아래의 GET으로 해당 데이터가 저장되어있는지를 확인할 수 있다.
```Python
result = requests.get(url)
print(result)  # 리스트 형태로 DB에 저장된 값 확인 가능
```
상세 코드는 아래 주소에서 확인할 수 있다.<br>
[https://github.com/parkeunsang/django_blog](https://github.com/parkeunsang/django_blog)
