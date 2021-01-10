---
id: JPA1
title: JPA 시작하기-회원등록,수정
---
_[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic)을 보고 학습한 내용을 정리한 것입니다._

## JPA
* EntityManagerFactory 는 애플리케이션 로딩 시점에 하나만 생성.
* transaction 단위로 상품 담고 등의 로직은 EntityManager 를 통해서 
모든 데이터 변경은 transaction 안에서 이루어 져야함. 
엔티티 메니저는 쓰레드 간에 공유 x


* persistence.xml
    * jpa 설정 파일
    * /META-INF/persistence.xml 위치
    * persistence-unit name = "" 으로 이름 지정
    * javax.persistence 로 시작하는 건 JPA 표준 속성, hibernate 로 시작하는 건 하이버네이트 전용 속성 

    ````xml
    <persistence version="2.2"
        xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
        <persistence-unit name="hello">
            <properties>
                
                <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
                <property name="javax.persistence.jdbc.user" value="sa"/>
                <property name="javax.persistence.jdbc.password" value=""/>
                <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
                <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
    
            
                <property name="hibernate.show_sql" value="true"/>
                <property name="hibernate.format_sql" value="true"/>
                <property name="hibernate.use_sql_comments" value="true"/>
            </properties>
        </persistence-unit>
    </persistence>
    ````
### 데이터베이스 방언
- JPA는 특정 데이터베이스에 종속적이지 않다.
- 방언 : SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능
- **hibernate.dialect** 속성에 지정할 수 있다.
    - H2 : org.hibernate.dialect.H2Dialect
    - Oracle 10g : org.hibernate.dialect.Oracle10gDialect
    - MySQL : org.hibernate.dialect.MySQL5InnoDBDialect


### 회원 등록하기
* Persistence.createEntityManagerFactory("hello") 의 hello 는 <br/>persistence.xml 의 persistence-unit name="hello" 의 hello
* tx.begin(); : 트랜잭션 내에서 시작
* tx.commit(); : 트랜잭션 내에서 저장 후 종료

    ````java
    public class JpaMain {

        public static void main(String[] args) {
            EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

            EntityManager em = emf.createEntityManager();

            //모든 데이터에 대한 처리는 transaction 안에서 이루어져야함.
            EntityTransaction tx = em.getTransaction();
            tx.begin();
            
            try {
                Member member = new Member();
                member.setId(2L);
                member.setName("HelloB");
                em.persist(member);

                tx.commit();
            }catch (Exception e){
                tx.rollback();
            }finally {
                em.close();
            }
            emf.close();
        }
    }
    ````
    

### 회원 수정

````java
public class JpaMain {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        //모든 데이터에 대한 처리는 transaction 안에서 이루어져야함.
        EntityTransaction tx = em.getTransaction();
        tx.begin();
            
        try {
            Member findMember = em.find(Member.class, 1L);
            findMember.setName("HelloJPA");
            //em.persist(findMember);
            tx.commit();
        }catch (Exception e){
            tx.rollback();
        }finally {
            em.close();
        }
        emf.close();
    }
}
````

```java
Member findMember = em.find(Member.class, 1L);
findMember.setName("HelloJPA");
```

- primary key가 1L인 Member를 찾아서 이름을 HelloJPA 를 바꾸고 
em.persist(findMember) 를 통해 변경된 내용을 저장해 주어야 할 것 같지만
em.persist(findMember); 를 안써도 된다.

+ 마치 자바 컬렉션을 다루는 것처럼 설계 되었다.

 

* **위의 원리!!!!! (변경감지)**

    * JPA 를 통해서 Entity를 가져오면 JPA 가 관리를 한다.
    * JPA 가 변경이 되었는지 transaction commit 시점에 전부 확인을 해서
    변경이 되었으면 변경된 값으로 update를 해준다. 그 후 commit 이 된다.

 

+) JPQL

- SQL 을 추상화한 객체지향 쿼리 언어
- JPQL은 엔티티 객체를 대상으로 쿼리.
- SQL은 데이터베이스 테이블 대상으로 쿼리