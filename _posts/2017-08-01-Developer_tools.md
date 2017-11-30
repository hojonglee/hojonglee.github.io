---
layout: post
title: "SpringBoot <br> (Developer tools)"
author: "이호종"
comments: true
categories: [dev]
---

## 20. Developer tools
스프링 부트는 좀 더 쾌적한 어플리케이션 개발 경험을 만들어줄 수 있는 추가 개발도구 모음을 포함한다. `spring-boot-devtools` 모듈은 프로젝트에 포함되어 개발시간 기능을 추가로 제공할 수 있다. 개발도구 지원을 포함하기 위해 간단히 빌드에 모듈 의존성을 추가한다.

**maven.**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

**Gradle.**
```javascript
dependencies {
    compile("org.springframework.boot:spring-boot-devtools")
}
```

> devtools는 전체 패키지 어플리케이션이 수행될 때 자동으로 비활성화된다. 어플리케이션이 `java -jar`를 사용하여 실행되거나 특별한 클래스로더를 이용하여 시작된다면 "생산 어플리케이션(production application)"으로 간주된다. 의존성을 선택사항으로 표시하는 것은 devtools가 프로젝트를 사용하여 다른 모듈에 일시적으로 적용되는 것을 막는 가장 좋은 방법이다. Gradle은 선택적인 의존성을 즉시 사용할 수 있도록 지워하지 않는다. 그 동안에 `propdeps-plugin`을 확인하고 싶을지도 모른다. (? 이 부분 번역이 이상합니다.)

> 기본적으로 다시 패키징한 아카이브는 devtools를 포함하지 않는다. [특정 원격 devtools 특징](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-remote)을 포함시키려면 `excludeDevtools` 빌드 요소를 비활성화하여 설정에 포함시켜야 한다. 이 요소는 Maven과 Gradle 플러그인 모두를 지원한다.

### 20.1 Property defaults
스프링 부트에서 지원하는 몇몇 라이브러리는 성능 향상을 위해 캐시를 이용한다. [템플릿 엔진](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features-spring-mvc-template-engines)은 템플릿 파일을 반복적으로 분석하는 것을 피하기 위하여 컴파일된 템플릿을 캐시한다. 또한 스프링 MVC는 정정리소스를 서비스할때 response에 대한 HTTP 캐시 헤더를 추가할 수 있다.

캐싱은 산출물에서는 매우 유용함하지만 개발중에는 어플리케이션에서 변경되는 부분을 바로 확인할 수 없어서 생산성을 떨어뜨릴 수 있다. 이러한 이유로 spring-boot-devtools에서는 기본적으로 비활성화 되어있다.

캐시 옵션은 대게 `application.properties`파일에 설정한다. 예를 들면, Thymeleaf는 `spring.thymeleaf.cache` 속성을 제공한다. `spring-boot-devtools` 모듈은 이런 설정를 수동으로 설정하지 않고 자동으로 적절한 개발 시간 설정을 적용한다.

> 적용된 속성은 [DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.0.0.M2/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)에서 확인할 수 있다.

### 20.2 Automatic restart
`spring-boot-devtools`를 사용하는 어플리케이션은 클래스패스에 있는 파일이 변경되면 자동으로 재시작을 한다. IDE에서 작업을 할때 코드의 변화에 매우 빠른 피드백을 주는 유용한 기능이다. 기본적으로 클래스패스 디렉토리의 모든 요소에 대하여 변화를 모니터링한다. 정적자원과 뷰 템플릿 같은 특정요소는 [재시작을 필요로 하지 않는다](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-restart-exclude).

##### Triggering a restart
DevTools 클래스패스 자원을 모니터하기 때문에 재시작 할 수 있는 유일한 방법은 클래스패스를 업데이트 하는 것이다. 클래스패스를 업데이트 하는  방법은 IDE에 따라 다른다. 이클립스에서는 변경된 파일을 저장하는 방법으로 재시작한다. IntelliJ IDEA는 프로젝트 빌드(`Build -> Make Project`)가 동일한 효과를 가진다.

