---
layout: post
title: "Spring boot에서 DBMS를 연동하는 방법 - JPA 기본편"
date: 2018-06-06 11:23:10 +0900
categories: [spring]
tags: [spring boot, JPA]
---

지난 포스트에서 간단한 REST API 개발에 대해서 알 수 있었습니다. 아주 쉽게 GET 메소드에 대한 것만 정의하였고 나머지에 대한 메소드는 다루지 않았는데요. 오늘은 모든 REST API를 사용해보면서 테스트 또한 진행할 것입니다.

그러면 DBMS 연동에 대한 이야기 시작하도록 하겠습니다.



## DBMS <--> Spring boot

PHP, Node.js 등에서 DBMS를 연동하기 위해서는 PDO를 사용하거나 mongoose 등의 라이브러리를 사용하는 방법이 존재합니다. 여기서 mongoose는 Node.js에서 MongoDB를 사용하기 위한 라이브러리이고, PDO는 PHP에서 MySQL 등의 DBMS를 연동하기 위한 객체입니다. 

Spring boot에서는 Spring과 마찬가지로 DBMS를 연동하기 위해 존재하는 2가지 방법이 있습니다.

- JPA
- MyBatis / iBatis

아마 기존의 개발자 분들이시라면 Mybatis / ibatis가 좀 더 익숙할 것입니다. XML 파일을 이용해서 질의를 정의하고 이를 메소드로 호출하는 방법이지요. 

JPA 또한 비슷합니다. 단, JPA는 XML를 사용하지 않고 인터페이스로 정의하여 질의를 정의하고 역시 이를 메소드로 호출하여 실행하는 방식입니다.

또 다른 점이 있다면 JPA는 CRUD 등을 종속할 수 있어 여러분들이 직접 CRUD를 만들지 않아도 자동으로 정의되어 있는 형태의 템플릿을 제공합니다. 예를 들어 DBMS에 어떠한 모델링에 해당되는 데이터를 추가하려 한다면 Mybatis에서는 insert 문을 직접 정의하여 메소드를 만들어야 합니다. 하지만 JPA에서는 CRUD가 이미 정의되어 있는 인터페이스가 존재하여 insert, update, delete 등 기본적인 질의문에 대해서는 여러분들이 직접 정의하지 않아도 정해진 메소드만 호출하면 바로 DBMS에 추가, 수정, 삭제가 가능합니다.



## What is JPA ?

그럼 이 JPA는 무엇이고 어떤 배경으로 탄생한 것일까요?

JPA는 Java Persistence API의 약자로 Java 언어에서의 프로그래밍 인터페이스 명세사항입니다. 정확하는 Java라기 보단 JVM이라고 해야 맞겠지만 통상 Java 언어를 위해 만들어진 것이기 때문에 Java 언어라고 이야기 하겠습니다. 

즉, Java 플랫폼을 사용하는 애플리케이션 내부 관계형 데이터의 관리를 위해 만들어진 인터페이스라고 볼 수 있죠.



## How to use

그럼 이제 JPA를 사용하는 방법에 대해 알아봅시다.

```groovy
buildscript {
    /*
    	...
    */
    
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-noarg:$kotlinVersion"
    }
}

apply plugin: 'kotlin-jpa'

dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
}
```

기존 프로젝트에서 계속 진행하시고자 할 떄는 build.gradle에서 위의 내용만을 추가하시면 됩니다.

![Spring_initializr-jpa](/media/images/spring/JPA-1.png)

혹은 새 프로젝트를 만들어 JPA를 사용하고자 할 경우에는 위와 같이 SQL 디펜던시에서 JPA를 체크해주시면 됩니다.

자 그럼 JPA를 추가했으니 이제 여러분들이 사용할 DBMS를 고르셔야 합니다. 저는 MySQL을 주로 사용하므로 MySQL을 예로 들어 설명하도록 하겠습니다.

```groovy
runtime('mysql:mysql-connector-java')
```

build.gradle에 MySQL connector를 런타임해줍니다. 이는 Java에서 제공하는 JDBC를 사용하겠다는 이야기입니다.

```properties
server.address=localhost
server.port=5000

# API 호출시, SQL 문을 콘솔에 출력한다.
spring.jpa.show-sql=true

# DDL 정의시 데이터베이스의 고유 기능을 사용합니다.
# ex) 테이블 생성, 삭제 등
spring.jpa.generate-ddl=true

# MySQL 을 사용할 것.
spring.jpa.database=mysql

# MySQL 설정
spring.datasource.url=jdbc:mysql://localhost:3306/jpaexampleNK?createDatabaseIfNotExist=true&useUnicode=true&characterEncoding=UTF-8&characterSetResults=UTF-8&useSSL=true
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# MySQL 상세 지정
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
```

