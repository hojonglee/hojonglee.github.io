---
layout: post
title: "SpringBoot <br> (Build Systems)"
author: "이호종"
comments: true
categories: [dev]

---

# Part III. Using Spring Boot
이번 장은 스프링 부트를 이용하는 방법에 대한 좀 더 자세한 내용을 다룬다. build systems, auto-configuration, 구동방법에 대해 다룬다. 스프링 부트에 대한 특별한 내용은 없지만 개발을 좀 더 쉽게 접근할 수 있게 하는 약간의 추천 방법이 있다.

## 13. Build systems
의존성 관리를 지원하는 빌드 시스템(Maven, Gradle)을 강력히 추천한다.

### 13.1 Dependency management (의존성 관리)
스프링부트의 각 버전은 지원하는 dependency 리스트를 제공한다. 스프링 부트가 의존성 리스트와 버전을 관리하기 때문에 사용자가 빌드 구성에 버전을 명시할 필요가 없다. 스프링 부트를 업그레이드 할 경우에도 dependency 리스트는 일관된 방법으로 업그레이드 된다.

> 사용자의 필요에 의해 특정 버전의 dependency을 사용할 수 있다.

스프링 부트에서 제공하는 dependency 리스트에는 3rd-party 모듈뿐만 아니라 모든 스프링 모듈을 포함한다. 이 리스트는 Maven, Gradle 모두에서 사용할 수 있는 표준 BOM(`spring-boot-dependencies`)으로 사용할 수 있다.

> 각 스프링 부트의 배포판은 스프링 프레임워크의 기본 버전과 연관이 있으며, 사용자 특정 버전을 사용하지 않는 것을 **강력** 권고한다.

### 13.2 Maven

