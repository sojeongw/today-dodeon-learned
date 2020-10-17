# 프로젝트 생성

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>jpa-basic</groupId>
  <artifactId>hello-jpa</artifactId>
  <version>1.0.0</version>

  <dependencies>
    <!-- JPA 하이버네이트 -->
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-entitymanager</artifactId>
      <version>5.3.10.Final</version>
    </dependency>
    <!-- H2 데이터베이스 -->
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <version>1.4.199</version>
    </dependency>
  </dependencies>

  <properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

</project>
```

- Java 8
- Maven
    - groupId: jpa-basic
    - artifactId: ex1-hello-jpa
    - version: 1.0.0
    
## JPA 설정하기
### persistence.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
  xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

  <!-- JPA 이름 설정: 데이터베이스 당 하나를 부여한다. -->
  <persistence-unit name="hello">
    <properties>
      <!-- 필수 속성: DB 접속 정보 -->
      <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
      <property name="javax.persistence.jdbc.user" value="sa"/>
      <property name="javax.persistence.jdbc.password" value=""/>
      <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
      <!-- JPA는 특정 DB에 종속적이지 않아 hibernate 전용 옵션을 추가해줘야 한다.  -->
      <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
      <!-- 옵션 -->
      <property name="hibernate.show_sql" value="true"/>
      <property name="hibernate.format_sql" value="true"/>
      <property name="hibernate.use_sql_comments" value="true"/>
      <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
    </properties>
  </persistence-unit>
</persistence>
```

- resources/META-INF/persistence.xml
- persistence-unit
    - 이름 지정
- javax.persistence
    - JPA 표준 속성
- hibernate
    - 하이버네이트 전용 속성
    
![](../../.gitbook/assets/kimyounghan-orm-jpa/02/스크린샷%202021-03-13%20오후%203.20.17.png)

- H2Dialect
    - JPA는 특정 DB에 종속되지 않는다.
    - 각각의 DB가 제공하는 SQL 문법과 함수가 조금씩 다르다.
    - 이렇게 SQL 표준을 지키지 않는 특정 DB만의 고유한 기능을 방언이라고 한다.
    