> DevTools는 제대로 동작하는 독립적인 어플리케이션 클래스 로더가 필요하므로 forking이 활성화되 있는 한 지원되는 빌드 플러그인(Maven, Gradle)을 통해서 어플리케이션을 시작할 수 있다. Gradle과 Maven은 클래스패스에 DevTools가 발견될때 기본적으로 동작한다.

> 자동 재시작은 LiveReload를 사용할때 매우 잘 동작한다. [아래](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-livereload)에서 자세한 내용을 확인할 수 있다. JRebel을 사용하면 자동 재시작은 동적클래스 재로딩을 위해 비활성화 된다. 다른 DevTools 특성(LiveReload 및 속성 재정의)은 여전히 사용가능하다.

> DevTools는 어플리케이션 컨텍스트의 종료 후크를 사용하여 다시 시작하는 동안 종료한다. 종료 후크(`SpringApplication.setRegisterShutdownHook(false)`)를 비활성화하면 동작하지 않는다.

> 클래스패스의 항목이 변경되어 재시작 여부를 결정할때, DevTools는 자동으로 `spring-boot`, `spring-boot-devtools`, `spring-boot-autoconfigure`, `spring-boot-actuator`, `spring-boot-starter` 프로젝트를 무시한다.

> DevTools는 `ApplicationContext`에서 사용하는 `ResourceLoader`를 사용자 정의해야 한다. 어플리케이션이 이미 제공했다면 그건 감싸져야(wrapping) 한다. `ApplicationContext`의 `getResource` 함수는 직접 재정의(override) 되는 것을 지원하지 않는다.

##### Restart vs Reload
재시작 기술을 두개의 클래스로더를 사용하는 방법으로 스프링 부트에서 제공한다. 변경되지 않은 클래스(예. 3rd-party jars)는 기본 클래스로더가 로드한다. 개발자가 개발한 클래스는 재시작 클래스로더가 로드한다. 어플리케이션이 재시작될때, 재시작 클래스로더는 버려지고 새로 하나를 생성한다. 이 방법은 기본 클래스로더가 이미 사용가능하고 메모리에 올라와 있기 때문에 어플리케이션이 "Cold Start" 하는 것 보다 매우 빠르다.

