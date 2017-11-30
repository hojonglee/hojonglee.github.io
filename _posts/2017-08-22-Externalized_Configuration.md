---
layout: post
title: "SpringBoot <br> (Externalized Configuration)"
author: "이호종"
comments: true
categories: [dev]

---

## 24. Externalized Configuration
스프링부트는 설정을 외부에 두는 것을 허용한다. 그래서 같은 어플리케이션 코드를 다른 환경에서 수행할 수 있다. 설정을 외부에 두기 위해 속성(properties)파일, YAML 파일 환경변수와 명령줄인자를 사용할 수 있다. 속성값은 `@Value` 어노테이션을 사용하거나 스프링의 `Environment` 추상화를 통해 접근하거나 `@ConfigurationProperties`를 통해 [구조화된 객체에 바인딩하여](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-external-config-typesafe-configuration-properties) 빈에 직접 주입할 수 있다.

스프링부트는 값의 합리적인 재정의를 허락하도록 설계된 매우 특별한 `PropertySource` 순서를 사용한다. 속성은 다음 순서를 고려한다.

1. 홈디렉토리(DevTools가 활성화 되어있을때 `~/.spring-boot-devtools.properties`)에서 [DevTools 전역 설정 속성](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-globalsettings)
2. 테스트할때 [`@TestPropertySource`](http://docs.spring.io/spring/docs/5.0.0.RC2/javadoc-api/org/springframework/test/context/TestPropertySource.html) 어노테이션
3. 테스트할때 [`@SpringBootTest#properties`](http://docs.spring.io/spring-boot/docs/2.0.0.M2/api/org/springframework/boot/test/context/SpringBootTest.html) 어노테이션 속성
4. 명령줄 인자
5. `SPRING_APPLICATION_JSON`(환경변수 또는 시스템 속성에 내장된 인라인 JSON)의 속성
6. `ServletConfig` 초기 인자
7. `ServletContext` 초기 인자
8. `java:comp/env`의 JNDI 속성
9. 자바 시스템 속성값(`System.getProperties()`).
10. OS 환경 변수
11. `random.*`안에 속성을 가지고 있는 `RandomValuePropertySource`
12. 패키지된 jar 밖의 [프로파일 지정된 어플리케이션 속성값](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-external-config-profile-specific-properties) (`application-{profile]`.properties 와 YAML 변종)
13. jar 안에 패키지된 [프로파일 지정된 어플리케이션 속성값](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-external-config-profile-specific-properties) (`application-{profile]`.properties 와 YAML 변종)
14. 패키지된 jar 밖의 어플리케이션 속성값 (`application.properties`와 YAML 변종)
15. jar 안의 패키지된 어플리케이션 속성값 (`application.properties`와 YAML 변종)
16. `@Configuration` 클래스 안의 [`@PropertySource`](http://docs.spring.io/spring/docs/5.0.0.RC2/javadoc-api/org/springframework/context/annotation/PropertySource.html) 어노테이션
17. 기본 설정값 (`SpringApplication.setDefaultProperties`를 사용해 설정된)

구체적은 예제를 제공하기위해 `name`속성값을 사용하는 `@Component`를 개발하는 것을 가정하였다.

```java
import org.springframework.stereotype.*
import org.springframework.beans.factory.annotation.*

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...
}
```

어플리케이션 클래스패스(예를들면 jar 내부)에 `name`에 대해 합리적인 기본 설정값을 제공하는 `application.properties`를 가질 수 있다. 새 환경에서 구동될때, `application.properties`는 `name`을 재정의한 jar의 외부에 제공된다. 그리고 일회성 테스트에서 특정 명령줄 스위치로 구동할 수 있다. (예. `java -jar app.jar --name="Spring"`)

> `SPRING_APPLICATION_JSON` 속성값은 환경변수와 함께 명령줄로 제공될 수 있다. 유닉스(리눅스) 쉘에서 예제는:
> ```bash
> $ SPRING_APPLICATION_JSON='{"foo":{"bar":"spam"}}' java -jar myapp.jar
> 
> ```
> 이 예제에서는 스프링 `환경(Environment)`에서 `foo.bar=spam`으로 끝난다. 또한 시스템 변수에서 `spring.application.json`으로 JSON을 제공할 수 있다.
> 
> ```bash
> $ java -Dspring.application.json='{"foo":"bar"}' -jar myapp.jar
> ```
> 또는 명령줄 인자로
> ```bash
> $ java -jar myapp.jar --spring.application.json='{"foo":"bar"}'
> ```
> 또는 JNDI변수 `java:comp/env/spring.application.json`로써 제공할 수 있다.

### 24.1 Configuring random values
`RandomValuePropertySource`는 임의의 값을 주입하는데 유용하다 (예.비밀 또는 테스트 케이스에 넣기). 예를 들면 인티저, 롱, uuid 또는 문자열을 생성할 수 있다.

```bash
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

`random.int*` 문법은 `OPEN value (,max) CLOSE`이다. 여기서 `OPEN, CLOSE`는 문자이고 `value, max`는 정수이다. `max`가 주어지면 `value`는 최소값이고 `max`는 최대값이다(서로 다른값이어야 한다).

명령줄 속성이 `Environment`에 추가되는 것을 원하지 않는다면 `SpringApplication.setAddCommandLineProperties(false)`를 사용하여 비활성화 할 수 있다.

### 24.2 Accessing command line properties
기본으로 `SpringApplication`은 모든 명령줄 옵션 인자('--'로 시작하는 예. `--server.port=9000`)를 `property`로 변한하고 스프링 `환경`에 추가한다. 위에 말했듯이 명령줄 속성값은 항상 다른 속성값 소스보다 우선한다.

명령줄 속성값이 `환경`에 추가되길 원하지 않는다면 `SpringApplication.setAddCommandLineProperties(false)`를 사용하여 비활성화 할 수 있다.

### 24.3 Application property files
`SpringApplication`은 아래 기술된 위치와 스프링 `Environment`에 추가된 위치에서 `application.properties` 파일로부터 속성값을 읽어들인다.

1. 현재 디렉토리의 `/config` 하위 디렉토리
2. 현재 디렉토리
3. 클래스패스 `/config` 패키지
4. 클래스패스 기본위치(root)

이 목록은 우선순위로 정렬되었다(리스트에서 높은 순위의 속성이 아래 순위의 속성을 재정의 한다).

> '.properties' 대신에 [YAML (.yml) 파일을 사용](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-external-config-yaml)할 수 있다.

설정파일 이름으로 `application.properties` 대신 다른 이름을 사용하고 싶으면 `spring.config.name` 환경 속성값을 설정하여 변경할 수 있다. 또한 `spring.config.location` 환경 속성값을 사용하여 명시적 위치를 사용 할 수도 있다(콤마로 구분된 디렉토리 위치 또는 파일 경로의 목록).

```bash
$ java -jar myproject.jar --spring.config.name=myproject
```
또는
```bash
$ java -jar myproject.jar \
--spring.config.location=classpath:/default.properties,classpath:/override.properties
```

> `spring.config.name`과 `spring.config.location`은 어떤 파일이 로드되어야 하는지 매우 이른 시점에 결정한다. 그래서 두 속성은 환경 속성값으로써 정의 되어야 한다(일반적으로 OS 환경변수, 시스템 속성 또는 명령줄 인자).

`spring.config.location`이 디렉토리(파일이 아니라)를 포함하고 있다면 `/`(프로필 관련 파일 이름을 포함하여 로딩되기 전에 `spring.config.name`으로 생성된 이름이 추가된다.)로 끝나야 한다. `spring.config.location`에 기술된 파일은 프로필별 변형을 지워하지 않고 그대로 사용하고 모든 프로필별 속성값으로 재정의 된다.

기본 검색 경로 `classpath:,classpath:/config,file:,file:config/`는 `spring.config.location`의 값과 상관없이 항상 사용된다. 검색 경로는 제일 낮은 순위부터 제일 높은 순위로 정렬된다(`file:config/`가 가장 높다). 사용자 지정 위치를 정하면 모든 기본 위치보다 우선하며 같은 가장 낮은 순위와 가장 높은 순위 순서를 사용한다. 이렇게 `application.properties`안에 어플리케이션에 대한 기본 값을 설정할 수 있고 (또는 `spring.config.name`으로 선택한 다른 기본이름) 기본설정을 유지하는 다른 파일을 실행시점에 재정의 할 수 있다.

> 시스템 속성값이 대신 환경변수를 사용하면 대부분의 OS는 마침표고 구분된 키 이름을 사용할 수 없지만 대신 밑줄을 사용할 수 있다(예. `spring.config.name` 대신 `SPRING_CONFIG_NAME`).

> 컨테이너에서 구동한다면 JNDI 속성값(`java:com/env`안의) 또는 서블릿 컨텍스트 초기화 인자를 환경변수나 시스템 속성값 대신에 사용할 수 있다.

### 24.4 Profile-specific properties
`application.properties` 파일과 함께 프로필 특정 속성값은 `application-{profile}.properties` 명명규칙을 사용하여 정의할 수 있다. `환경`은 활설 프로필이 설정되어있지 않은 경우 사용되는 기본 프로필(기본적으로 `[default]`) 모음(예. 명시적으로 활정화된 프로필이 없으면 `application-default.properties`로 부터 속성값을 읽어온다)을 가지고 있다.

프로필 특정 속성값은 표준 `application.properties`와 같은 장소에서 읽어오며 프로필 특정 파일은 패치지된 jar 내부 또는 외부에 관계없이 항상 비 특정 파일을 대체한다.

여러 프로필이 기술되면 마지막 프로필이 적용된다. 예를들면 `spring.profiles.active`속성에 의해 기술된 프로필은 `SpringApplication` API를 통해 설정된 프로파일 다음에 추가되므로 우선 적용된다.

> `spring.config.location`에 파일을 지정한 경우 해당 파일의 프로필 관련 변형은 고려되지 않는다. 프로필 특정 속성값을 사용하고 싶다면 `spring.config.location`에 디렉토리를 사용하라.

### 24.5 Placeholders in properties
`application.properties`의 값은 사용될때, 기존 `환경`을 통해 필터링되어 이전에 정의된 값을 다시 참조할 수 있다. (예. 시스템 속성값)

```bash
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

> 기존 스프링 부트 속성값의 '짧은' 변형을 생성할 때 이 기술을 사용할 수 있다. 자세한 사용방법은 [Section 72.4, “Use ‘short’ command line arguments”](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#howto-use-short-command-line-arguments)을 참고하라.

### 24.6 Using YAML instead of Properties
[YAML](http://yaml.org/)은 JSON의 상위집합이고 계층적인 구성 데이터를 기술하기위한 아주 편리한 형식이다. `SpringApplicatioin` 클래스는 클래스패스에 [SnakeYAML](http://www.snakeyaml.org/) 라이브러리가 있을때 속성값(properties)에 대한 대체재로써 YAML을 자동으로 지원한다.


> 'Starters'를 사용한다면 SnakeYAML은 `spring-boot-starter`를 통해 자동으로 제공된다.

#### 24.6.1 Loading YAML
스프링 프레임워크는 YAML 문서를 읽는데 사용할 수 있는 두 가지 편리한 클래스를 제공한다. `YamlPropertiesFactoryBean`은 `Properties`로써 YAML을 읽어오고 `YamlMapFactoryBean`은 YAML을 `Map`으로 읽어온다.

예를 들면 아래 YAML 문서:
```
environments:
    dev:
        url: http://dev.bar.com
        name: Developer Setup
    prod:
        url: http://foo.bar.com
        name: My Cool App
```

속성값(properties)으로 변경하면:
```
environments.dev.url=http://dev.bar.com
environments.dev.name=Developer Setup
environments.prod.url=http://foo.bar.com
environments.prod.name=My Cool App
```

YAML 목록은 `[index]` 역 참조자(dereferencers)와 함께 속성 키로 표현된다. YAML 예는:

```
my:
   servers:
       - dev.bar.com
       - foo.bar.com
```

속성값(properties)으로 변경하면:

```
my.servers[0]=dev.bar.com
my.servers[1]=foo.bar.com
```

스프링 `DataBinder` 유틸리티(`@ConfigurationProperties`의 기능)를 사용하는 것과 같이 속성값에 묶으려면 `java.util.List` (또는 `Set`) 유형의 대상 빈에 속성이 있어야 하며 설정자(setter)를 제공하거나 여러 값을 초기화 해야한다. 예 위 속성을 이용해 묶는다.

```java
@ConfigurationProperties(prefix="my")
public class Config {

    private List<String> servers = new ArrayList<String>();

    public List<String> getServers() {
        return this.servers;
    }
}
```

> 재정의가 예상대로 동작하지 않을때 추가 관리가 필요하다. 위 예제에서 `my.servers`가 여러 장소에서 재정의될때 목록이 아니라 각 요소가 재정의 대상이다. 높은 우선순위의 `PropertySource`가 리스트를 재정의 할 수 있게 하려면 그것을 하나의 속성값으로 정의해야 한다.
> ```
> my:
   servers: dev.bar.com,foo.bar.com
> ```

#### 24.6.2 Exposing YAML as properties in the Spring Environment

`YamlPRopertySourceLoader` 클래스는 스프링 `환경`에서 `PropertySource`로써 YAML을 노출하는데 사용할 수 있다. 이것은 YAML 속성값을 접근하는 지시자 문법으로 친숙한 `@Value` 어노테이션을 사용하는 것을 허용한다.

#### 24.6.3 Multi-profile YAML documents

`spring.profile` 키를 사용하여 문서가 적용되는 시점을 나타냄으로써 하나의 파일에 여러개의 프로필 특정 YAML 문서를 기술할 수 있다.
예를 들면:
```
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production
server:
    address: 192.168.1.120
```

위 예제에서 `development` 프로필이 활성화 되면 `server.address` 속성값이 `127.0.0.1`이 된다. `development`와 `production` 프로필이 **비활성화** 상태이면 속성값은 `192.168.1.100`이다.

어플리케이션 컨텍스트가 시작될때 명시적으로 활성화된 프로필이 없으면 기본 프로필이 활성화된다. 그래서 YAML에 `security.user.password`에 대한 값은 **오직** "기본" 프로필에서만 허용된다.

```
server:
  port: 8000
---
spring:
  profiles: default
security:
  user:
    password: weak
```

이 예제에서 비밀번호는 어느 프로필에서 붙어있지 않기 때문에 항상 설정된다. 필요에 따라 다른 프로필에서 명시적으로 다시 설정해야 한다.

```
server:
  port: 8000
security:
  user:
    password: weak
```

"spring.profiles" 요소를 사용하여 지정된 스프링 프로필은 `!` 문자를 사용하여 선택적으로 무효화 될 수 있다. 무효화된 프로필과 무효화되지 않은 프로필은 한 문서에 함께 기술될 수 있으면 적어도 하나의 무효화되지 않은 프로필이 일치해야하며 무효화된 프로필은 일치하지 않을 수 있다.

#### 24.6.4 YAML shortcomings

YAML 파일은 `@PropertySource` 어노테이션을 통해 읽어들일 수 없다. 이런경우 properties 파일을 사용해야한다.

#### 24.6.5 Merging YAML lists

[위에서 보았듯이](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-external-config-loading-yaml), YAML 설정은 결국 properties 속성값으로 변경된다. 프로필을 통해 "목록" 속성값을 재정의 할때, 이 처리과정은 직관성이 떨어질 수 있다.

예를 들면 `MyPojo` 객체가 기본값이 `null`인 `name`과 `description` 속성이 있다고 가정하자 `FooProperties`에 `MyPojo`의 리스트를 노출해보자:

```java
@ConfigurationProperties("foo")
public class FooProperties {

    private final List<MyPojo> list = new ArrayList<>();

    public List<MyPojo> getList() {
        return this.list;
    }

}
```

아래 설정을 고려하자:

```yml
foo:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
foo:
  list:
    - name: my another name
```

`dev` 프로필이 활성화 상태가 아니면 `FooProperties.list`는 위에 정의된대로 하나의 `MyPojo`를 가진다. 그러나 `dev` 프로필이 활성화되면 `list`는 *여전히* 단지 하나의 객체(name은 "my another name"이고 description은 `null`)를 가진다. 이 설정은 리스트에 두번째 `MyPojo` 인스턴스를 추가하지 않고 객체를 합치지도 않는다.

collection이 여러 프로필에 기술 된 경우 높은 우선순위의 프로필이 사용된다.(오직 하나)

```yml
foo:
  list:
    - name: my name
      description: my description
    - name: another name
      description: another description
---
spring:
  profiles: dev
foo:
  list:
     - name: my another name
```

위 예제에서 `dev` 프로필이 활정화될때 `FooProperties.list`은 하나의 `MyPojo` 객체(name은 "my another name"이고 description은 `null`)를 가진다.

### 24.7 Type-safe Configuration Properties

특별히 여러 속성값을 사용하거나 데이터가 자연적으로 구조화어 있다면 설정 속성값을 주입하기 위해 `@Value("${property}")` 어노테이션을 사용하는 것은 때때로 다루기 어려울 수 있다. 스프링 부트는 강력하게 유형이 지정된 빈이 어플리케이션을의 구성을 관리하고 유효성 검사하는 속성값을 동작시키는 대안 방법을 제공한다.

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("foo")
public class FooProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    public boolean isEnabled() { ... }

    public void setEnabled(boolean enabled) { ... }

    public InetAddress getRemoteAddress() { ... }

    public void setRemoteAddress(InetAddress remoteAddress) { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        public String getUsername() { ... }

        public void setUsername(String username) { ... }

        public String getPassword() { ... }

        public void setPassword(String password) { ... }

        public List<String> getRoles() { ... }

        public void setRoles(List<String> roles) { ... }

    }
}
```

위에 정의된 POJO는 아래 속성값을 따른다:

- `foo.enable`, 기본값 `false`
- `foo.remote-address`, `String`으로부터 강제 변환된 타입
- `foo.security.username`, 이름이 속성의 이름으로 정의되는 중첩된 "security". 특히 반환 유형은 전혀 사용되지 않으며 `SecurityProperties` 일 수 있다.
- `foo.security.password`
- `foo.security.roles`, `String`의 collection

> 바인딩은 스프링 MVC와 같이 표준 Java Beans 속성 기술자를 통하기 때문에 getter, setter 함수는 보통의 경우 필수이다. 아래와 같은 경우에 setter는 생략할 수 있다.
> - Map 객체는 초기화 되는 만큼 바인더에 의해 변화되기 때문에 getter는 필요하지만 setter는 아니다.
> - Collection과 Array는 색인(일반적으로 YAML) 또는 콤마로 분류된 값(properties)를 사용하여 접근 가능하다. 후자의 경우 setter는 필수적이다. 항상 같은 유형으로 setter를 추가하는 것이 좋다. collection을 초기화 했으면 변경할 수 없는 것(위 예제와 같이)인지 확인하라
> - 중첩된 POJO 속성값이 초기화(위 예제의 `Security` 필드와 같이)되었다면 setter는 필요하지 않다. 기본 생성자를 사용하여 인스턴스를 바로 생성하려면 setter가 필요하다.
> getter와 setter를 자동으로 추가하기 위해 Project Lombok을 사용하는 경우가 있다. Lombok이 컨테이너에서 자동으로 객체를 인스턴스화 하는 것과 같은 특정 생성자를 생성하지 못하게 하라.

> [`@Value`와 `@ConfigurationProperties`의 차이점](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-external-config-vs-value)을 확인하라

또한 `@EnableConfigurationProperties` 어노테이션에 등록 할 속성 클래스를 나열해야 한다.

```Java
@Configuration
@EnableConfigurationProperties(FooProperties.class)
public class MyConfiguration {
}
```

> `@ConfigurationProperties` 빈이 위 방법으로 등록될때 빈은 전통적인 이름(`<prefix>-<fqn>`)을 갖는다. `<prefix>`는 `@ConfigurationProperties` 어노테이션에 기술된 환경 키 접두사이고 `<frn>`은 빈의 전체 식별가능한 이름이다. 어노테이션이 어떤 접두사도 제공하지 않으면 오직 빈의 전체 식별가능한 이름만을 사용한다.
> 위 예제의 빈 이름은 `foo-com.example.FooProperties`이다.

위 설정이 `FooProperties`에 대한 보통 빈을 생성할지라도 `@ConfigurationProperties`가 환경만을 다룰 것을 권장하고 특히 문맥에서 다른 빈을 주입하지 않는다. `@EnableConfigurationProperties` 어노테이션은 *또한* 자동으로 프로젝트에 적용되어서 `@ConfigurationProperties`와 함께 모든 *존재하는* 빈 어노테이션은 `환경`으로부터 설정된다. 위 `MyConfiguration`은 `FooProperties`가 이미 빈이라는 것을 확인함으로써 바로가기가 될 수 있다.

```java
@Component
@ConfigurationProperties(prefix="foo")
public class FooProperties {
    // ... see above
}
```

이 형식의 설정은 `SpringApplication` 외부 YAML 설정에서 특히 잘 동작한다.

```yml
# application.yml

foo:
    remote-address: 192.168.1.1
    security:
        username: foo
        roles:
          - USER
          - ADMIN

# additional configuration as required
```

`@ConfigurationProperties` 빈을 사용하여 작업하려면 다른 빈과 동일한 방식으로 주입하면 된다.

```java
@Service
public class MyService {

    private final FooProperties properties;

    @Autowired
    public MyService(FooProperties properties) {
        this.properties = properties;
    }

     //...

    @PostConstruct
    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        // ...
    }
}
```

> `@ConfigurationProperties`를 사용하는 것은 IDEs에서 특정 키에 대하여 자동완성 기능을 제공하기 위해 메타 데이터를 생성하는 것을 허용한다. 자세한 내용은 [Appendix B, Configuration meta-data](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#configuration-metadata)를 확인하라.

#### 24.7.1 Third-party configuration

아직 작성하지 못한 부분은 추후 작성할 예정임.


***

Spring Boot의 영어 문서를 이해하기 쉽게 한글로 옮길 계획이다.

http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle 를 참고하여 작성한다. 영어 독해에 어려움이 있어 일부는 구글번역을 이용하였다.

***