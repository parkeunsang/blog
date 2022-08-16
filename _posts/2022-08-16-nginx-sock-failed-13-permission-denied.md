---
title: 'Nginx sock failed (13: Permission denied) 에러 해결'
layout: post
date: '2022-08-16 21:57:35'
author: Edward Park
categories:
- Dummy
tags:
- Nginx
cover: "/assets/instacode.png"
---

## 배경
Docker환경(Python:3.7 이미지 사용)에서 Django 프로젝트를 **nginx와** **uwsgi를** 이용해 배포하는 과정에서 **502 Bad Gateway** 에러가 발생했다.[실습가이드 참조](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html#some-notes-about-this-tutorial)
<img src="/blog/post_images/extra/502.png" title="502error">
실습 가이드에서 친절하게 에러 로그를 쌓고있는 위치를 알려주었고, `/var/log/nginx/error.log` 파일에서 로그를 보니 아래와 같은 에러로 Bad Gateway문제가 발생했음을 알 수 있었다.
```
connect() to unix:///root/hello/hello.sock failed (13: Permission denied)
while connecting to upstream, client: 172.17.0.1, server: localhost, request:
"GET / HTTP/1.1", upstream: "uwsgi://unix:///root/hello/hello.sock:", host: "localhost:8000"
```
소켓을 연결하는 과정에서 권한 문제가 발생했고, 실습가이드에서 위와같은 문제가 발생시 소켓 접근권한을 666으로 변경하면 해결될 것이라고 했지만 여전이 같은 문제가 발생했다.

## 해결
구글링을 해보니 uid를 www-data로 설정해라, setenforce 명령어를 이용해 SELinux를 Permissive 모드로 바꿔라 등의 방법이 있었는데 나에게는 모두 적용이 되지 않았다. <br>
그러던 중 스택오버플로우에서 **폴더 권한**이 문제라는 댓글을 보게 되었고, 아차 싶었다. 소켓이 위치한 폴더가 **root**였던 것이다.<br>
Python 3.7 Docker이미지의 홈 디렉토리는 /root 였고, 그냥 평소대로 홈 디렉토리에서 작업했던 게 이런 문제를 발생시켰다. 그래서 소켓위치를 root폴더가 아닌 다른 폴더에 위치시킨 후 uwsgi연결을 하니 정상적으로 접근이 가능했다.<br>
평소에 영어 읽기가 귀찮아서 스택오버플로우 등의 사이트 내용을 대충 훑기만 했었는데 좀더 꼼꼼히 읽는 습관을 들여야겠다.