Maven 사용자는 기본사항이 포함되어있는 `spring-boot-starter-parent`를 상속받을 수 있다.
이 부모 프로젝트는 다음과 같은 기능을 제공한다.
- 기본 컴파일 레벨로 Java 1.8
- UTF-8 인코딩
- 공통 dependency에서 `<version>` 태그를  생략할 수 있는 `spring-boot-dependencies` POM에서 상속받은 공통 dependency 관리 영역
- 민감한 [resource filtering](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)
- 민감한 플러그인 설정 ([exec plugin](http://www.mojohaus.org/exec-maven-plugin), [surefire](https://maven.apache.org/surefire/maven-surefire-plugin/), [Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin), [shade](https://maven.apache.org/plugins/maven-shade-plugin/))
- 프로파일 구분 파일(e.g. `application-foo.properties`와 `application-foo.yml`)을 포함한 `application.properties`와 `application.yml`에 대한 민감한 리소스 필터링

스프링 형식의 지시자 (`${...}`)를 수용한 기본 설정파일 때문에 메이븐 필터링은 `@..@` 지시자를 사용하도록 변경하였음. (메이븐 요소 `reosource.delimiter` 를 변경사여 사용할 수 있음)

#### 13.2.1 Inheriting the starter parent

`parent` 항목을 설정하면 `spring-boot-starter-parent`를 상속한 프로젝트를 만들 수 있다.

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.0.M2</version>
</parent>
```
> 위 dependency에서 특정 스프링 부트 버전만 지정하면 된다. 추가 시작 모듈을 가져오는 경우 안전하게 버전 정보는 생략할 수 있다.

위 설정에서 프로젝트의 속성을 재정의하여 개별 dependency를 재정의 할 수 있다. 예를들면, Spring Data 버전을 업그레이드 할때 `pom.xml`에 다음 내용을 추가한다.

```xml
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

> `spring-boot-dependencies` [pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.M2/spring-boot-dependencies/pom.xml)에서 지원하는 속성을 확인하자.

#### 13.2.2 Using Spring Boot without the parent POM

모든 사람이 `spring-boot-starter-parent` POM을 상속받아 사용하는 것을 좋아하는 것은 아니다. 아마 회사에서 이미 사용하고 있는 표준 부모 POM을 가지고 있거나, 명시적으로 Maven 설정을 하는 것을 선호 할 수도 있다.
`spring-boot-starter-parent`을 사용하지 않아도 `scope=import` dependency를 이용하여 dependency 관리의 이점을 챙길 수 있다.

```xml
<dependencyManagement>
     <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.0.0.M2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

위 설정은 속성을 사용하여 개별 dependency를 재정의 할 수 없다. 같은 결과물을 얻으려면 프로젝트에 `dependencyManagement`에서 `spring-boot-dependencies` 앞에 요소를 추가해야 한다. 아래 POM을 참고하자

```xml
<dependencyManagement>
    <dependencies>
        <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Fowler-SR2</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.0.0.M2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

> 위 예제에서는 BOM을 지정하지만 모든 dependency 유형을 같은 방법으로 재정의 할 수 있다.

#### 13.2.3 Using the Spring Boot Maven plugin

스프링 부트는 실행가능한 jar를 만들수 있는 [Maven plugin](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#build-tool-plugins-maven-plugin)을 포함하고 있다. `<plugins>` 영역에 추가하여 사용할 수 있다.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

> 스프링 부트 스타터 parent pom을 사용하는 경우 플러그인을 추가하기 만하면 된다. 부모에 정의 된 설정을 변경하지 않으려면 플러그인을 구성 할 필요가 없다.

### 13.3 Gradle

스프링부트에서 Gradle을 사용하려면 다음을 참고하라
- Reference: [HTML](http://docs.spring.io/spring-boot/docs/2.0.0.M2/gradle-plugin//reference/html), [PDF](http://docs.spring.io/spring-boot/docs/2.0.0.M2/gradle-plugin//reference/pdf/spring-boot-gradle-plugin-reference.pdf)
- [API](http://docs.spring.io/spring-boot/docs/2.0.0.M2/gradle-plugin//api)

> 나는 Gradle을 사용하지 않기 때문에 Gradle 부분 문서는 건너뛰고 다음 기회에 Gradle을 공부하여 작성하기로 한다.

### 13.4 Ant

스프링 부트 프로젝트를 아파치 Ant+Ivy를 이용하여 빌드 할 수 있다. `spring-boot-antlib` "AntLib" 모듈은 Ant로 실행가능한 jar를 만들 수 있도록 도와준다.

> Ant역시 사용하지 않기 때문에 더 자세한 내용을 알고 싶으면 [공식문서](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-ant)를 참고하자

### 13.5 Starters

Starters는 사용자 어플리케이션에 포함시킬수 있는 편리한 dependency 설명자의 모음이다. 샘플코드를 찾고 dependency 설명자를 복사/붙여넣기 하는 것 없이 스프링과 관련 모듈을 쉽게 프로젝트에 포함시킬 수 있다. 예를 들면, 스프링과 JPA를 사용하려고 할 때, 그냥 `spring-boot-starter-data-jpa` dependency를 프로젝트에 포함시키면 된다.

starters는 프로젝트를 빠르게 구동시키고 일관되게 관리되는 dependency의 모음을 지원하는 많은 dependency를 포함하고 있다.

공식적인 starters는 유사한 이름으로 구성 되어있다. `spring-boot-starter-*` 인데, \*부분에는 특정 어플리케이션 이름이 들어간다. 이 이름은 starter를 찾을 때, 어떤 역할을 하는지 대략 알 수 있다. 많은 IDE에 Maven에서 이름으로 dependency를 찾을 수 있다. 예를 들면, Eclipse나 STS에서 적절한 플러그인을 설치하고 POM 편집기에서 `ctrl-space`를 누르고 "spring-boot-starter"를 입력하면 전체 리스트를 볼 수 있다.

스프링 부트의 starter 목록 및 자세한 설명은 [여기](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-starter)에서 확인 할 수 있다.

***
Spring Boot의 영어 문서를 이해하기 쉽게 한글로 옮길 계획이다.

http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle 를 참고하여 작성한다. 영어 독해에 어려움이 있어 일부는 구글번역을 이용하였다.

***