만약 재시작이 빠르지 않거나, 클래스 로딩에 문제가 있으면 ZeroTurnaround에서 [JRebel](http://zeroturnaround.com/software/jrebel/)과 같은 재로딩 기술을 고려해야 한다. 클래스가 로드될때 클래스를 다시 작성하여 재로딩을 빠르게 해준다.

#### 20.2.1 Excluding resources
특정 자원은 변경되었을때 재시작을 일으킬 필요가 없다. 예를 들면, Themeleaf 템플릿은 변경 즉시 반영된다. 기본적으로 `/META-INF/maven`, `/META-INF/resources`, `/resources`, `/static`, `/public`, `/template`에서 자원이 변하는 것은 재시작을 유발하지 않지만 [live reload](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-livereload)를 유발한다. 이 예외요소를 사용자 설정하려면 `spring.devtools.restart.exlude` 요소를 사용하면 된다. 예를 들면 아래와 같이 설정하여 `/static`과 `/public`의 요소만 제외할 수 있다.

```
spring.devtools.restart.exlude=static/**, public/**
```

> 기본 설정을 그대로 두고 추가 예외처리를 하려면 `spring.devtools.restart.additioinal-exclude`요소를 사용하면된다.

#### 20.2.2 Watching additional paths
클래스패스에 없는 파일의 변화를 감지하여 재시작 혹은 재로드를 원할때가 있다. 이럴때 `spring.devtools.restart.additional-paths` 속성에 추가 경로를 등록하면 된다. [위에서 설명한](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-restart-exclude) `spring.devtools.restart.exclude` 속성을 사용하여 추가 경로 아래의 변경으로 인해 전체 재시작 또는 live reload가 유발되는지 여부를 제어할 수 있다.

#### 20.2.3 Disabling restart
재시작 요소를 사용하고 싶지 않으면 `spring.devtools.restart.enabled` 속성을 사용할 수 있다. 대부분의 경우에 `application.properties`에 설정한다. (이것은 여전히 재시작 클래스 로더를 초기화하지만 파일 변경을 감시하지는 않는다.)

*완전히* 재시작 지원을 비활성화 시킬 필요가 있다면 특정 라이브러리에서는 동작하지 않기 때문에 `SpringApplication.run(...)`이 호출되기 전에 `System` 속성를 설정할 필요가 있다.

```java
public static void main(Sting[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

#### 20.2.4 Using a trigger file
IDE를 사용하여 지속적으로 변경된 파일을 컴파일 한다면 특정 시점에 재시작이 유발되길 원한다면 "trigger file"을 사용할 수 있다. "trigger file"은 실제 재시작을 유발때 변경되어야 하는 특수 파일이다. 파일을 변경하는 것은 단지 검사를 유발한다. DevTools가 작업을 수행해야 하는 것을 감지한 경우에만 재시작을 한다. 트리거 파일은 수동 혹은 IDE 플러그인을 통해 수정된다.

트리거 파일을 사용하려면 `spring.devtools.restart.trigger-file` 속성을 사용한다.

> 모든 프로젝트가 동일한 방식으로 작동하도록 [전역설정](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-globalsettings)으로 `spring.devtools.restart.trigger-file`을 설정할 수 있다.

#### 20.2.5 Customizing the restart classloader
위 [Restart vs Reload](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-spring-boot-restart-vs-reload)에서 설명했듯이 재시작 기능은 두 클래스 로더를 사용하여 구현되어 있다. 대부분의 어플리케이션에서 이런 방법은 잘 작동하지만 때때로 클래스 로딩 이슈를 유발할 수 있다.

기본적으로 IDE에서 열린 프로젝트는 "restart" 클래스 로더를 사용하여 로드하고 모든 일반 `.jar` 파일은 "base" 클래스 로더를 사용하여 로드한다. 멀티 모듈 프로젝트를 진행하고 각 모듈은 IDE에 포함되어있지 않다면 사용자 정의가 필요하다. `META-INF/spring-devtools.properties` 파일을 생성해서 진행한다.

`spring-devtools.properties` 파일은 `restart.exclude.`와 `restart.include.`를 포함할 수 있다. `include` 요소는 "restart" 클래스로더로 가져오고 `exclude` 요소는 "base" 클래스로더에서 내려와야 하는 요소이다. 속성의 값은 클래스패스를 적용하는 정규식 패턴이다.

예를 들면
```ini
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```

> 모든 속성 키는 유일하다. 속성이 `restart.include.`나 `restart.exclude.`로 시작하는 한 고려된다.(?)

> 클래스패스로부터 모든 `/META-INF/spring-devtools.properties` 로드된다. 프로젝트 안의 파일 또는 프로젝트가 사용하는 라이브러리는 패키지할 수 있다.

#### 20.2.6 Known limitations
재시작 기능은 표준 `ObjectInputStream`을 사용하여 역직렬화 한 객체에서는 잘 작동하지 않는다. 역직렬화 데이터가 필요하다면 스프링의 `ConfigurableObjectInputStream`과 `Thread.currentThread().getContextClassLoader()`를 함께 사용해야 한다.

몇 3rd-party 라이브러리는 콘텍스트 클래스로더를 고려없이 역직렬화한다. 이런 문제가 있다면 라이브러리 제작자에게 수정 요청을 해야 한다.

### 20.3 LiveReload
`spring-boot-devtools` 모듈은 리소스가 변경될때 브라우저 리프레시를 유발할 수 있는 내장된 LiveReload 서버를 포함한다. LiveReload 브라우저 확장 모듈은 크롬, 파이어폭스, 사파리에 대해서 [livereload.com](http://livereload.com/extensions/)에서 무료로 사용가능하다.

LiveReload 서버를 사용하고 싶지 않으면 어플리케이션을 실행할때 `spring.devtools.livereload.enable` 속성을 `false`로 설정하면 된다.

> 한번에 하나의 LiveReload 서버만 실행할 수 있다. 어플리케이션을 실행하기 전 다른 LiveReload 서버가 실행되는지 확인해야 한다. IDE에서 여러 개의 어플리케이션을 시작한다면 첫번째 LiveReload만 지원된다.

### 20.4 Global settings
`$HOME` 디렉토리(파일이름은 "."으로 시작한다)에 `.spring-boot-devtools.properties` 파일을 추가하여 전역 DevTools 설정을 할 수 있다. 이 파일에 추가하는 모든 속성은 DevTools를 사용하는 *모든* 스프링부트 어플리케이션에 적용된다. 예를 들면, 아래와 같이 설정하면 [trigger file](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-restart-triggerfile)을 사용하여 항상 재시작을 설정할 수 있다.

##### ~/.spring-boot-devtools.properties.
```bash
spring.devtools.reload.trigger-file=.reloadtrigger
```

### 20.5 Remote applications
스프링부트 개발 도구는 단지 로컬 개발에만 한정하지 않는다. 어플리케이션을 원격으로 동작하려면 몇 가지 특성을 사용할 수 있다. 원격지원은 포함되어있고 활성화하려면 `devtools`가 패키지 아카이브에 포함되어있는지 확인해야 한다.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```
그리고 `spring.devtools.remote.secret`속성을 설정해야 한다.
```bash
spring.devtools.remote.secret=mysecret
```

> 원격 어플리케이션에서 `spring-boot-devtools`를 활성화하는 것은 보안위험이 있다. 산출물을 배포할때는 절대 활정화하면 안된다.

원격 DevTools 지원은 두 가지를 지원한다. 하나는 연결을 받아들이는 서버 쪽 종말점이고 다른 하나는 IDE에서 실행하는 클라이언트 어플리케이션이다. 서버 요소는 `spring.devtools.remote.secret` 속성이 자동으로 활성화된다. 클라이언트 요소는 수동으로 동작시켜야 한다.

#### 20.5.1 Running the remote client application
원격 클라이언트 어플리케이션은 IDE에서 동작되도록 설계되 있다. 연결할 원격 프로젝트로서 같은 클래스패스를 사용한 `org.springframework.boot.devtools.RemoteSpringApplication`을 실행시켜야 한다. 어플리케이션으로 전달할 non-option 인수는 연결할 원격 URL이어야 한다.

Eclipse나 STS를 사용하고 Cloud Foundry에 배포할 프로젝트 이름이 `my-app`일때, 
- `Run`메뉴에서 `Run Configurations...`를 선택한다.
- 새로운 `Java Application` "실행 설정(launch configuration)"을 생성한다.
- `my-app` 프로젝트를 찾는다.
- 메인 클래스로 `org.springframework.boot.devtools.RemoteSpringApplication`를 사용한다.
- `Program arguments`로 원격 URL(`https://myapp.cfapps.io`)을 설정한다.

원격 클라이언트 실행결과는 다음과 같다.
```
  .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote :: 2.0.0.M2

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```

> 원격 클라이언트는 실제 어플리케이션과 같은 클래스패스를 사용하기 때문에 직접 어플리케이션의 속성을 읽을 수 잇다. `spring.devtools.remote.secret` 속성을 읽고 인증정보를 서버에 전달하는 방법이다.

> 연결 프로토콜로 `https://`를 사용하는 것은 항상 유용하다. 트래픽은 암호화되고 비밀번호는 가로채질 수 없다.

> 원격 어플리케이션에 접속하기 위해 프록시를 사용한다면 `spring.devtools.remote.proxy.host`와 `spring.devtools.remote.proxy.prot` 속성을 설정하면 된다.

#### 20.5.2 Remote update
원격 클라이언트는 수정사항에 대하여 어플리케이션의 클래스패스를 [local restart](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#using-boot-devtools-restart)와 같은 방식으로 모니터링한다. 모든 수정된 자원은 원격 어플리케이션에 반영되고 (*필요하다면*) 재시작을 유발한다. 이것은 로컬에 없는 클라우드 서비스를 사용하는 기능을 반복한다면 유용하다. 일반적으로 원격 업데이트와 재시작은 전체를 다시 빌드하고 배포하는 주기보다 매우 빠르다.

> 원격 클라이언트가 실행될때 파일은 단지 모니터되고 있다. 원격 클라이언트 시작전에 파일을 변경하면 원격서버에 반영되지 않는다.

#### 20.5.3 Remote debug tunnel
자바 원격 디버깅은 원격 어플리케이션에서 이슈를 진단하는데 유용하다. 어플리케이션이 데이터센터 외부에 배포될때 원격 디버깅을 활성화시키는건 항상 가능하지는 않다. 원격 디버깅은 도커와 같은 컨테이너 기반 기술을 사용하면 설정이 어려울 수 있다.

이런 제한점을 돕기위해 DevTools는 HTTP로 동작하는 원격 디버그 트래픽의 터널링을 지원한다. 원격 클라이언트는 원격 디버거를 붙일 수 있는 포트 `8000`으로 올라온 로컬서버를 제공한다. 한번 연결이 되면 디버그 트래픽은 HTTP통신을 통해서 원격 어플리케이션에 전달된다. `spring.devtools.remote.debug.local-port` 속성을 사용하여 다른 포트를 사용할 수 있다.

원격 디버깅을 활성화하여 시작한 원격 어플리케이션을 확인할 필요가 있다. `JAVA_OPT`를 설정하여 구성할 수 있다. Cloud Foundry에서 manifest.yml을 추가하는 예이다.

```profile
---
	env:
    	JAVA_OPT: "-Xdebug -Xrunjdwp:server=y,transport=dt_socket,suspend=n"
```

> `address=NNNN` 옵션을 `-Xrunjdwp`에 전달하지 않아도 된다. 생략된 경우 자바는 랜덤한 포트를 선정한다.

> 인터넷을 통해 원격 서비스를 디버깅하는 것은 느릴 수 있고 IDE에서 타임아웃 시간을 증가시켜야 할 수도 있다. 이클립스에서 `Preferences...`에서 `Java` -> `Debug`를 선택하고 `Debugger timeout (ms)`를 좀 더 적당한 값으로 수정한다. (`60000`은 대부분의 상황에서 잘 동작한다.)

> IntelliJ IDEA에서 원격 디버그 터널링을 사용할 때, VM이 아닌 모든 스레드는 모든 브레이크 포인트는 중지되도록 설정해야 한다. 기본wjr으로 IntelliJ IDEA의 브레이크 포인트는 브레이크 포인트에 도달한 스레드를 일시 중지하는 것 뿐만 아니라 전체 VM을 전부 중지한다. 이것은 원격 디버그 터널을 관리하는 스레드를 일시 중단하여 원격 세션이 멈추는 부작용이 있다. IntelliJ IDEA에서 원격 디버그 터널을 사용할때 모든 브레이크 포인트는 VM이 아닌 스레드를 일시 중지하도록 구성되어야 한다. 자세한 내용은 [IDEA-165769](https://youtrack.jetbrains.com/issue/IDEA-165769)를 참고하라.

## 21. Packaging your application for production
실행가능한 jar는 산출물 배포에 사용할 수 있다. 스스로 모두 포함함으로써 클라우드 기반 배포에 이상적으로 적합하다.

health, auditing, metrix Rest JMX end-points와 같은 추가 "산출물 준비" 속성에 대해서 `spring-boot-actuator`를 추가하는 것을 고려하라. [Part V. "Spring Boot Actuator: Production-ready features"](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#production-ready)에 자세한 내용이 있다.

## 22. What to read next
스프링 부트를 사용하는 방법과 함게 따라야할 모범사례를 잘 이해해야 한다. 특정 [스프링 부트 기능](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#boot-features)을 깊이있게 학습할 것이다. 건너뛴다면 "[산출물 준비](http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle/#production-ready)" 속성에 대해서 읽을 수 있다.

***
Spring Boot의 영어 문서를 이해하기 쉽게 한글로 옮길 계획이다.

http://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/htmlsingle 를 참고하여 작성한다. 영어 독해에 어려움이 있어 일부는 구글번역을 이용하였다.

***