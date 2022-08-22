# 6장: 쿠버네티스로 하는 클러스터링

6장의 예제를 사용하려면 다음 조건이 필요하다:
* 쿠버네티스가 설치 및 구성되어 있어야 한다.

## 코드 예제

### 예제 1: 쿠버네티스에 애플리케이션 배포하기

[sample1](sample1)에는 쿠버네티스 클러스터의 여러 복제본에서 도커 컨테이너를 시작하는 YAML 구성이 있다.

배포를 설치하려면 다음 명령을 실행한다.

    $ kubectl apply -f deployment.yaml

한 개의 도커 컨테이너를 갖는 3개의 포드가 생성된 것을 확인할 수 있다.

    $ kubectl get pods
    NAME                                    READY   STATUS    RESTARTS   AGE
    calculator-deployment-dccdf8756-h2l6c   1/1     Running   0          1m
    calculator-deployment-dccdf8756-tgw48   1/1     Running   0          1m
    calculator-deployment-dccdf8756-vtwjz   1/1     Running   0          1m

실행 중인 포드의 로그를 확인하려면, 다음 명령을 실행한다.

    $ kubectl logs pods/calculator-deployment-dccdf8756-h2l6c

### 예제 2: 애플리케이션 노출하기

[sample2](sample2)에는 예제 1에서 생성한 배포를 노출하는 서비스를 생성하는 YAML 구성이 있다.

서비스를 설치하려면, 다음 명령을 실행한다.

    $ kubectl apply -f service.yaml

서비스가 예제 1에서 생성한 3개의 포드 복제본을 가리키는지 확인하려면, 다음 명령을 실행한다.

    $ kubectl describe service calculator-service | grep Endpoints
    Endpoints: 10.16.1.5:8080,10.16.2.6:8080,10.16.2.7:8080

### 예제 3: 애플리케이션 확장하기

예제 1에서 애플리케이션을 확장(또는 축소)하려면, 다음 명령을 실행한다.

    $ kubectl scale --replicas 5 deployment calculator-deployment

쿠버네티스가 계산기 포드의 개수를 3에서 5로 증가시킨 것을 알 수 있다.

    $ kubectl get pods
    NAME                                    READY   STATUS    RESTARTS   AGE
    calculator-deployment-dccdf8756-h2l6c   1/1     Running   0          19h
    calculator-deployment-dccdf8756-j87kg   1/1     Running   0          36s
    calculator-deployment-dccdf8756-tgw48   1/1     Running   0          19h
    calculator-deployment-dccdf8756-vtwjz   1/1     Running   0          19h
    calculator-deployment-dccdf8756-zw748   1/1     Running   0          36s

### 예제 4: 애플리케이션 업데이트하기

[sample4](sample4)에는 예제 1에서 업데이트한 배포가 포함된 YAML 구성이 있다. 쿠버네티스가 배포를 업데이트하는 방법을 보려면, 다음 명령을 실행한다.

    $ kubectl apply -f deployment.yaml

그런 다음, 모든 포드가 종료되고 쿠버네티스가 새로운 포드를 시작하는 것을 확인한다.

    $ kubectl get pods
    NAME                                       READY STATUS      RESTARTS AGE
    pod/calculator-deployment-7cc54cfc58-5rs9g 1/1   Running     0        7s
    pod/calculator-deployment-7cc54cfc58-jcqlx 1/1   Running     0        4s
    pod/calculator-deployment-7cc54cfc58-lsh7z 1/1   Running     0        4s
    pod/calculator-deployment-7cc54cfc58-njbbc 1/1   Running     0        7s
    pod/calculator-deployment-7cc54cfc58-pbthv 1/1   Running     0        7s
    pod/calculator-deployment-dccdf8756-h2l6c  0/1   Terminating 0        20h
    pod/calculator-deployment-dccdf8756-j87kg  0/1   Terminating 0        18m
    pod/calculator-deployment-dccdf8756-tgw48  0/1   Terminating 0        20h
    pod/calculator-deployment-dccdf8756-vtwjz  0/1   Terminating 0        20h
    pod/calculator-deployment-dccdf8756-zw748  0/1   Terminating 0        18m

쿠버네티스 클러스터의 여러 복제본에서 도커 컨테이너를 시작한다.