이제 서버 애플리케이션에서 DBMS 설정을 하여야 합니다. 여러분들이 사용할 DBMS가 어느 호스트에 있는지, 포트주소는 몇번인지 어떤 DBMS를 사용할 것인지, 사용자 이름과 비밀번호도 지정해야겠죠?

먼저 맨 위의 address와 port 부분은 지난 포스트에서 다뤘던 호스트 주소와 포트 주소를 설정하는 부분입니다. 만약 외부에서 서비스를 하고자 할 경우, localhost가 아닌 외부에서 사용할 IP를 지정하셔야 합니다. localhost로 지정할 경우 로컬 외에는 접속이 불가능합니다.

DDL이라는 것은 Data Definition Language의 약자로 데이터베이스의 스키마를 생성하는 CREATE, 변경하는 ALTER, 제거하는 DROP/TRUNCATE(내용만 삭제)의 질의를 사용할 수 있도록 하는 것인데, 이것을 왜 true를 시켰냐면 여러분들이 현재 테스트를 하는 용도로 사용할 것임을 고려한 것입니다. 즉 여러분들이 MySQL에서 수동으로 스키마 생성을 하지 않고 Spring boot에서 스키마가 존재하지 않으면 자동으로 생성하기 위해 주는 것입니다. 만약 이것이 false로 되어 있으면 스키마가 없어 서버 애플리케이션에서 오류가 발생합니다. 스키마 자체가 삭제될 수 있는 위험도 있기 떄문에 테스트 이외 실 서비스에서는 사용하지 않는 것을 권장합니다.

MySQL의 URL은 여러분들이 사용할 DBMS의 서버 주소와 포트 주소를 URL 형식으로 입력합니다. 프로토콜은 jdbc:mysql 형태의 프로토콜로 뒤에 나오는 파라메터들은 다음과 같습니다.

- createDatabaseIfNotExist: 데이터베이스가 존재하지 않으면 자동으로 생성하는 옵션입니다.
- useUnicode: 유니코드 사용 여부를 설정합니다.
- characterEncoding: 문자열 인코딩 종류를 설정합니다.
- characterSetResult: 결과값의 인코딩 종류를 설정합니다.
- useSSL: SSL 사용 여부에 대한 설정입니다.

마지막으로 JDBC 사용을 위해 driver class name에 MySQL JDBC 드라이버 패키지 이름을 적어줍니다.

추가로 MySQL에서는 여러가지 DB Collection을 제공하는 MyISAM 등 여러가지 있습니다. 저는 여기서 InnoDB 형태를 사용할 것이어서 platform에 InnoDB를 정의하였습니다.

이제 모든 설정이 끝났으니 한 번 실습에 들어가보도록 하겠습니다.



## Example

간단한 모델로 상점에서 파는 Item을 모델링해보도록 하겠습니다. 상품에는 각 상품의 고유번호, 이름, 그리고 가격이 존재합니다. 이를 Spring boot에서 모델링하려면 Java 언어에서와 똑같이 클래스를 생성하고 그에 대한 Getter / Setter를 주거나 필요한 메소드를 생성하면 됩니다.

```java
package xyz.neonkid.jpaexample.Model;

import javax.persistence.Column;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import java.io.Serializable;
import lombok.Data;

@Entity
@Data
@Table(name = "items")
public class Item implements Serializable {
    public Item() {}
    
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", updatable = false, nullable = false)
    public Long id;

    @Column(name = "name", updatable = false, nullable = false)
    public String name;

    @Column(name = "price", updatable = false, nullable = false)
    public int price;
}
```

```kotlin
package xyz.neonkid.jpaexample.Model

import java.io.Serializable
import javax.persistence.*

@Entity
@Table(name = "items")
data class Item(@Column(name = "name", updatable = false, nullable = false) val name: String, @Column(name = "price", updatable = false, nullable = false) val price: Int) : Serializable {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", updatable = false, nullable = false)
    val Id: Long? = null
}
```

위에는 Java, 밑에는 Kotlin 코드입니다.

먼저 이 클래스가 DBMS에서 Entity로 사용될 것임을 명시하기 위해 어노테이션 Entity를 사용해줍니다. 이 부분은 Java와 같습니다. 코틀린에서는 Data class를 기본으로 제공하여 getter setter를 만들어줍니다. Java에서는 이를 처리하기 위해 수동으로 Getter / Setter를 만들어주거나 Lombok 라이브러리를 사용해야 했다면 코틀린에서는 이를 기본으로 제공하기 떄문에 조금 귀찮음을 덜어주었죠.

