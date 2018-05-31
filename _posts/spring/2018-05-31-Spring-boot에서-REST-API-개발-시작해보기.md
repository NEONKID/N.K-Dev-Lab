---
layout: post
title: "Spring boot에서 REST API 개발 시작해보기"
date: 2018-05-31 11:23:10 +0900
categories: [spring]
tags: [spring boot, kotlin]
---



안녕하세요. N.K Dev Lab에 글을 안쓴지가 정말 오래되었네요. 올해는 저의 취업 시즌인 만큼 블로그에 글을 쓰는 것에 많이 소홀했었습니다. 더욱이 Dev Lab 리뉴얼과 관련하여 여러 일들이 있었는데, 저의 Dev Lab이 다시 Jekyll로 돌아오게 되었습니다.

이유는 여러가지가 있지만 이제 제가 취업을 하게 되면 현재 하고 있는 일들을 조금 미리 간소화 하는 작업이라고 보시면 될 것 같습니다. 한 가지 예를 들어, 이제 Dev Lab에 올리는 글은 저의 손에 의해 수동으로 NKLAB(Tistory)에 게시되지 않고 자동화 된 코드에 의해서 올라가게 되는 작업 등 Hugo 보다는 Jekyll이 좀 더 편하더군요.



본론으로 넘어가서, 오늘은 Spring boot에 대한 이야기를 하고자 합니다. 왜 갑자기 Spring boot? 사실 Spring boot를 따로 다룰 정도로 저는 그렇게 많은 관심이 있지 않았습니다. 하지만 안드로이드 앱을 개발하면서 느낀 것은 이제 더 이상 싱글 규모의 앱을 개발하기에는 한계치가 있다는 것을 느끼게 되었습니다. 처음에는 그를 발판삼아 Python의 Flask를 사용했었고, 지금은 Java, Kotlin과 밀접한 Spring boot를 사용하기로 하였습니다.



## Spring boot in Kotlin

원래 Spring은 Java 언어를 사용했죠. 그런데 문득 생각이 들었습니다. 저는 코틀린을 이제 배우려 하고 있고 가능한한 탈 자바를 하기 위한 노력을 진행 중입니다. 안드로이드도 똑같은 JVM 위에서 돌아가면서 코틀린을 사용할 수 있는데 그러면 Spring boot에서도 코틀린을 사용하는 방법이 있지 않을까?

여러 번 구글링 시도 동안 외국의 많은 개발자들은 실제로 이 Spring boot를 코틀린 언어로 사용하는 예제 프로젝트들이 굉장히 많이 있었고, 저는 이를 적용하기로 하였습니다.

![Spring_Initializer](/media/images/spring/1.png)

IntelliJ IDEA, Eclipse, 혹은 Web Spring Initializr 등을 이용해서 Spring boot 프로젝트를 생성합니다.

![Choose_Kotlin](/media/images/spring/2.png)

원하시는 이름을 넣으신 후, Kotlin 언어를 선택해줍니다. 저는 Maven보단, Gradle에 익숙하므로 Gradle을 선택했습니다만 프로젝트 타입은 여러분이 원하시는 것을 넣어도 무방합니다.

![Choose_package](/media/images/spring/3.png)

저희는 REST API를 만들 것이기 때문에 Rest Repositories 항목을 선택해줍니다.
이렇게 한 후, 완료 버튼을 클릭하면 Kotlin 언어를 사용할 수 있는 Spring boot 프로젝트가 생성됩니다.



## How to build

어떻게 스프링 부트 프로젝트가 Kotlin으로 동작할 수 있는 것일까요? 이를 알아보기 위해 build.gradle 파일을 열어봤습니다.

```groovy
buildscript {
    ext {
        kotlinVersion = '1.2.41'
        springBootVersion = '2.0.2.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:${kotlinVersion}")
        classpath("org.jetbrains.kotlin:kotlin-allopen:${kotlinVersion}")
    }
}

apply plugin: 'kotlin'
apply plugin: 'kotlin-spring'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'xyz.neonkid'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8
compileKotlin {
    kotlinOptions {
        freeCompilerArgs = ["-Xjsr305=strict"]
        jvmTarget = "1.8"
    }
}
compileTestKotlin {
    kotlinOptions {
        freeCompilerArgs = ["-Xjsr305=strict"]
        jvmTarget = "1.8"
    }
}

repositories {
    mavenCentral()
}


dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-rest')
    compile('com.fasterxml.jackson.module:jackson-module-kotlin')
    compile("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    compile("org.jetbrains.kotlin:kotlin-reflect")
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

```

일반 스프링 부트 프로젝트에 플러그인들이 몇 개 더 추가되어 있음을 알 수 있을 것입니다. kotlin-spring 플러그인과 코틀린 컴파일 옵션이 추가되어 있는데, 이들은 실제 안드로이드 프로젝트에서도 비슷하게 적용되며 스프링 부트 또한 이런 플러그인과 의존 라이브러리를 적용하여 코틀린 언어를 사용할 수 있습니다.

그럼 이제 본격적인 REST API를 개발해보도록 하겠습니다.



