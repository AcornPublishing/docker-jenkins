# 4장: 지속적 통합 파이프라인

4장의 예제를 사용하려면 다음 조건이 필요하다:
* 젠킨스 인스턴스가 실행 중이어야 한다.
* 자바 JDK 8+  설치 환경
* 파이썬 설치 환경

## 코드 예제

### 예제 1: 스프링 부트 프로젝트용 기본 커밋 파이프라인

[sample1](sample1)에는 두 개의 숫자를 합산하는 간단한 스프링 부트 웹 서비스가 있다. 다음 명령으로 시작할 수 있다.

    $ ./gradlew bootRun

REST 호출로 동작 여부를 확인할 수 있다.

    $ curl localhost:8080/sum?a=1\&b=2
    3

자동 단위 테스트를 실행하려면, 다음 명령을 실행한다.

    $ ./gradlew test

이 프로젝트용 지속적 통합 파이프라인은 'Jenkinsfile'에 있으며, 다음과 같다.

```
pipeline {
     agent any
     stages {
          stage("Compile") {
               steps {
                    sh "./gradlew compileJava"
               }
          }
          stage("Unit test") {
               steps {
                    sh "./gradlew test"
               }
          }
     }
}
```

### 예제 2: 코드 커버리지 스테이지

[sample2](sample2)은 예제 1에 코드 커버리지 점검을 추가해서 웹 서비스를 확장한다. 다음 명령으로 실행할 수 있다.

    $ ./gradlew test jacocoTestCoverageVerification

다음 명령으로 코드 커버리지 리포트를 생성할 수 있다.

    $ ./gradlew test jacocoTestReport

리포트는 `build/reports/jacoco/test` 디렉토리에 생성된다.

젠킨스 파이프라인에 코드 커버리지를 포함하려면 다음 스테이지를 추가한다.

```
stage("Code coverage") {
     steps {
          sh "./gradlew jacocoTestReport"
          sh "./gradlew jacocoTestCoverageVerification"
     }
}
```

### 예제 3: 정적 코드 분석 스테이지

[sample3](sample3)은 예제 1에 정적 코드 분석 점검을 추가해서 웹 서비스를 확장한다. 다음 명령으로 실행할 수 있다. 

    $ ./gradlew checkstyleMain

젠킨스 파이프라인에 정적 코드 분석을 포함하려면 다음 스테이지를 추가한다.

```
stage("Static code analysis") {
     steps {
          sh "./gradlew checkstyleMain"
     }
}
```

## 연습 문제 해법

### 연습 1: 단위 테스트가 있는 파이썬 프로젝트 생성하기

[exercise1](exercise1) 디렉토리에는 파이썬 계산기 프로젝트의 소스 코드가 포함되어 있다. 소스 코드는 `calculator.py` (프로그램)와 `test_calculator.py` (단위 테스트)의 두 파일로 구성되어 있다.

다음 명령으로 프로그램을 실행한다.

    $ python calculator.py 2 2
    4

단위 테스트는 다음 명령으로 실행한다.

    $ python test_calculator.py
    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.002s
    OK

### 연습 2: 파이썬 기반의 지속적 통합 파이프라인 빌드하기

다음은 파이썬 계산기 프로젝트용 지속적 통합 파이프라인용 `Jenkinsfile`이다.

```
pipeline {
    agent any
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage("Unit test") {
            steps {
                sh "python test_calculator.py"
            }
        }
    }
}
```