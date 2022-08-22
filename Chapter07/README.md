# 7장: 앤서블로 하는 구성 관리

7장의 예제를 사용하려면 다음 조건이 필요하다:
* 앤서블이 설치되어 있어야 한다.
* SSH가 구성된 두 대의 우분투 머신 (각 머신에 비밀번호없이 'ssh'로 접속할 수 있어야 한다)
* 원격 머신을 가리키는 앤서블 인벤토리 파일(`/etc/ansible/hosts`)

앤서블 인벤토리 파일 예제:

	[webservers]
	web1 ansible_host=<machine-ip-1> ansible_user=<machine-user-1>
	web2 ansible_host=<machine-ip-2> ansible_user=<machine-user-2>

## 코드 예제

### 예제 1: 애드혹 명령

인벤토리가 올바른지 확인하려면, 앤서블 애드혹 `ping` 명령을 실핸한다.

	$ ansible all -m ping
	web1 | SUCCESS => {
	    "ansible_facts": {
	        "discovered_interpreter_python": "/usr/bin/python3"
	    },
	    "changed": false,
	    "ping": "pong"
	}
	web2 | SUCCESS => {
	    "ansible_facts": {
	        "discovered_interpreter_python": "/usr/bin/python3"
	    },
	    "changed": false,
	    "ping": "pong"
	}

### 예제 2: 플레이북

[sample2](sample2)에는 'web1' 호스트에 'apache2'를 설치하고 실행할 수 있는 플레이북이 있다.

다음 명령으로 실행한다.

	$ ansible-playbook playbook.yml

### 예제 3: 핸들러

[sample3](sample3)에는 'apache2'를 설치하고 실행하는 것과 별도로, 로컬 'apache2' 구성을 복사하고, 구성이 변경된 경우에만 서버를 재시작하는 플레이북이 있다.

구성을 생성하고 플레이북을 실행하려면 다음 명령을 실행한다.

	$ touch foo.conf
	$ ansible-playbook playbook.yml

참고로 `ansible-playbook`을 여러 번 실행해도, 아무것도 변경되지 않는다. 이제, 새 구성을 생성하고 플레이북을 다시 실행하면 'apache2' 서버가 재시작하는 것을 볼 수 있다.

	$ echo "something" > foo.conf
	$ ansible-playbook playbook.yml

### 예제 4: 변수

[sample4](sample4)에는 변수 사용을 보여주는 플레이북이 있다.

결과를 보려면 다음 명령을 실행한다.

	$ ansible-playbook playbook.yml

### 예제 5: 앤서블로 배포하기

[sample5](sample5)에는 'web1'의 Hazelcast 서버와 (Hazelcast를 사용하는) 'web2'의 계산기 서비스를 시작하는 플레이북이 있다.

플레이북을 실행하려면, 먼저 `src/main/java/com/leszko/calculator/CalculatorApplication.java`를 수정하고,  `<machine-ip-1>`를 사용자의 `web1` 머신 IP 주소로 변경한다.그런 다음, 프로젝트를 빌드하고 앤서블 플레이북을 적용한다.

	$ ./gradlew build
	$ ansible-playbook playbook.yml

결과적으로, 두 개의 독립 애플리케이션을 배포했다. 다음 명령으로 이를 테스트할 수 있다.

	$ curl http://<machine-ip-2>:8080/sum?a=1\&b=2
	3

### 예제 6: 앤서블 도커 플레이북

[sample6](sample6)에는 2개의 플레이북이 있다:
 * `install-docker-playbook.yml`: 우분투 20.04 서버에 도커 커뮤니티 에디션을 설치하는 플레이북
 * `hazelcast-playbook.yml`: 도커 데몬이 실행 중인 서버에서 Hazelcast 도커 컨테이너를 시작하는 플레이북

 서버에서 도커 데몬을 설치하려면, 다음 명령을 실행한다.

	$ ansible-playbook install-docker-playbook.yml

만약 명령이 실패하면, 다시 실행한다.

그런 다음, Hazelcast 컨테이너를 시작하기 위해서, 다음 명령을 실행한다.

	$ ansible-playbook hazelcast-playbook.yml


## 연습 문제 해법

### 연습 1: 서버 인프라를 생성하고 이를 앤서블로 관리한다.

버추얼박스나 클라우드 서비스(AWS, GCP, or Azure), 또는 베어메탈 서버를 사용할 수 있다. SSH 공개를 `authorized_keys`로 구성한다. 그런 다음, 각 서버에 파이썬을 설치한다. 마지막으로, 인벤토리 파일(`/etc/ansible/hosts`)에 서버 IP와 사용자 이름을 추가한다.

구성이 완료되면 모든 서버에 다음과 같이 `ping`을 할 수 있다.

	$ ansible all -m ping

### 연습 2: 앤서블로 파이썬 기반 "hello world" 웹 서비스를 배포한다.

[exercise2](exercise2) 디렉토리에는 hello world 애플리케이션의 소스 코드와 이를 배포하는 앤서블 플레이북이 있다.

Hello World 서비스를 `web1` 서버에 설치하려면, 다음 명령을 실행한다.

	$ ansible-playbook playbook.yml

그 후에 Hello World 서비스를 호출할 수 있다.

	$ curl http://<web1-ip>:5000/hello
	Hello World!