## Application 설정

서버 애플리케이션을 개발하기 전에 반드시 해야할 것이 있습니다. 그것은 바로 사용할 URL 주소와 포트 주소를 지정하는 것인데요. 스프링 부트 애플리케이션의 서버 설정은 application.properties 파일에서 할 수 있습니다.

```properties
server.address=localhost
server.port=8000
```

먼저 간단하게 서버 주소와 포트 주소만을 설정해보겠습니다. 차후에는 이 파일에서 DB 등 다양한 옵션을 정의할 수 있으며 다른 포스트에서 자세히 다뤄보도록 할 것입니다.

이제 서버 애플리케이션을 빌드해보면 ...

![properties](/media/images/spring/4.png)

이렇게 8000번 포트로 Listen이 되는 것을 알 수 있습니다.



## REST API 생성

서버도 동작하는 것을 확인했으니 이제 정말로 API를 만들어보도록 하겠습니다. 여기서는 REST API에 대한 충분한 이론이 있는 상태에서 시작하는 것이니, 혹시 REST API에 대해 잘 모르시겠다면 충분한 구글링으로 이론을 숙지하신 후 계속 읽어주시기 바랍니다.

![add_controller](/media/images/spring/5.png)

Entry Point 생성을 위해 아무 이름의 코틀린 클래스를 하나 생성해줍니다.

```kotlin
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestMethod
import org.springframework.web.bind.annotation.RestController

@RestController
class HomeController {
    @RequestMapping(value = ["/"], method = [RequestMethod.GET])
    fun Index() : String = "Hello Spring boot for Kotlin !"
}
```

클래스를 생성한 다음 아무런 이름의 메소드를 만들어줍니다. 메소드의 이름은 무엇을해도 상관없습니다. REST API 생성시 중요한 것은 RequestMapping 어노테이션입니다. 해당 어노테이션에는 다음과 같은 정보가 들어갑니다.

- value: Entry point 정보 (Array)
- method: REST API 메소드 (Array)

아마 기존의 Spring 개발자 분들이 계신다면 아마 이 방법 보다는 아래의 방법이 더 익숙하실 수도 있을 것입니다.

```kotlin
import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.ResponseBody

@Controller
class HomeController {
    @GetMapping
    @ResponseBody
    fun Index() : String = "Hello Spring boot for Kotlin !"
}
```

위에 보시는 코드는 맨 처음 적은 코드와 동일한 역할을 하는 코드입니다. 하지만 왜 이렇게 다른 것일까요? 저도 처음에 배웠을 때는 헷갈렸지만 바로 위에 있는 코드는 Spring Data REST 라는 디페던시가 나오기 이전에 REST API 구현을 위한 코드이고, 처음에 짰던 코드는 Spring Data REST를 의존한 코드입니다. 

차이가 무엇일까요? 일단 RestController 어노테이션을 사용할 경우 메소드에 @ResponseBody 어노테이션이 자동으로 붙게 됩니다. 따라서 굳이 수동으로 해당 어노테이션을 주지 않아도 자동으로 값이 반환되는 것을 알 수 있습니다. 그냥 반환하면 500 Error가 발생합니다.

한 가지 더, 코드가 좀 더 간결합니다. 기존 코드는 GET, POST 등의 메소드를 여러개 사용하고자 하는 경우 그에 해당하는 어노테이션을 여러개 붙여야 하지만 Data REST에서는 어노테이션 파라메터에 Array 형태가 적용되어 있어 한줄에 여러 메소드를 정의할 수 있습니다.



## REST API 테스트

그럼 이제 API를 테스트해보도록 하죠.

![API_TEST](/media/images/spring/6.png)

위와 같이 GET 메소드로 최상위 포인트를 접속하게 되면 우리가 정해준 Hello 메시지가 뜨는 것을 확인할 수 있습니다.



## 마치며...

여기까지 Spring boot에 대한 간단한 REST API 개발 방법에 대해 알아봤습니다. 한동안 Dev Lab 글이 휴식을 취한 이유가 조금 눈치를 채셨을 수도 있지만 그동안에 제가 다른 공부에 매진하고 있었습니다.

서버 애플리케이션 개발은 한 번 쯤 해보고 싶었던 분야이자 이제는 제가 소프트웨어 개발자로 성장하기 위한 필수 단계입니다. 과거에는 클라이언트 애플리케이션을 개발하는 것으로 만족했다면 이제는 서버 애플리케이션을 융합한 솔루션 하나를 스스로 개발해보는 것이 되었습니다.

이렇게 제 목표가 바뀌게 된 것은 세상의 변화에도 그 이유가 있습니다. 앞으로는 싱글 애플리케이션보다는 서버-클라아언트 모델의 솔루션을 더 많이 만나게 될 것이며 그를 위해서 제가 더욱 더 서버 애플리케이션 개발 공부에 매진해야 하는 생각이 가득했습니다.

앞으로 Dev Lab에서 안드로이드 등의 클라이언트 애플리케이션 뿐만 아니라 서버 애플리케이션에 대해서도 계속 포스팅 해 나가도록 하겠습니다.