---
layout: post
title: "SpringBoot <br> (SpringApplication)"
author: "이호종"
comments: true
categories: [dev]
---

# Part IV. Spring Boot features
이번 섹션은 스프링부트의 자세한 내용을 다룬다. 독자가 사용하거나 사용자화 할 중요한 기능에 대해 배울 수 있다. 준비가 되지 않은 경우 [Part II. "Getting started"](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#getting-started)와 [Part III. "Using Spring Boot"](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot)섹션을 읽어 기초를 다질 수 있다.

## 23. SpringApplication
`SpringApplication` 클래스는 `main()` 메소드로 시작하는 스프링 어플리케이션을 부트스트랩하는 편리한 방법을 제공한다. 많은 경우에 정적 `SpringApplication.run` 메소드를 그냥 위임할 수 있다.

```java
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

어플리케이션을 실행하면 아래와 유사한 로그를 확인할 수 있다.

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.0.0.M2

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```

어플리케이션을 실행한 사용자와 같은 관련된 시작 정보를 포함한 `INFO`로그 메세지가 기본으로 보인다.

### 23.1 Startup failure
어플리케이션 시작에 실패하면 등록된 `FailureAnalyzers`는 전용 에러 메세지와 구체적인 조치를 보여준다. 이미 사용 중인 포트 `8080`으로 웹 어플리케이션을 실행하면 다음과 같은 메세지를 볼 수 있다.

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

> 스프링부트는 다양한 `FailureAnalyzer`를 제공하고 쉽게 [사용자가 추가](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#howto-failure-analyzer)할 수 있다.

실패 분석기가 예외처리를 할 수 없다면 무엇이 잘못되었는지 이해하기 쉬운 전체 자동설정 보고서를 보여준다. [`DEBUG`속성을 활성화](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-external-config)하거나 `org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer`에 대한 [`DEBUG` 로그를 활성화](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-custom-log-levels)하면 볼 수 있다.

아래와 같이 `java -jar`로 실행시킬때 `DEBUG` 속성을 활성화 할 수 있다.

```bash
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### 23.2 Customizing the Banner
시작할때 나오는 배너는 클래스패스에 `banner.txt`파일을 추가하거나 파일이 있는 위치를 `banner.location`을 설정하여 변경할 수 있다. `banner.charset`을 이용하여 (기본은 `UTF-8`) 파일의 문자 인코딩을 설정할 수 있다. 텍스트 파일 뿐만 아니라 `banner.gif`, `banner.jpg`, `banner.png`같은 이미지 파일을 클래스패스에 추가하거나 `banner.image.location` 속성을 설정할 수 있다. 이미지는 텍스트 배너 위에 ASCII 표현으로 변경되어 출력된다.

`banner.txt`파일안에 아래와 같은 변수를 사용할 수 있다.

##### Table 23.1 Banner variables
| Variable | Description |
|--------|--------|
|`${application.version}`|`MANIFEST.MF`에 선언되는 어플리케이션의 버전. 예를 들면 `Implementation-Version: 1.0`은 `1.0`으로 출력된다.|
|`${application.formatted-version}`|디스플레이를 위해 정의된 `MANIFEST.MF`에 선언되는 어플리케이션의 버전 (괄로로 묶고 버전숫자 앞에 `v`를 붙인다). 예를 들면 `(v1.0)`|
|`${spring-boot.version}`|사용하는 스프링부트의 버전 (`2.0.0.M2`).|
|`${spring-boot.formmatted-version}`|디스플레이를 위해 정의된 스프링부트의 버전 (`(v2.0.0.M2)`)|
|`${Ansi.NAME}`, `${AnsiColor.NAME}`, `${AnsiBackground.NAME}`, `${AnsiStyle.NAME}`|여기서 `NAME`은 안시 이스케이프 코드의 이름이다. 자세한 내용은 [AnsiPropertySource](https://github.com/spring-projects/spring-boot/tree/v2.0.0.M2/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)을 확인하라 |
|`${application.title}`|`MANIFEST.MF`에 선언되는 어플리케이션의 이름. 예를 들면, `Implementation-Title: MyApp` 은 `MyApp`으로 출력된다.|

> `SpringApplication.setBanner(...)` 함수는 배너를 프로그래밍적으로 생성하려면 사용할 수 있다. `org.springframework.boot.Banner` 인터페이스와 `printBanner()` 함수를 구현하여 사용한다.

`spring.main.banner-mode` 속성을 사용하여 배너가 설정된 로거(`log`)를 사용하여 `System.out`(`console`)으로 출력되는지 아니면 전혀 출력안되는지(`off`) 결정할 수 있다.

출력된 배너는 `springBootBanner` 이름으로 싱글톤 빈으로 등록된다.

> YAML은 `off`를 `false`로 매핑하므로 어플리케이션에서 배너가 비활성화시키려면 큰따옴표를 추가하면된다.
> ```yaml
> spring:
> 	main:
> 		banner-mode: "off"
> ```

### 23.3 Customizing SpringApplication
`SpringApplication` 기본 설정대신에 로컬 인스턴스를 생성하고 사용자화할 수 있다. 예를 들면 배너를 끄고 싶으면 아래와 같이 작성하면 된다.

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

> `SpringApplication`으로 전달된 생성자 인자는 스프링 빈을 설정한다. 대부분의 경우에 `@Configuration` 클래스를 참조한다. 그러나 XML 설정 또는 스캔된 패키지를 참조할 수 있다.

또한 `application.properties`파일을 사용하여 `SpringApplication`을 설정하는 것도 가능하다. 자세한 내용은 [챕터 24. Externalized Configuration](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-external-config)을 참고하자.

설정 옵션의 전체 목록은 [`SpringApplication` Javadoc](http://docs.spring.io/spring-boot/docs/2.0.0.M2/api/org/springframework/boot/SpringApplication.html)에서 확인할 수 있다.

### 23.4 Fluent builder API
`ApplicationContext` 구조 (부모/자식 관계가 있는 여러 컨텍스트)를 빌드하하거나 'fluent' 빌드 API를 사용하는걸 선호한다면 `SpringApplicationBuilder`를 사용할 수 있다.

`SpringApplicationBuilder`는 여러개의 함수 호출을 함께 묶고 계층관계로 생성한 `parent`와 `child` 메소드를 허용한다.

예:
```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```

> `ApplicationContext` 계층구조를 생성하는데 몇 가지 제한이 있다. 웹 구성요소는 **반드시** 자식 컨텍스트 안에 포함되어야하고, 같은 `Environment` 부모와 자식 컨텍스트에 함께 사용되어야 한다. 자세한 내용은 [`SpringApplicationBuilder` Javadoc](http://docs.spring.io/spring-boot/docs/2.0.0.M2/api/org/springframework/boot/builder/SpringApplicationBuilder.html)에서 확인하자.

### 23.5 Application events and listeners
[`ContextRefreshedEvent`](http://docs.spring.io/spring/docs/5.0.0.RC2/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)와 같은 일반적인 스프링 프레임워크 이벤트 외에도 `SpringApplication`은 몇가지 추가 어플리케이션 이벤트를 전송한다.

> 일부 이벤트는 실제로 `ApplicationContext`가 생성되기 전에 유발되어서 `@Bean`으로 리스너에 등록할 수 없다. `SpringApplication.addListeners(...)` 또는 `SpringApplicationBuilder.listeners(...) 함수를 통해서 등록할 수 있다.
> 리스너를 어플리케이션이 생성되는 방식과 상관없이 자동으로 등록하려면 `META-INF/spring.factories`파일에 프로젝트를 추가하고 리스너를 `org.springframework.context.ApplicationListener` 키를 사용하여 참조할 수 있다.
> ```ini
> org.springframework.context.ApplicationListener=com.example.project.MyListener
> ```

어플리케이션 이벤트는 아래 순서로 발생한다.
1. `ApplicationStartingEvent`는 어플리케이션의 실행이 시작될때 리스너와 초기화의 등록을 제외한 모든 처리 전에 발생한다.
2. `ApplicationEnvironmentPreparedEvent`는 컨텍스트에 사용 된 `Environment`가 알려져 있지만 컨텍스트가 생성되기 이전에 발생한다.
3. `ApplicationPreparedEvent`는 새로고침이 시작되기 전이지만 빈 정의가 로드된 다음에 발생한다.
4. `ApplicationReadyEvent`는 새로고침과 어플리케이션이 서비스 요청을 받을 준비가 되었다는 것을 알려주는 모든 관련 콜백이 처리된 후 발생한다.
5. `ApplicationFailedEvent`는 시작할때 예외가 있는 경우 발생한다.

> 대개 어플리케이션 이벤트를 사용할 필요가 없지만 이벤트가 있는 것을 알아두면 편리할 수 있다. 내부적으로 스프링 부트는 다양한 작업을 처리하기 위해 이벤트를 사용한다.

### 23.6 Web environment
`SpringApplication`은 사용자 대신에 올바른 타입의 `ApplicationContext`생성하려고 시도한다. 기본적으로 `AnnotationConfigApplicationContext` 또는 `AnnotationConfigServletWebServerApplicationContext`는 웹 어플리케이션을 개발하는지 아닌지에 따라서 사용된다.

'웹 환경'을 결정하는 데 사용하는 알고리즘은 매우 단순하다(몇 개의 클래스에가 있는 경우). 기본을 재정의할 필요가 있을때 `setWebEnvironment(boolean webEnvironment)`를 사용할 수 있다.

`setApplicationContextClass(...)`를 호출하는 방식으로 `ApplicationContext` 타입을 완전히 제어하는 것도 가능하다.

> JUnit 테스트 중 `SpringApplication`을 사용할 때 `setWebEnvironment(false)`를 호출하는 것도 바람직하다.

### 23.7 Accessing application arguments
`SpringApplication.run(...)`에 전달할 어플리케이션 인자에 접근할 필요가 있다면 `org.springframework.boot.ApplicationArguments` 빈을 주입할 수 있다. `ApplicationArguments` 인터페이스는 구문분석 된 `옵션` 및 `비옵션` 인자 뿐만 아니라 원시 `String[]` 인자에 대한 접근을 제공한다.

```java
import org.springframework.boot.*
import org.springframework.beans.factory.annotation.*
import org.springframework.stereotype.*

@Component
public class MyBean {

    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }

}
```

> 또한 스프링부트는 `CommandLinePropertySource`를 스프링 `환경`과 함께 등록한다. 이것은 `@Value` 어노테이션을 이용하여 간단한 어플리케이션 인자를 주입하게 해준다.

### 23.8 Using the ApplicationRunner or CommnadLineRunner
`SpringApplication`이 시작할때 특별한 코드가 실행되길 원한다면 `ApplicationRunner` 또는 `CommandLineRunner` 인터페이스를 구현하여 이용할 수 있다. 두 인터페이스는 같은 방법으로 동작하고 `SpringApplication.run(...)`이 완료되기 직전에 호출되는 단일 `run` 함수를 제공한다.

`CoomandLineRunner` 인터페이스는 단순 문자 배열로 인자에 접근하는 것을 제공하는 반면 `ApplicationRunner`가 위에 기술된대로 `ApplicationArguments` 인터페이스를 사용한다.

```java
import org.springframework.boot.*
import org.springframework.stereotype.*

@Component
public class MyBean implements CommandLineRunner {

    public void run(String... args) {
        // Do something...
    }
}
```

일부 `CommandLineRunner` 또는 `ApplicationRunner` 빈이 특별한 순서로 호출되어야 하게 정의한다면 추가로 `org.springframework.core.Ordered` 인터페이스를 구현하거나 `org.springframework.core.annotation.Order` 어노테이션을 사용할 수 있다.

### 23.9 Application exit
각 `SpringApplication`은 `ApplicationContext`가 정상적으로 종료되도록 JVM에 종료 훅을 등록한다. 모든 표준 스프링 생명주기 콜백(`DisposableBean` 인터페이스나 `@PreDestroy`어노테이션과 같은)을 사용할 수 있다.

어플리케이션이 종료될때 특변한 종료 코드를 반환하길 원한다면 추가로 빈은 `org.springframework.boot.ExitCodeGenerator` 인터페이스를 구현할 수 있다.

### 23.10 Admin features
어플리케이션에 대해 관리자 관련 기능을 `spring.application.admin.enabled` 속성을 특정하여 활성화 할 수 있다. 이것으로 플랫폼 `MBeanServer`에 [`SpringApplicationAdminMXBean`](https://github.com/spring-projects/spring-boot/tree/v2.0.0.M2/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java)을 표출한다. 이 기능을 스프링부트 어플리케이션을 원격으로 관리하기 위하여 사용한다. 또한 모든 서비스 랩퍼 구현에 유용하다.

> 어플리케이션이 수행되는 HTTP 포트를 알고 싶으면 키 `local.server.port`로 속성을 가져와라

> MBean이 어플리케이션을 종료하는 함수를 표출하기 위해 이 기능을 활성화할때는 주의하라.

***
Spring Boot의 영어 문서를 이해하기 쉽게 한글로 옮길 계획이다.

http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle 를 참고하여 작성한다. 영어 독해에 어려움이 있어 일부는 구글번역을 이용하였다.

***