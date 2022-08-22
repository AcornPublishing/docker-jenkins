# 5장: 자동 인수 테스트

5장의 예제를 사용하려면 다음 조건이 필요하다:
* 도커가 설치되어야 한다. 
* 자바 JDK 8+이 설치된 환경
* 젠킨스 인스턴스가 실행 중이어야 한다. 
* 자바 JDK 8+ 와 도커가 설치된 젠킨스 에이전트(executor)
* 루비(Dev)와 큐컴버가 설치된 환경

## 코드 예제

### 예제 1: 도커 레지스트리 사용하기

[sample1](sample1)에는 파이썬 인터프리터가 포함된 도커 이미지를 빌드하는 프로젝트가 포함되어 있다. 다음 명령으로 빌드할 수 있다.

    $ docker build -t ubuntu_with_python .

도커 허브 레지스트리를 사용하도록 이미지에 태그를 한다. (만약 계정이 없다면, [여기](https://hub.docker.com/signup)에서 생성한다).

    $ docker tag ubuntu_with_python <username>/ubuntu_with_python:1

도커 허브에 로그인한다..

    $ docker login --username <username> --password <password>

이미지를 레지스트리로 푸시한다.

    $ docker push <username>/ubuntu_with_python:1

이미지가 푸시되었는지 확인하려면, 도커 데몬에서 이미지를 제거하고 레지스트리에서 가져와본다.

    $ docker rmi ubuntu_with_python <username>/ubuntu_with_python:1
    $ docker pull <username>/ubuntu_with_python:1

### 예제 2: 스프링 부트 애플리케이션 도커화하기

[sample2](sample2)에는 스프링 부트 애플리케이션과 관련된 Dockerfile이 포함되어 있다. 다음 명령으로 빌드할 수 있다.

	$ ./gradlew build
	$ docker build -t calculator .

도커 이미지가 올바른지 테스트하기 위해, 이미지를 실행하고 'curl'을 사용해서 GET 요청을 한다.

	$ docker run -p 8080:8080 --name calculator calculator
	$ curl localhost:8080/sum?a=1\&b=2
    3

젠킨스 파이프라인에 그래들 빌드 스테이지와 도커 빌드 스테이지를 포함하려면, 다음 부분을 'Jenkinsfile'에 추가한다.

```
stage("Package") {
     steps {
          sh "./gradlew build"
     }
}

stage("Docker build") {
     steps {
          sh "docker build -t <username>/calculator ."
     }
}
```

Note that you need to change `<username>`을 사용자의 도커 허브 ID로 변경해야 한다. 또한 젠킨스 executor에 도커가 구성되어 있어야 한다.

이 예제의 마지막 부분으로, 도커 허브에 로그인하고, 도커 이미지를 푸시하는 스테이지를 추가한다.

```
stage("Docker login") {
     steps {
          withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-hub-credentials',
                   usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
               sh "docker login --username $USERNAME --password $PASSWORD"
          }
     }
}

stage("Docker push") {
     steps {
          sh "docker push <username>/calculator"
     }
}
```

젠킨스 빌드를 실행하기 전에, 젠킨스에서 `docker-hub-credentials` 자격 증명을 설정해야 한다.

### 예제 3: 인수 테스트 스테이지

[sample2](sample2) 프로젝트에서 간단한 인수 테스트 스테이지를 실행하려면, 다음 `acceptance_test.sh` 파일을 생성한다.

```bash
#!/bin/bash
test $(curl localhost:8765/sum?a=1\&b=2) -eq 3
```

그런 다음, `Jenkinsfile`에 다음 스테이지를 추가한다:
```
stage("Deploy to staging") {
     steps {
          sh "docker run -d --rm -p 8765:8080 --name calculator <username>/calculator"
     }
}

stage("Acceptance test") {
     steps {
          sleep 60
          sh "chmod +x acceptance_test.sh && ./acceptance_test.sh"
     }
}
```

테스트 컨테이너를 중단시키는 클린업 스테이지를 추가한다.

```
post {
     always {
          sh "docker stop calculator"
     }
}
```

### 예제 4: 큐컴버 인수 테스트 작성하기

[sample4](sample4)에는 큐컴버 프레임워크를 사용하는 인수 테스트가 포함된 스프링 부트 애플리케이션이 있다. 다음 파일을 확인해 보자:
 * `src/test/resources/feature/calculator.feature`: 인수 테스트 시나리오(인수기준(
* `src/test/java/acceptance/StepDefinitions.java`: 스텝 정의
* `src/test/java/acceptance/AcceptanceTest.java`: 큐컴버 JUnit Runner

다음 명령으로 인수 테스트를 실행한다:

    $ ./gradlew acceptanceTest -Dcalculator.url=http://localhost:8080

`calculator.url`를 사용자의 계산기 서비스 주소로 변경한다.

이 인수 테스트를 사용자의 `Jenkinsfile`에서 사용하려면, 다음 스테이지를 추가한다:
```
stage("Acceptance test") {
     steps {
          sleep 60
          sh "./gradlew acceptanceTest -Dcalculator.url=http://localhost:8765"
     }
}
```

## 연습 문제 해법

### 연습 1: 루비 기반 책 라이브러비 웹 서비스

[exercise1](exercise1) 디렉토리에는 `book-library` 웹 서비스와 큐컴버 인수 테스트의 소스 코드가 있다.

루비에 필요한 파일을 설치한다.

    $ gem install rest-client sinatra json cucumber

테스트가 실패하는지 확인한다.

    $ cucumber
    ...
    1 scenario (1 failed)
    3 steps (1 failed, 1 skipped, 1 passed)
    0m0.116s

웹 서비스를 시작한다.

    $ ruby book-library.rb -p 8080

테스트가 성공하는지 확인한다.

    $ cucumber
    ...
    1 scenario (1 passed)
    3 steps (3 passed)
    0m0.195s

### 연습 2: 북 라이브러리 도커화하기

[exercise2](exercise2) 디렉토리에는 `book-library` 웹 서비스와 `Dockerfile`가 있다.

도커 이미지를 빌드하려면, 다음 명령을 실행한다.

    $ docker build -t <username>/book-library .

그런 다음 다음 명령으로 로컬에서 실행한다.

    $ docker run -p 8080:8080 <username>/book-library

이미지를 도커 허브로 푸시하려면, `docker login`으로 로그인하고, 다음 명령을 실행한다.

    $ docker push <username>/book-library

### Exercise 3: 북 라이브러리용 Jenkinsfile

[exercise3](exercise3) 디렉토리에는 `book-library` 도커 이미지를 빌드하고 푸시하는데 사용하는 `Jenkinsfile`이 있다.

다음을 수행한다:
* `<username>`을 사용자의 도커 허브 계정으로 변경한다.
* 사용자의 도커 허브 자격 증명을 젠킨스에 추가한다.