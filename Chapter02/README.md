# 2장: 도커 소개

아래 명령어는 도커가 설치된 환경에서만 사용할 수 있다.

## 코드 예제

### 예제 1: Dockerfile

[sample1](sample1)에는 파이썬 인터프리터가 포함된 도커 이미지를 빌드하는 프로젝트가 있다. 다음 명령으로 빌드할 수 있다.

    $ docker build -t ubuntu_with_python .

### 예제 2: Complete Docker application

[sample2](sample2)에는  Hello World Python 도커 이미지를 빌드하는 프로젝트가 있다. 다음 명령으로 빌드할 수 있다.

    $ docker build -t hello_world_python .
    $ docker run hello_world_python
    Hello World from Python!

### 예제 3: Environment variables

[sample3](sample3)은 도커 이미지와 함께 환경 변수의 사용법을 보여준다. 다음 명령으로 빌드할 수 있다.

    $ docker build -t hello_world_python_name .
    $ docker run -e NAME=Rafal hello_world_python_name
    Hello World from Rafal ! 

## 연습 문제 해법

### 연습 1: CouchDB를 도커 컨테이너로 실행하기

다음 명령으로 CouchDB 데이터베이스를 시작할 수 있다.

    $ docker run -p 5984:5984 couchdb

그리고, 아래 웹 주소에 접속 가능한지 확인한다.
[http://localhost:5984/\_utils/](http://localhost:5984/_utils/)

### 연습 2: REST 서비스로 도커 이미지 생성하기

[exercise2](exercise2) 디렉토리에는 REST 웹서비스가 포함된 도커 이미지의 소스 코드가 포함되어 있다. 다음 명령으로 빌드할 수 있다.

    $ docker build -t hello-service .
    $ docker run -d -p 5000:5000 hello-service
    $ curl http://localhost:5000/hello
    Hello World!