---
title: "[Git] commit 메세지 수정"
layout: post
date: '2021-09-23 20:58:33'
author: Edward Park
categories:
- Dummy
tags: []
cover: "/assets/instacode.png"
---

## 배경
git은 형상관리, 코드공유 등에 요긴하게 이용된다. git의 핵심은 단연 **코드**이며 이 외의것은 무시되기 쉽다. 나 또한 commit 메세지라던지, branch 관리 등의 필요성을 크게 못느껴 이를 등한시 했었다. 하지만 협업을 하다보니 이런 부가적인(?) 것들이 왜 필요한지 점차 깨달아가는 것 같다.<br>
이번 포스트는 최근 삼성전자 오픈소스 프로젝트에 참여하며 커밋 메세지 규칙(4단어 이상)을 지키지 않고 pull request를 보내었다가 봇에게 반려당한 경험을 바탕으로 쓰게 되었다.

## 직전 커밋 메세지 수정
직전 커밋 메세지의 경우 아래의 명령어를 통해 수정 가능하다.
```Shell
$ git commit --amend
```
해당 명령어를 실행 시 아래와 같은 수정 화면이 등장하며 원하는 메세지로 변경 후 `ctrl+X`, `Y`, `Enter` 를 순서대로 누르면 저장이 된다.<br>
아래 예시에서는 **second**가 커밋 메세지이다.
<img src="/blog/post_images/git/commit_1.png" title="commit">

## 이전 커밋 메세지 수정
직전 커밋 메세지가 아니라 이전의 메세지를 수정하고 싶으면 다음과 같은 과정을 거쳐야 한다.
1. 원하는 위치(여기서는 3번째 이전)로 커밋을 이동
 ```Shell
 $ git rebase -i HEAD~3
 ```
2. 커밋 메세지 수정 상태로 변환(pick -> reword)
<img src="/blog/post_images/git/commit_2.png" title="commit">
reword로 글자 수정 후 저장하면 바로 해당 커밋(들)의 메세지를 수정할 수 있는 창이 나온다.
3. 커밋 메세지 수정 후 저장<br>
(위의 amend 명령어를 입력했을때 나오는 창과 동일.)

## Git branch에 push한 경우
이미 push를 완료한 경우 커밋 메세지를 수정해서 일반적인 방법으로 push를 하면 conflict가 발생한다.<br>
따라서 force옵션을 통해 강제로 push를 해 커밋 메세지를 수정해야한다. 하지만 이와 같은 경우 해당 브랜치를 사용하고 있는 다른 사용자들과 conflict가 발생할 수 있으므로 가급적 자제해야한다.
```Shell
$ git push origin master[branch name] --force
```
참고 : [git 커밋 메세지 수정하기](https://velog.io/@mayinjanuary/git-%EC%BB%A4%EB%B0%8B-%EB%A9%94%EC%84%B8%EC%A7%80-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0-changing-commit-message)
