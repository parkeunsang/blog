---
title: Certbot 404 에러 해결(feat. Docker, Django, Nginx)
layout: post
date: '2024-02-05 13:54:00'
author: Edward Park
categories:
- Django
tags:
- Docker
- Django
- Certbot
cover: "/assets/instacode.png"
---

## 에러 발생
- Ref: [Deploying a Django Application with Docker, Nginx, and Certbot](https://medium.com/@akshatgadodia/deploying-a-django-application-with-docker-nginx-and-certbot-eaf576463f19)<br>
- 문제 발생 단계: 위 블로그를 참고해 Docker compose를 활용하여 Django, Nginx, Certbot 컨테이너를 띄우고, SSL 인증을 하는 과정에서 `docker-compose run --rm certbot certonly --webroot --webroot-path=/var/www/certbot --email your_email@example.com --agree-tos --no-eff-email -d {my_domain}` 명령어로 인증서, 인증키를 발급하는 과정에서 문제 발생<br>
- 에러 메시지: `Invalid response from http://{my_domain}/.well-known/acme-challenge/{value}: 404`

## 원인 파악
- 404는 페이지가 없다는 내용인데, .well-knwon~ 이부분은 자동으로 생성되는 거라 not found가 나올 이유가 없음
- {value}에 해당하는 부분은 16진수 문자열로, certbot에서 생성된 id값 같은데, docker-compose.yml 파일을 가만 보니 certbot과 연동되는 부분이 없었음(**certbot 컨테이너와 volume을 공유하지 않고 있었음**)

## 해결
- 기존 `docker-compose.yml`
```docker
version: '3'
services:
  backend:
    build: .
    command: sh -c "python3 manage.py runserver 0.0.0.0:8001"
    restart: always
    expose:
      - "8000"
    volumes:
      - ./:/app
      - /var/www/static/:/var/www/static/
      - /var/certbot/conf:/etc/letsencrypt/:ro
    env_file:
      - .env

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/www/static/:/var/www/static/
      - ./nginx/conf.d/:/etc/nginx/conf.d/
    depends_on:
      - backend

  certbot:
    image: certbot/certbot:latest
    volumes:
      - /var/certbot/conf:/etc/letsencrypt/:rw
      - /var/certbot/www/:/var/www/certbot/:rw
    depends_on:
      - nginx
```

- 수정 `docker-compose.yml`: nginx의 volumes에 certbot에서 사용하고있는 volume을 추가하여 해결
```docker
...
  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/www/static/:/var/www/static/
      - ./nginx/conf.d/:/etc/nginx/conf.d/
      - /var/certbot/conf:/etc/letsencrypt/:rw
      - /var/certbot/www/:/var/www/certbot/:rw
    depends_on:
      - backend
...
```
