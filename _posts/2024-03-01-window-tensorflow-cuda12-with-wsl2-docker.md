---
title: Window Tensorflow 최신 CUDA(12 이상) 사용법(with WSL2, Docker)
layout: post
date: '2024-03-01 12:30:00'
author: Edward Park
categories:
- DataScience
tags:
- Python
- Docker
- Tensorflow
cover: "/assets/instacode.png"
---

## Intro
현재 CUDA 12.1 버전으로 Torch를 사용하고 있었는데, Tensorflow는 [Tensorflow gpu-linux/Mac OS](https://www.tensorflow.org/install/source?hl=ko)에서 확인해보니까 CUDA 12.1버전을 지원하지 않고 있었다. 그리고 [window](https://www.tensorflow.org/install/source_windows?hl=ko)의 경우 **CUDA 11.2 이후 버전은 지원하지 않는다**고 나와있었다(Tensorflow 버전 2.10 까지만 사용 가능)<br>

해당 문서에 WSL2를 이용하면 최신 버전의 CUDA 사용할 수 있다고 나와 있었고, 마침 정리가 잘 되어 있는 문서([Docker로 Tensorflow-gpu-jupyter 설치](https://wikidocs.net/209146))가 있어서 쉽게 환경을 구축할 수 있었다.<br>
\*본 포스트에서는  **jupyter lab(notebook)**을 사용할 수 있는 환경 구축을 목표로 함

## Docker사용 시 장점
원래 Tensorflow 에서 GPU를 사용하기 위해서는<br>
	1. NVIDIA 드라이버 설치
	2. CUDA 설치
	3. CUDNN 설치

위 3가지 작업을 해야 한다. 여기서 NVIDIA 드라이버는 PC/Laptop의 그래픽카드에 맞는걸 설치하는 것이고, CUDA 및 CUDNN는 사용하고자 하는 Tensorflow 버전에 따라 설치해야하는 버전이 다르다([Tensorflow gpu-linux/Mac OS](https://www.tensorflow.org/install/source?hl=ko) 참조). 이로 인해 **하나의 PC에서 여러 버전의 CUDA를 설치하고, 관리해야하는 번거로움**이 생길 수 있다.<br>
그러나 Docker를 사용할 시 이러한 번거로움을 해결할 수 있다. **CUDA와 CUDNN를 Local에 설치할 필요가 없기 때문**이다.

## 0. 사전 환경 세팅
Window에 WSL2 및 Docker가 설치된 상태여야 한다. 아래와 같이 wsl 창에서 docker 관련 command가 인식이 되는 상태면 OK
<img src="/blog/post_images/tensorflow/wsl1.png" title="wsl1">

## 1. NVIDIA-Docker 세팅
- 참고 문서: [WSL2-Ubuntu에 NVIDIA-Docker 설치하기](https://wikidocs.net/206697)

원래는 Docker에서 GPU를 사용하기 위해서는 NVIDIA-DOCKER를 설치해야 하는데, Docker 19.03 이후에는 간단한 플러그인을 통해 Docker에서 NVIDIA GPU 환경을 사용할 수 있다고 나와 있다.<br>
**A. 플러그인 설치(wsl shell에서 명령어 실행)**


```Shell
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
$ sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
$ sudo apt-get install -y nvidia-container-toolkit
```
**B. Docker 재시작**<br>
Docker desktop을 쓰는 경우 아이콘을 우클릭해 재시작 할 수 있고, 아닌 경우 `sudo service docker restart` 명령어로 재시작<br>
**C. 설치 확인**<br>
`sudo docker run --gpus all nvidia/cuda:11.8.0-base-ubuntu20.04 nvidia-smi` 커맨드를 입력 시 nvidia-smi 결과 값이 나오면 정상적으로 설치가 완료 된 상태. 여기서 11.8.0 부분을 본인이 사용하고자 하는 cuda 버전으로 변경해서 실행.(ex. cuda:12.2.0-base)<br>
참고로 해당 커맨드는 nvidia 환경의 ubuntu 이미지를 pull 하고, 실행 후 nvidia-smi 커맨드를 입력결과를 반환하는 것이다.

## 2. Tensorflow-gpu-jupyter 이미지 Pull
- 참고 문서: [Docker로 Tensorflow-gpu-jupyter 설치](https://wikidocs.net/209146)

이제 별 다른 작업 없이 적절한 docker image를 pull하면 된다.(지원 버전은 [dockerhub tensorflow](https://hub.docker.com/r/tensorflow/tensorflow/tags?page=1&name=2.15.0-gpu) 참조)

```Shell
$ sudo docker pull tensorflow/tensorflow:2.15.0-gpu-jupyter
```

## 3. 이미지 실행
```Shell
$ docker run -it --gpus all -p 8888:8888 tensorflow/tensorflow:2.15.0-gpu-jupyter
```
wsl shell에서 위 명령어를 입력시 아래와 같이 jupyter lab이 실행되며, http://127~ 부분을 ctrl+좌클릭 하면 jupyter lab 화면으로 이동된다.
<img src="/blog/post_images/tensorflow/wsl2.png" title="wsl2">
<img src="/blog/post_images/tensorflow/lab1.png" title="lab1">

## 4. GPU 사용 가능 여부 확인
jupyter lab에서 notebook을 생성하고 아래의 명령어를 통해 gpu 사용 가능 여부를 확인할 수 있다.
```Python
import tensorflow as tf

if tf.test.gpu_device_name():
    print('Default GPU Device: {}'.format(tf.test.gpu_device_name()))
else:
    print("Please install GPU version of TF")
```
<img src="/blog/post_images/tensorflow/lab3.png" title="lab3">

참고로 pandas 등 필요한 라이브러리는 terminal에서 설치가 가능하다
<img src="/blog/post_images/tensorflow/lab2.png" title="lab2">

## 5. Volume 연동
기본적으로 독립된 container 환경에서 jupyter lab이 서비스 되기 때문에 사용하고자하는 workspace(폴더)를 공유하는 작업이 필요하다. <br>
docker의 -v(\--volume) 옵션을 통해 local에 있는 폴더와 container의 폴더를 연결할 수 있다. 참고로 container의 기본 폴더 위치는 `/tf` 이다. (terminal에서 pwd 명령어를 통해서 확인 가능)
```Shell
$ docker run -it --gpus all -p 8888:8888 -v /mnt/c/Users/edward/my_projects:/tf/my_projects tensorflow/tensorflow:2.15.0-gpu-jupyter
```

`/mnt/c/Users/edward/my_projects` 이 부분에 본인의 local 폴더 경로를 입력해 연동하여 사용 가능하다.

## 6. 재기동
`docker ps -a` 명령어를 이용해 container id를 알아내고, `docker start [container-id]` command를 통해 jupyter lab을 실행시킬 수 있다.
<img src="/blog/post_images/tensorflow/wsl3.png" title="wsl3">
접속할 url은 `docker logs [container-id]` 명령어를 통해 확인 가능하다.

## 7. 주의사항
- PC의 그래픽카드에서 지원하는 CUDA 버전과 사용하고자 하는 Tensorflow의 버전이 호환이 되는지 반드시 확인하여야 한다.
- 띄워놓은 container에서 환경을 세팅해(필요한 라이브러리 설치 등) 사용하다가 container가 삭제되면 다시 환경을 세팅해야하는 번거로움이 생길 수 있기 때문에 local의 python virtual env를 사용하거나, 사용하고 있는 container를 docker image로 관리하는 등의 방법을 통해 사용하는 것을 권장한다.
