---
title: "[MongoDb] pymongo 설치/기본적인 명령어들"
layout: post
date: '2020-11-25 03:02:11'
author: Edward Park
categories:
- DataBase
tags:
- Mongodb
cover: "/assets/instacode.png"
---

## Install MongoDb in ubuntu

[Documentation](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)

```
sudo systemctl start mongod #stop
sudo systemctl status mongod
```

location : /var/lib/mongodb

sudo apt list --installed\|grep mongodb

sudo apt-get purge --auto-remove packagename

If you installed via the package manager, the data directory `/var/lib/mongodb` and the log directory `/var/log/mongodb` are created during the installation.

<br>
<br>
### MongoDb 데이터 저장 구조

> DB-Collection-Document-Key_value<br>
상위 --- 하위

### Python에서 MongoDb 사용
```Python

from pymongo import MongoClient

client = MongoClient('127.0.0.1')
```


### 명령어s
#### DB

- **DB들 확인**

  client.list_database_names()

- **DB 생성**

  db = client.name

- **DB 의 collection 이름 확인**

  db.collection_names()

- **DB 의 특정 collection drop**

  db.colname.drop()

### Collection

- **collection 의 document  개수 확인**

  db.colname.count()

- **collection 에 document 삽입(dictionary 형태만 가능)**

  db.colname.insert_one({'a':1})

  db.colname.insert([{~},{~}])

- **collection 의 원소 삭제**

  db.colname.remove({'a':1})

- **collection의 원소 업데이트**

  db.colname.update({"a":1},{"$set":{"new":123}})

- **collection 의  document 조회**

  - 모든 원소 조회

    for i in db.colname.find():
        print(i)

  - 조건부 원소 조회

    db.colname.find_one({"a":1})

    db.colname.find({"b":{"$lte":"30"}}) #b값이 30미만인 documents

    ​	$in:["a","b"] #b값이 리스트 원소안에 속하는지

    ​	$or:["a":1,{"b":1}] #논리 연산자 ​/ or, and, not , nor 

- **중복 제거**

  db.colname.ensure_index([("a" , pymongo.ASCENDING), ("unique" , True), ("dropDups" , True)])
