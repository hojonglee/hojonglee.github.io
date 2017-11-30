---
layout: post
title: "SpringBoot <br> (Spring Beans and dependency injection)"
author: "이호종"
comments: true
categories: [dev]
---

## 17. Spring Beans and dependency injection
빈을 정의하고 dependency를 주입하는 모든 표준 스프링 프레임웍 기술을 자유롭게 사용할 수 있다. 간단하게 `@ComponentScan`로 빈을 찾고 `@Autowired`로 생성자를 주입할 수 있다.

위에 제시된 루트 패키지에 어플리케이션 클래스를 위치시키는 코드로 구성하면 어떤 인자 없이 `@ComponentScan`을 추가할 수 있다. 모든 어플리케이션 구성요소(`@Component`, `@Service`, `@Repository`, `@Controller` 등)는 자동으로 스프링빈으로 등록된다.

다음은 `RiskAssessor` 빈을 얻기 위해 생성자 주입을 사용하는 `@Service` 빈 예제이다.

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...
}
```

하나의 생성자만 가지고 있는 빈은 `@Autowired`를 생략할 수 있다.
```java
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...
}
```

> 생성자 주입을 사용하면 `riskAssessor` 필드는 `final`로 표시되어 나중에 변경될 수 없는 것을 알려준다.

***

## 18. Using the @SpringBootApplication annotation
대부분의 스프링 부트 개발자는 메인 클래서에 `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan` 어노테이션을 붙인다. 이 어노테이션은 빈번하게 함께(특히 위에 우수 사례를 따른다면) 사용되기 때문에 스프링 부트는 편리하게 사용할 수 있도록 `@SpringBootApplication`을 대체재로 제공한다.

`@SpringBootApplication` 어노테이션은 `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan`을 기본 인자로 사용하는 것과 같다.
```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

> `@SpringBootApplication` 은 `@EnableAutoConfiguration`, `@ComponentScan`의 인자를 설정할 수 있는 별칭(alias)도 제공한다.

***

## 19. Running your application
어플리케이션을 jar로 패키징하고 내장된 HTTP 서버를 사용하는 중요한 이점 중 하나는 어플리케이션을 다른 것과 마찬가지로 실행시킬 수 있는 것이다. 스프링부트 어플리케이션을 디버그 하는 것은 역시 쉽다. 특별한 IDE 플러그인이 필요 없다.

> 이번 장은 jar 패키징 만을 다룬다. 어플리케이션을 war로 패키징한다면 서버와 IDE 문서를 참고하는 것이 좋다.

### 19.1 Running from an IDE
스프링부트 어플리케이션은 IDE에서 기본 자바 어플리케이션으로 구동할 수 있지만 먼저 프로젝트를 가져와야 한다. 가져오기 단계는 IDE와 빌드 시스템에 따라 다양하다. 대부분의 IDE는 메이븐 프로젝트를 직접 가져올 수 있다. 예를 들면 Eclipse에서는 `File` -> `Import...` -> `Existing Maven Projects`를 선택한다.

IDE에서 프로젝트를 직접 가져올 수 없다면 빌드 플러그인을 통해 IDE 기본 정보를 생성 할 수 있다. 메이븐은 [Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/)와 [IDEA](https://maven.apache.org/plugins/maven-idea-plugin/)에 대한 플러그인을 포함하고 Gradle은 [다양한 IDE](https://docs.gradle.org/3.4.1/userguide/userguide.html)에 대한 플러그인을 제공한다.

> 실수로 웹 어플리케이션을 2번 실행하여 "Port already in use" 에러가 발생한다. STS 사용자는 `Run`대신에 `Relaunch`를 사용하여 기존 어플리케이션을 닫고 실행 시킬 수 있다.

### 19.2 Running as a packaged application
스프링부트 Maven 또는 Gradle을 사용하여 실행가능한 jar를 만들었다면 `java -jar`명령을 통하여 어플리케이션을 실행할 수 있다.

```
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar
```

어플리케이션에 원격 디버깅을 지워하도록 실행시킬 수 있다. 어플리케이션을 디버거에 붙이는 것을 허용한다.

```
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myproject-0.0.1-SNAPSHOT.jar
```

### 19.3 Using the Maven plugin
스프링부트 메이븐 플러그인은 빠르게 컴파일하고 실행하기 위해 `run` 목표를 포함한다. 어플리케이션은 IDE에서 실행되는 것과 같이 실행된다.

```
$ mvn spring-boot:run
```

또한, 유용한 시스템 환경변수를 사용할 수 있다.
```
$ export MAVEN_OPTS=-Xmx1024m
```

### 19.4 Using Gradle plugin
스프링 부트 Gradle 플러그인은 어플리케이션을 실행할 수 있는 `bootRun` 작업을 포함하고 있다. `org.springframework.boot`와 `java` 플러그인을 적용할 떄마다 `bootRun` 작업이 추가된다.

```
$ gradle bootRun
```

또한, 유용한 시스템 환경변수를 사용 할 수 있다.

```
$ export JAVA_OPTS=-Xmx1024m
```

### 19.5 Hot swapping
스프링부트 어플리케이션은 그냥 자바 어플리케이션이기 때문에 JVM 핫스왑은 즉시 사용할 수 있다. JVM 핫스왑은 대체할 수 있은 바이트코드로 인해 다소 제한된다. 좀 더 완전한 솔루션인 JRebel을 사용할 수 있다. `spring-boot-devtools` 모듈은 빠른 어플리케이션 재시작을 지원한다.

[20장 Developer tools](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools) 아래 [Hot swapping "How-to"](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#howto-hotswapping)에서 자세한 내용이 있다.

***
Spring Boot의 영어 문서를 이해하기 쉽게 한글로 옮길 계획이다.

http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle 를 참고하여 작성한다. 영어 독해에 어려움이 있어 일부는 구글번역을 이용하였다.

***