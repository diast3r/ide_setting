#	Index
- [개발환경](#개발환경)
- [환경세팅](#환경세팅)
	* [플러그인 설치](#플러그인-설치)
	* [템플릿 추가](#템플릿-추가)
		* [HTML 템플릿 추가](#html-템플릿-추가)
		* [XML 템플릿 추가](#xml-템플릿-추가)
	* [프로젝트 생성](#프로젝트-생성)
		* [새 프로젝트 생성](#새-프로젝트-생성)
		* [의존성 추가](#의존성-추가)
		* [DB 연결 설정](#db-연결-설정)
		* [DB 예외처리](#db-예외처리)
		* [Lombok 설치](#lombok-설치)
	* [테스트](#테스트)
		* [디버그 설정](#디버그-설정)
		* [Thymeleaf, Jackson 라이브러리 작동 확인](#thymeleaf-jackson-라이브러리-작동-확인)
* [형상관리 (GitHub, SourceTree)](#형상관리-github-sourcetree)
<br><br>

#	개발환경
*	Eclipse(STS)  
*	java: jdk-17  
*	Lombok  
*	MyBatis  
*	Spring Data JPA  
*	MySQL  
<br><br>


#	환경세팅
1.	플러그인 설치
	-

	help - eclipse marketplace
	java and web 검색해서 jsp 지원하는 톱니바퀴모양 플러그인 설치  
	=>	html템플릿 추가 등을 위한 플러그인  
	잠시 후 창이 뜨면 전부 선택한 후 trust selected => sts 재시작  
	
2.	템플릿 추가  
	-
	*	HTML 템플릿 추가  
		-
		html 템플릿을 추가해서 새로 만드는 html에 자동으로 타임리프 문법 적용  
		Window > Preferences > Web > HTML Files > Editor > Templates에서 New 메뉴 클릭  
		이름, 설명, 반드시 New HTML로 선택할 것  
		pattern에 다음 내용 복붙하기  

		```html
		<!DOCTYPE html>
		<html 
				xmlns:th="http://www.thymeleaf.org"
				xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
				layout:decorate="~{layout/defaultLayout}">
		<section layout:fragment="content" class="contents d-flex justify-content-center">

		</section>

		<th:block layout:fragment="script">
			<script>

			</script>
		</th:block>
		```
		
	*	XML 템플릿 추가  
		-
		Window > Preferences > XML > XML Files > Editor > Templates에서 New 메뉴 클릭  
		이름, 설명, 반드시 New XML로 선택할 것  
		pattern에 다음 내용 복붙하기  

		```xml
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
			"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		```
		
3.	프로젝트 생성  
	-

	*	새 프로젝트 생성  
		-
		*	file > new > spring starter project로 선택  
		프로젝트명, 그룹, 패키지 변경 후 생성  
		build tool은 gradle-groovy 선택  
		*	**라이브러리 의존성 선택**  
			>Spring Boot DevTools  
			Lombok  
			Spring Configuration Processor  
			Spring Data JPA  
			MyBatis Framework  
			MySQL Driver  
			Thymeleaf  
			Spring Web
	
	*	의존성 추가
		-
		*	**p6spy 라이브러리 의존성 추가**  
			```groovy
			implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.0'
			```
			수행된 쿼리를 콘솔로 확인할 수 있음.  
			<br>
		*	**타임리프 레이아웃 의존성 추가**  
			```groovy
			implementation 'nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect'
			```
			`common.html`에 공통내용을 만들어놓으면 다른 html에서 이를 모듈처럼 갖다붙일 수 있게 하는 라이브러리. 사용법을 찾아보자.  
	*	DB 연결 설정
		-
		yml 파일?  
		
		>`.yml`은 `.properties`에서 중복되는 부분을 제거한 문법을 사용한다.  
		>이 문법은 **띄어쓰기에 굉장히 예민**하므로 복붙하고나서 손으로 직접 탭, 공백을 다시 써주는 것이 좋다.  
		>`yml`파일에 설정을 추가할 때는 중복을 제거하고 **기존 설정과 겹치는 상위 요소** 아래에 넣어줘야 한다.
		>```yml
		>#들여쓰기는 공백 2번
		>#기존 설정
		>spring:
		>  datasource:
		>    driver-class-name: com.mysql.cj.jdbc.Driver
		>```
		>```yml
		>#추가할 설정
		>spring:
		>  datasource:
		>    url: jdbc:mysql://localhost:(DB포트번호)/(데이터베이스명)?characterEncoding=UTF-8
		>```
		>```yml
		>#최종 설정
		>spring:
		>  datasource:
		>    driver-class-name: com.mysql.cj.jdbc.Driver
		>	url: jdbc:mysql://localhost:(DB포트번호)/(데이터베이스명)?characterEncoding=UTF-8
		>```
		>
		>`application.properties` 파일을 없애고 `application.yml`를 만들어서 사용하면 된다.  
		
		<br>

		*	**Camel case 컬럼명 설정 추가**  
			**DB의 테이블 컬럼명이 Camel case인 경우**에 필요한 설정.  
			Hibernate가 쿼리를 만들 때 column명을 Snake case로 바꾸는 것을 방지한다.  
			설정 전: `@Column(name = "loginId")` > `SELECT U.login_id FROM user;`  
			설정 전: `@Column(name = "loginId")` > `SELECT U.loginId FROM user;`  
			
			=> DB 컬럼명에 따라 필요 없을 수도 있음.  

			`application.yml`
			```yml
			spring:
			  jpa:
			    hibernate:
			      naming:
			      implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
			        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
			```
			<br>
		*	**`application.yml`에 db 연결 정보 추가.**  
			```yml
			spring:
			  datasource:
			  driver-class-name: com.mysql.cj.jdbc.Driver
			  url: jdbc:mysql://localhost:(DB포트번호)/(데이터베이스명)?characterEncoding=UTF-8
			  username: 계정이름
			  password: 비밀번호
			```
			포트번호, 데이터베이스명, 계정이름, 비밀번호  수정하기.<br><br>

		*	**`application.yml`에 MyBatis가 `xml` 파일을 읽을 경로 추가**  
			```yml
			mybatis:
			  mapper-locations: mappers/*Mapper.xml
			```
			MyBatis가 `xml` 파일을 읽는 경로는 `src/main/resources/mappers/...Mapper.xml`가 될 것임.<br><br>

		*	**`application.yml`에 JPA 설정 추가**
			```yml
			spring:
			  jpa:
			  show-sql: true
			  hibernate:
			    ddl-auto: none
			    properties:
			      hibernate:
			        format_sql: true
			```
			
	*	DB 예외처리  
		-

		DB 사용하기 전에 간단하게 작동을 확인할 때 추가하는 설정.  
		프로젝트를 만들 때 **DB 사용을 위한 의존성은 추가했지만 DB연결 설정을 안 했으면 에러 발생.**  
		<br>
		바로 DB연결까지 할 것이라면 생략.
		*	**DB 설정 에러를 예외처리하는 어노테이션 추가하기**  
			`(프로젝트명)Application.java` 파일 열기 ex) `SnsApplication.java`  
			`@SpringApplication`위에 붙이기.  

			```java
			@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
			```
			
	*	Lombok 설치  
		-

		*	**lombok.jar 다운로드**  
			"lombok download" 검색해서 lombok.jar 원하는 디렉토리에 저장  
			**디렉토리에 한글명 포함되면 안됨**  

		*	**`ini` 파일에 Lombok 경로 지정(STS 재시작 필요)**  
			STS 설치돼있는 폴더에서 `SpringToolSuite4.ini` 파일을 메모장으로 열고 lombok의 설치경로를 추가한다.  
			>-javaagent:(디렉토리 경로)\lombok.jar  

			ex) -javaagent:D:\lombok\lombok.jar  
			**경로가 잘못된 경우 sts 실행해도 아무 반응 없음.**
			<br><br>

4.	테스트  
	-

	*	디버그 설정  
		-
		서버를 디버그 모드로 실행하려고 하면 에러가 날 수 있음.  
		프로젝트 우클릭 > run as > run configurations > profile칸에 `--debug` 추가.  
		**오류 발생 여부와 상관 없이 한 번 이상 서버를 실행해야 설정할 수 있다.**  

	*	Thymeleaf, Jackson 라이브러리 작동 확인
		-
		1)	`com.(프로젝트명).test.TestController` 추가  
		2)	`@ResponseBody` + `return` `String` => HTML 잘 나오는지 확인  
		3)	`@ResponseBody` + `return` `Map` => JSON (Jackson 라이브러리 동작) 나오는지 확인  
		4)	`return` `String` + `"뷰 경로"`=> thymeleaf HTML 잘 나오는지 확인  
			*	아래 내용을 `application.yml` 파일에 추가.  
				```yml
				spring:
				  thymeleaf:
				    cache: false
				```
				
			*	`resources/templates/test/test.html` 파일 만들어서 Thymeleaf가 잘 출력하는지 확인.

		5)	DB 연동 확인 (MyBatis)  
			*	[DB 예외처리](#db-예외처리) 에서 추가한 어노테이션 제거.  

			*	`com.(프로젝트명).(도메인).mapper` 패키지와 그 안의 `...Mapper.java`인터페이스 생성  
				ex) com.memo.post.PostMapper  
			*	`mappers` 패키지 안에 
				mapper xml 작성할 때는 dtd 추가.  
				```xml
				<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
				"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
				```
				[XML 템플릿 추가](#xml-템플릿-추가)를 이미 했다면 새 파일을 만들 때 선택하기.

#	형상관리 (GitHub, SourceTree)
*	Repository 생성
	create할 때 경로는 `(workspace 경로)\(프로젝트)`  
	ex) `D:\Spring Project\memo\Memo`  
	**절대 .metadata가 있는 경로를 선택하면 안됨.**
