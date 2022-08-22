# 8장: 지속적 인도 파이프라인

8장의 예제를 사용하려면 다음 조건이 필요하다:
* 젠킨스 인스턴스(자바 JDK 8+, 도커, 젠킨스 에이전트 설치된 kubectl)
* 도커 레지스트리(도커 허브 계정)
* 두 개의 쿠버네티스 클러스터(한 개는 `staging` 용도, 한 개는 `production` 용도)
* 젠킨스에 Timestamp 플로그인 설치(공백을 포함하지 않도록 구성된 타임스탬프 형식)
* 젠킨스에 설정된 도커 허브 자격 증명

## 코드 예제

### 예제 1: 완전한 지속적 인도 파이프라인

[sample1](sample1)에는 완전한 지속적 인도 파이프라인을 정의한 `Jenkinsfile`과 계산기 프로젝트가 있다.

## 연습 문제 해법

### 연습 1: Hello World 서비스용 성능 테스트 생성하기

[exercise1](exercise1) 디렉토리에는 hello world 애플리케이션의 소스 코드와 성능 테스트가 있다.

Hello World 애플리케이션을 실행하려면, 다음 명령을 실행한다.

	$ python app.py

성능 테스트를 실행하려면, 다음 명령을 실행한다.

	$ ./performance-test.sh localhost:5000

만약 명령 실행 결과가  `0` 이면, 성능 테스트를 통과한 것이다.

### 연습 2: Hello World 서비스용 젠킨스 파이프라인 생성하기

[exercise2](exercise2) 디렉토리에는 완전한 Hello World 서비스와 `Jenkinsfile`이 있다.
