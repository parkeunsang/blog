---
title: "[MySQL] database 이름 변경"
layout: post
date: '2021-02-11 21:01:32'
author: Edward Park
categories:
- DataBase
tags:
- MySQL
cover: "/assets/instacode.png"
---

## Abstract
일반적으로 폴더의 이름을 변경하거나 프로그래밍에서 어떤 객체의 이름을 변경하는것은 매우 간단한 일이므로 나는 SQL의 **데이터베이스(이하 DB)** 이름을 변경하는것도 이와 마찬가지로 쉬운 일이라고 생각을 했었다. 하지만 막상 이를 시도해보니 쉽지가 않았다. <br>
그래서 이를 해결해나간 과정을 간략하게 기술하고자 한다.

## Command: RENAME
Mysql 5.1이하의 버전에서는 RENAME 이라는 명령어로 database의 이름을 바꿀 수 있었지만, stable하지않다는 이유로 그 이후의 버전에서는 해당 명령어가 사라졌다. 그래서 현재는 mysql내부의 명령어만으로는 database의 이름을 바꿀 수 없다.
```Shell
$ mysql -V
```
참고 : Mysql version check

## Rename using mysqldump / mysqladmin
그래서 해당 database를 db.sql 형태의 sql파일로 저장한 다음 다시 mysql에서 database를 생성해 이를 불러온 후 원래 database를 삭제하는 과정으로 rename이 진행된다.<br>
참고 : [MySQL docs](https://dev.mysql.com/doc/refman/8.0/en/mysqldump-copying-database.html)

```Shell
$ mysqldump -u root -p db1 > dump.sql  # 로컬에 sql 파일로 저장
$ mysqladmin -u root -p create db2  # 새로운 database 생성
$ mysql -u root -p db2 < dump.sql  # sql 파일 불러오기
$ mysql -u root -p  # mysql 접속
mysql> use db1 
mysql> DROP DATABASE db1;  # db1 삭제
```
