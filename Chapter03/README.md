# 3장: 젠킨스 구성

3장의 명령어를 사용하려면 다음 조건이 필요하다:
* 도커가 설치되어야 한다.
* 도커 허브에 계정이 있어야 한다.
* 도커 허브에 로그인 상태여야 한다. (`docker login`)

## 코드 예제

### 예제 1: 젠킨스 에이전트 빌드하기

[sample1](sample1)에는 파이썬 인터프리터가 포함된 젠킨스 에이전트 빌드 프로젝트가 있다. 다음 명령으로 빌드할 수 있다.

    $ docker build -t leszko/jenkins-agent-python .

`leszko`를 사용자의 도커 허브 계정으로 변경해야 한다.

그런 다음, 이미지를 도커 허브 레지스트리로 푸시할 수 있다.

    $ docker push leszko/jenkins-agent-python

다음으로, 사용자의 레지스트리를 공개로 설정한다. (또는 젠킨스 마스터에서 도커 허브 용 자격 증명을 구성한다).

### 예제 2: 젠킨스 마스터 빌드하기

[sample2](sample2)에는 커스텀 젠킨스 마스터 도커 이미지 빌드 프로젝트가 있다. 다음 명령으로 빌드할 수 있다.

    $ docker build -t jenkins-master .

그런 다음, 생성된 젠킨스 마스터를 실행한다.

    $ docker run -p 8080:8080 jenkins-master

## 연습 문제 해법

### 연습 1: 젠킨스 마스터와 에이전트 도커 이미지 빌드하기

[exercise1](exercise1) 디렉토리에는 젠킨스 마스터([exercise1/master](exercise1/master))와 젠킨스 에이전트([exercise1/agent](exercise1/agent)) 도커 이미지의 소스 코드가 포함되어 있다.

젠킨스 마스터를 빌드하고 실행하려면 master 디렉토리에서 다음 명령을 실행한다.

    $ docker build -t jenkins-master .
    $ docker run -p 8080:8080 jenkins-master

('leszko'를 사용자의 도커 허브 계정으로 변경하고), agent 디렉토리에서 다음 명령을 실행한다. 

    $ docker build -t leszko/jenkins-agent-ruby .
    $ docker push leszko/jenkins-agent-ruby

도커 허브에서 `leszko/jenkins-agent-ruby` 레포지토리를 공개로 설정하고, 젠킨스 마스터에서 이 이미지를 에이전트로 설정한다.


### 연습 2: 루비 스크립트를 생성 및 실행하는 파이프라인 생성하기

Hello World from Ruby를 출력하는 루비 스크립트 생성 및 실행 스크립트는 다음과 같다.

```
pipeline {
    agent any
    stages {
        stage("Hello") {
            steps {
                sh "echo \"puts 'Hello World from Ruby'\" > hello.rb"
                sh "ruby hello.rb"
            }
        }
    }
}
```