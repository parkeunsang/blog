---
title: Docker 환경 Jekyll port 포워딩 문제
layout: post
date: '2023-07-10 00:53:39'
author: Edward Park
categories:
- Dummy
tags:
- Docker
cover: "/assets/instacode.png"
---

## 배경
원래는 dual booting으로 노트북에 ubuntu를 설치해서 사용했었는데, 이번에 노트북을 바꾸면서 따로 ubuntu를 설치하지 않아서 운영하던 jekyll 기반 블로그를 잠깐 쉬게 되었다(window 환경에서는 jekyll 서버 구동이 어려움).<br>
그리고 최근에 wsl2를 이용해 window환경에서 Docker로 jekyll image를 이용해서 로컬 jekyll 서버를 띄운 후 블로그 글을 작성하려고 했는데, 문제가 생겼다.<br>
 docker container실행시 `-p 4000:4000` 옵션으로 포트포워딩을 했는데, 로컬 pc에서 127.0.0.1:4000/blog/ 로 접속이 안되는 것이었다.

## 원인 및 해결
컨테이너에서 `bundle exec jekyll serve` 명령어 실행시 127.0.0.1:4000 경로로 jekyll 서버 접근이 가능한데, 컨테이너의 localhost이다 보니까 로컬환경에서 접근이 제한되었다. 그래서 `bundle exec jekyll serve --host=0.0.0.0` 으로 host를 컨테이너의 localhost가 아니라 외부에서 접근 가능한 host로 변경해서 jekyll 서버를 띄워줘야 했고, 이렇게 하니 정상적으로 접근이 되었다.
