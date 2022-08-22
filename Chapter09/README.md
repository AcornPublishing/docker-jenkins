# 9장: 지속적 인도 - 고급편

9장의 예제를 사용하려면 자바 JDK 8+와 도커가 설치되어 있어야 한다.

## 코드 예제

### 예제 1: 초기 Flyway 마이그레이션

[sample1](sample1)에는 초기 Flyway 마이그레이션과 계산기 프로젝트가 있다.

초기 마이그레이션 SQL은 `src/main/resources/db/migration/V1__Create_calculation_table.sql`에 정의되어 있다. 해당 파일을 살펴보고 마이그레이션을 적용한다.

	$ ./gradlew flywayMigrate -i

명령은 마이그레이션 스크립트를 자동으로 점검하고 데이터베이스 스키마를 업데이트 한다. 그러면 이제 계산기 애플리케이션을 실행할 수 있다.

	$ ./gradlew bootRun

이제 서비스를 호출할 수 있다.

	$ curl localhost:8080/sum?a=1\&b=2
	3

그러면 데이터베이스에 항목이 생성된다. 결과는 다음 주소에서 데이터베이스를 검색하여 확인할 수 있다. http://localhost:8080/h2-console (JDBC URL은 `jdbc:h2:/tmp/calculator` 사용).

### 예제 2: 하위 버전과 호환되는 Flyway 마이그레이션

[sample2](sample2)에는 하위 버전과 호환되는 Flyway 마이그레이션과 계산기 프로젝트가 있다.

마이그레이션은 `CALCULATION` 테이블에 새로운 컬럼 `CREATED_AT`을 추가한다. 마이그레이션은 
`src/main/resources/db/migration/V2__Add_created_at_column.sql`에 정의되어 있다. 마이그레이션을 적용하려면, 다음 명령을 실행한다.

	$ ./gradlew flywayMigrate -i

그런 다음, 애플리케이션을 다시 실행한다.

	$ ./gradlew bootRun

서비스를 다시 호출한다.

	$ curl localhost:8080/sum?a=2\&b=3
	5

데이터베이스에 새로운 항목이 추가되었는지 확인한다.

### 예제 3: 하위 버전과 호환되지 않는 Flyway 마이그레이션

The [sample3](sample3)에는 하위 버전과 호환되지 않는 Flyway 마이그레이션(테이블의 컬럼명을 변경)과 계산기 프로젝트가 있다.

컬럼명 변경은 다음과 같이 진행된다:
 1. 데이터베이스에 새로운 컬럼 추가하기
 2. 두 컬럼을 모두 사용하도록 코드 변경하기
 3. 데이터를 병합하기
 4. 코드에서 이전 컬럼 제거하기
 5. 데이터베이스에서 이전 컬럼 삭제하기

첫 번째 단계는 이미 포함되어 있으며, `src/main/resources/db/migration/V3__Add_sum_column.sql` 마이그레이션을 실행하고, 서비스를 시작하면 제대로 동작하는지를 확인할 수 있다.

	$ ./gradlew flywayMigrate -i
	$ ./gradlew bootRun

서비스를 다시 호출하고 데이터베이스의 항목을 확인한다. 참고로 지금까지 수행한 모든 변경 사항은 하위 버전과 호환된다.

이전 컬럼 데이터를 새로운 컬럼 데이터로 복사하는, 데이터 병합 마이그레이션(`src/main/resources/db/migration/V4__Copy_result_into_sum_column.sql`)을 생성한다.

	update CALCULATION
	set CALCULATION.sum = CALCULATION.result
	where CALCULATION.sum is null;

마이그레이션을 실행한다.

	$ ./gradlew flywayMigrate -i

다음 단계는 코드에서 이전 컬럼을 제거하는 것이다. `Calculation.java` 클래스에서 'result'를 사용하는 모든 코드를 삭제한다. 그런 다음, 서비스를 시작하고 데이터베이스의 데이터를 확인한다.

	$ ./gradlew bootRun