단, 코틀린에서 Data class를 사용할 떄는 멤버 변수들의 접근 지정자가 반드시 public 이어야 한다는 것입니다. private으로 할 경우 Getter / Setter가 만들어지지 않으므로 참고하시기 바랍니다.

본론으로 넘어가서 @Column 어노테이션은 DBMS의 칼럼을 설정하는 부분입니다. 기본적으로 필수로 줘야하는 부분은 아니지만 여러분들이 직접 칼럼에 대한 이름, 속성 등을 정의하고 싶다면 이 어노테이션을 사용하여 정의할 수 있습니다.

@Id 어노테이션은 이 멤버 변수가 DBMS에서 기본키(Primary key)임을 정의하는 것입니다. 이를 사용자가 직접 정의할 수도 있지만 저는 자동으로 Auto increment를 주기 원하여 @GeneratedValue라는 어노테이션을 한 개 더 사용했습니다. 

이제 Entity를 만들었으니 질의를 정의하는 방법에 대해 알아보겠습니다. 이 다음부터는 계속 Kotlin 언어를 사용하여 진행하도록 하겠습니다.

```kotlin
package xyz.neonkid.jpaexample.Model

import org.springframework.data.repository.CrudRepository
import org.springframework.stereotype.Repository

@Repository
interface ItemRepository : CrudRepository<Item, Long>
```

여기가 진짜로 JPA를 사용하는 방법이라고 할 수 있는 부분입니다. 먼저 interface를 한 개 만들어준 후 이를 Repository 어노테이션으로 정의해줍니다.

그리고 CrudRepository를 종속하는 데, 여기서 제네릭은 여러분들이 방금 만드신 상품 모델과 그리고 그를 기준점으로 잡는 기본키의 자료형을 넣어주시면 됩니다.

이제 REST API를 사용하여 DBMS에 정상적으로 동작하는지를 테스트해보도록 하겠습니다. 



## Test

이제 예제를 작성한 것이 잘 동작하는지를 테스트해보도록 하겠습니다. 테스트를 위해 저는 Postman이라는 유틸리티를 사용하였으며 curl 등을 사용하여도 무방합니다.

먼저 서버 애플리케이션을 실행해봅시다.

![MySQLConnector](/media/images/spring/JPA-2.png)

서버 애플리케이션을 실행하기 전에는 이렇게 아무런 데이터베이스가 생성되어 있지 않습니다.

![MySQLConnector-2](/media/images/spring/JPA-3.png)

서버 애플리케이션을 실행하고나면 이렇게 새로운 데이터베이스가 생성됩니다. 이는 저희가 application.properties에 DDL을 설정했기 때문입니다.

![Postman](/media/images/spring/JPA-4.png)

Postman을 이용하여 GET 메소드를 주게 되면 아무런 내용이 없음을 출력합니다. 

![Postman-2](/media/images/spring/JPA-5.png)

JSON을 사용하여 내용을 추가하여 POST 메소드로 보내주면 이렇게 추가되었다는 메시지가 나오게 됩니다.

![MySQLConnector-3](/media/images/spring/JPA-6.png)

DBMS에서도 아주 잘 추가 되었죠? 

![Postman-3](/media/images/spring/JPA-7.png)

다시 조회를 해보면 이렇게 정보가 나타납니다. 

엇, 그런데, 저는 Controller를 생성하지 않았습니다. 그런데 어떻게 REST API가 동작할 수 있을까요?
Spring Data JPA는 JPA가 직접 그 REST API를 만들어줍니다. 정말 쉽죠? 여러분들이 이전에 쓰는 Spring과는 아주 많이 다른 모습입니다. 실제 제가 해커톤에 나가서도 이렇게 Spring boot를 과감히 사용하여 개발할 수 있었던 것도 이와 같은 이유입니다. 평소 같았으면 절대 이렇게 못하죠.. 시간적인 부담도 있고요. 그렇기 때문에 초반에는 대부분 PHP나 Python, Javascript를 이용할 것입니다.



## 마치며..

여기까지 JPA에 대한 간단한 사용법에 대해 알아봤습니다. 

기존의 Spring 이었다면 많은 설정들과 기본적으로 사용해야 할 API도 직접 만들어줘야하는 번거로움이 아주 많았을 것이라 생각합니다.

하지만 Spring boot를 사용하면 이러한 간단한 설정만으로도 DBMS와 연동되면서 깔끔하게 만들어진 API를 사용해 쉽게 API 서버를 배포할 수 있다는 것을 보여드렸습니다.

다음 포스트에서는 복잡한 DBMS 쿼리를 사용한 JPA 고급편을 진행하도록 하겠습니다.