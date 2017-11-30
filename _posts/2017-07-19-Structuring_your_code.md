---
layout: post
title: "SpringBoot <br> (Structuring your code)"
author: "이호종"
comments: true
categories: [dev]
---

## 14. Structuring your code
스프링 부트는 구동하기 위한 어떤 특정한 구조도 요구하지 않지만, 몇 가지 좋은 예제는 있다.

### 14.1 Using the "default" package
`package`선언 없는 클래스는 "default package"로 간주된다. "default package"의 사용은 일반적으로 권장되지 않고 피해야 한다. 스프링 부트 어플리케이션에서는 `@ComponentScan`, `@EntityScan`, `@SpringBootApplication` 어노테이션을 사용할 때, 모든 jar로 부터 모든 클래스를 읽어야 하는 문제가 발생한다.

> 일반적으로 Java에서 사용하는 도메인을 뒤집은 형태의 package 명명법을 사용하는 것을 추천한다. (e.g. `com.example.project`).

### 14.2 Location the main application class
일반적으로 메인 어플리케이션 클래스는 루트 패키지에 놓는 것을 추천한다. `@EnableAutoConfiguration` 어노테이션은 대개 메인 클래스에 있고 특정 클래스를 찾기 위한 기본 패키지임을 암묵적으로 정의한다. JPA 어플리케이션을 만들 때, `@EnableAutoConfiguration`이 붙은 클래스의 패키지는 `@Entity`를 찾는데 사용된다.

루트 패키지를 사용하는 것은 `basePackage`를 특정할 필요 없이 `@ComponentScan` 어노테이션을 사용하는 것을 허용한다. 또한 메인 클래스가 루트 패키지에 있는 경우, `@SpringBootApplication` 어노테이션을 사용할 수 있다.

보통의 형태는 다음과 같다.

```
com
 +- example
     +- myproject
         +- Application.java
         |
         +- domain
         |   +- Customer.java
         |   +- CustomerRepository.java
         |
         +- service
         |   +- CustomerService.java
         |
         +- web
             +- CustomerController.java
```

`Application.java` 파일은 기본 `@Configuration`과 함께 `main` 메소드를 선언한다.

```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

***

## 15. Configuration classes
스프링 부트는 자바기반 설정을 선호한다. XML 소스를 사용한 `SpringApplication`을 사용하는 것이 가능하지만 일반적으로 주 소스는 하나의 `@Configuration` 클래스로 만드는 것이 좋다. 보편적으로 `main` 메소드가 있는 클래스는 주 `@Configuration`의 후보이다.

> 인터넷에 배포되어 있는 많은 스프링 설정 예제는 XML 설정을 사용하였다. 가능하면 동등한 자바 기반 설정을 사용해보라. `Enable*` 어노테이션을 찾는 것으로 시작할 수 있다.

### 15.1 Importing additional configuration classes
하나의 클래스에 모든 `@Configuration`을 넣을 필요는 없다. `@Import` 어노테이션을 사용해 추가 설정 클래스를 추가 할 수 있다. 다른 방법으로는 `@ComponentScan`을 사용해 `@Configuration` 클래스를 폼함한 모든 스프링 요소를 자동으로 추가할 수 있다.

### 15.2 Importing XML configuration
XML 기반 설정을 사용해야 한더라도 `@Configuration`클래스를 사용한 클래스로 시작하는 것을 추천한다. 추가적으로 `@ImportResource`어노테이션을 사용하여 XML 설정 파일을 읽어올 수 있다.

***

## 16. Auto-configuration
스프링 부트 자동 설정(auto-configuration)은 추가한 jar dependency를 기반으로 스프링 어플리케이션을 자동으로 설정하도록 시도한다. `HSQLDB`가 클래스패스에 있다면 수동으로 어느 DB 커넥션 빈(bean)을 설정하지 않아도 메모리 기반 DB를 자동 설정한다.

`@Configuration`클래스 중 하나에 `@EnableAutoConfiguration` 또는 `@SpringBootApplication`을 추가하여 자동 설정을 선택한다.

> 딱 하나의 `@EnableAutoConfiguration`을 추가해야 한다. 일반적으로 주 `@Configuration`클래스에 추가한다.

### 16.1 Gradually replacing auto-configuration (점차적인 자동 설정의 대체)
자동설정은 비침입적이다. 특정 시점에 자동설정의 일부를 대체하기 위해 사용자 설정을 정의할 수 있다. 사용자 `DataSource` 빈을 추가하면, 기본 내장된 DB 지원은 사라진다.

현재 자동 설정이 적용된 것과 이유를 알고 싶으면 `--debug` 스위치를 사용하여 어플리케이션을 실행시키면 된다. 코어 로거의 일부에 대한 디버그 로그와 자동설정에 대한 로그가 콘솔로 출력된다.

### 16.2 Disabling specific auto-configuration
자동설정을 원하지 않는 클래스는 `@EnableAutoConfiguration`의 `exclude` 요소를 사용하여 비활성화 시킬수 있다.

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

제외할 클래스가 클래스 패스에 없다면 `excludeName` 요소에 전체 이름을 넣는 것으로 제외할 수 있다. 마지막으로 제외할 자동설정 클래스 목록을 `spring.autoconfigure.exclude` 속성으로 제어할 수 있다.

> 제외할 클래스를 정의하는데 어노테이션과 속성 모두를 사용할 수 있다.

***
Spring Boot의 영어 문서를 이해하기 쉽게 한글로 옮길 계획이다.

http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle 를 참고하여 작성한다. 영어 독해에 어려움이 있어 일부는 구글번역을 이용하였다.

***