마지막으로, 데이터베이스에서 이전 컬럼을 삭제한다. 다음 내용으로  `src/main/resources/db/migration/V5__Drop_result_column.sql`을 생성한다.

	alter table CALCULATION
	drop column RESULT;

마이그레이션을 실행하고 서비스를 시작한 후, 데이터베이스를 다시 확인한다.

	$ ./gradlew flywayMigrate -i
	$ ./gradlew bootRun

## 연습 문제 해법

### 연습 1: Flyway로 MySQL에서 하위 버전과 호환되지 않는 변경 사항을 만든다.

[exercise1](exercise1) 디렉토리에는 초기 Flyway 마이그레이션이 있다.

다음 명령으로 MySQL 도커 컨테이너를 시작한다.

	$ docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123 -d mysql

MySQL 클라이언트 CLI (`mysql-client`)와 `flyway` 명령어 도구를 설치한다. 참고로, 관련 클라이언트 대신 도커 이미지를 사용할 수 있다.

예제 데이터베이스 `test`를 생성한다.

	$ mysql -h 127.0.0.1 -u root -p'abc123' -P 3306
	mysql> create database test;
	mysql> use test;

`users` 테이블을 생성하는 초기 마이그레이션을 실행한다.

	$ flyway -user=root -password=abc123 -url=jdbc:mysql://127.0.0.1:3306/test -locations=filesystem:. migrate

MySQL 클라이언트에 일부 데이터를 삽입한다.

	mysql> INSERT INTO USERS(ID, EMAIL, PASSWORD) VALUES(1, 'rafal@leszko.com','ala123');
	mysql> INSERT INTO USERS(ID, EMAIL, PASSWORD) VALUES(2, 'maria@leszko.com','123456');
	mysql> INSERT INTO USERS(ID, EMAIL, PASSWORD) VALUES(3, 'roza@leszko.com','password');

새로운 컬럼을 추가하는 `V2__Create_hashed_password_column.sql` 마이그레이션을 생성한다.

	alter table USERS
	add HASHED_PASSWORD varchar(100);

마이그레이션을 적용한다.

	$ flyway -user=root -password=abc123 -url=jdbc:mysql://127.0.0.1:3306/test -locations=filesystem:. migrate

데이터를 `PASSWORD`에서`HASHED_PASSWORD`로 복사하는 마이그레이션을 생성한다. 이름은 `V3__Copy_password_into_hashed_password.sql`로 지정한다.

	update USERS
	set USERS.HASHED_PASSWORD = MD5(USERS.PASSWORD);

마이그레이션을 적용한다.

	$ flyway -user=root -password=abc123 -url=jdbc:mysql://127.0.0.1:3306/test -locations=filesystem:. migrate

MySQL 클라이언트로 데이터가 복사되었는지 확인한다.

`PASSWORD` 컬럼을 삭제하는 마이그레이션을 생성한다. 이름은 `V4__Drop_password_column.sql`로 지정한다.

	alter table USERS
	drop column PASSWORD;

마이그레이션을 적용하고, MySQL 클라이언트로 결과를 확인한다.

	$ flyway -user=root -password=abc123 -url=jdbc:mysql://127.0.0.1:3306/test -locations=filesystem:. migrate
	mysql> select * from USERS;
	+----+------------------+----------------------------------+
	| ID | EMAIL            | HASHED_PASSWORD                  |
	+----+------------------+----------------------------------+
	|  1 | rafal@leszko.com | 77da6e74372583260cc783e8bdbe5b37 |
	|  2 | maria@leszko.com | e10adc3949ba59abbe56e057f20f883e |
	|  3 | roza@leszko.com  | 5f4dcc3b5aa765d61d8327deb882cf99 |
	+----+------------------+----------------------------------+

### 연습 2: 그래들 프로젝트 단위 테스트를 빌드하는 젠킨스 공유 라이브러리를 생성한다.

[exercise2](exercise2) 디렉토리에는 젠킨스 공유 라이브러리용 코드가 있다.

이를 사용하려면, 별도의 리포지토리에 복사하고 젠킨에서 가리키도록 설정해야 한다.