### 예제 5: 롤링 업데이트

[sample5](sample5)에는 롤링 업데이트 구성이 추가된 예제 1의 배포가 포함된 YAML 구성이 있다. 쿠버네티스가 롤링 업데이트하는 방법을 보려면, 다음 명령을 실행한다.

    $ kubectl apply -f deployment.yaml

이제 `kubectl get pods`를 실행하여 포드가 종료되고, 하나씩 실행되는지 확인한다.

    $ kubectl get pods
    NAME                                   READY STATUS      RESTARTS AGE
    calculator-deployment-78fd7b57b8-npphx 0/1   Running     0        4s
    calculator-deployment-7cc54cfc58-5rs9g 1/1   Running     0        3h
    calculator-deployment-7cc54cfc58-jcqlx 0/1   Terminating 0        3h
    calculator-deployment-7cc54cfc58-lsh7z 1/1   Running     0        3h
    calculator-deployment-7cc54cfc58-njbbc 1/1   Running     0        3h
    calculator-deployment-7cc54cfc58-pbthv 1/1   Running     0        3h

### 예제 6: 애플리케이션 의존성

[sample6](sample6)에는 Hazelcast 쿠버네티스 배포 구성과 Hazelcast를 사용하는 계산기 애플리케이션의 소스 코드가 있다.

우선, Hazelcast 애플리케이션을 시작한다.

    $ kubectl apply -f hazelcast.yaml

포드 로그를 확인하여 Hazelcast가 실행 중인지 여부를 알 수 있다. 다음으로, Hazelcast를 캐싱으로 사용하는 애플리케이션을 빌드한다. 또한 도커 이미지를 빌드하고 도커 허브 계정으로 푸시한다.

    $ ./gradlew build
    $ docker build -t <username>/calculator:caching .
    $ docker push <username>/calculator:caching

마지막으로, `deployment.yaml`의 `<username>`을 사용자의 도커 허브 계정으로 변경하고, deployment와 service를 적용한다.

    $ kubectl apply -f deployment.yaml
    $ kubectl apply -f service.yaml

로그에서 다음 행을 찾으면 계산기 애플리케이션이 성공적으로 Hazelcast 서버에 접속했는지를 알 수 있다.

    Members [1] {
        Member [10.16.2.15]:5701 - 3fca574b-bbdb-4c14-ac9d-73c45f56b300
    } 


## 연습 문제 해법

### 연습 1: 쿠버네티스 클러스터에서 hello world 애플리케이션을 실행한다.

[exercise1](exercise1) 디렉토리에는 hello world 애플리케이션의 소스 코드와 이를 배포하는 쿠버네티스 YAML 구성이 있다.

우선, hello world 애플리케이션 도커 이미지를 빌드하고, 도커 허브로 푸시한다.

    $ docker build -t <username>/hello-service:0.1 .
    $ docker push <username>/hello-service:0.1

`deployment.yaml`에서 `<username>`를 사용자의 도커 허브 계정으로 변경한다. 그런 다음, 다음 명령으로 쿠버네티스에 애플리케이션을 설치한다.

    $ kubectl apply -f deployment.yaml

애플리케이션이 제대로 동작하는지를 확인하는 요청을 한다.

    $ curl http://<NODE-IP>:<NODE-PORT>/hello
    Hello World!

### 연습 2: 새로운 기능 "Goodbye World!"를 구현하고, 롤링 업데이트를 사용하여 배포한다.

[exercise2](exercise2) 디렉토리에는 "Goodbye World!" 기능이 추가된 hello world 애플리케이션의 소스 코드와 롤링 업데이트 전략을 사용해 대포하는 쿠버네티스 YAML 구성이 있다.

우선 새로운 버전의 hello world를 빌드한다.

    $ docker build -t <username>/hello-service:0.2 .
    $ docker push <username>/hello-service:0.2

`deployment.yaml`에서 `<username>`를 사용자의 도커 허브 계정으로 변경한다. 그런 다음, 다음 명령으로 롤링 업데이트를 수행한다.

    $ kubectl apply -f deployment.yaml

애플리케이션이 제대로 동작하는지를 확인하는 요청을 한다.

    $ curl http://<NODE-IP>:<NODE-PORT>/bye
    Goodbye World!
