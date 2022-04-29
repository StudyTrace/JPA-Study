> 인프런- 김영한님의자바ORM 표준 JPA 프로그래밍 강의를 참고한 스터디입니다.
[https://www.inflearn.com/course/ORM-JPA-Basic/dashboard](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
> 

## SQL중심적 개발의 문제점

CRUD 반복

![https://images.velog.io/images/dudwls0505/post/e8e3225e-e08c-4bc1-b40a-2018e9c4b965/image.png](https://images.velog.io/images/dudwls0505/post/e8e3225e-e08c-4bc1-b40a-2018e9c4b965/image.png)

## 패러다임 불일치

객체와 테이블의 목적이 불일치 하기때문에 생기는 문제

### 1. 상속

객체는 상속, 테이블은 상속이없다.
**상속관계의 객체를 DB에 넣으려고한다면?**

ex ) 상속관계클래스가 2개면 일일히 다 insert..
, 조회하기위해선 join

### 2. 연관관계

![https://images.velog.io/images/dudwls0505/post/90a2062f-be1b-4f03-b63c-1b451c27558e/image.png](https://images.velog.io/images/dudwls0505/post/90a2062f-be1b-4f03-b63c-1b451c27558e/image.png)

객체는 참조,테이블은 외래키를 사용한다는 차이점

![https://images.velog.io/images/dudwls0505/post/67fee4b5-1fcd-4841-a4e6-46e531ddaa49/image.png](https://images.velog.io/images/dudwls0505/post/67fee4b5-1fcd-4841-a4e6-46e531ddaa49/image.png)

테이블: 조인으로 양방향참조
객체 : 연관관계를 맺을 객체들을 생성후 setter로 설정(단방향)

### 3.엔티티 신뢰문제

SQL을 실행할때 이미 탐색범위가 정해져서

쿼리문을 직접확인하지않으면 호출가능/불가능 여부를 알수가없음

```sql
SELECT M.*, T.*
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

```

```java
member.getTeam(); (o)
member.getOrder(); null

```

객체를 컬렉션에 저장하듯 DB에 저장하는방법에 대한 고민-> JPA

## JPA

- 라이브러리가 아닌 ORM을 사용하기위한 인터페이스
- 자바클래스와 db테이블을 매핑 (SQL매핑 X)

```java
public interface EntityManager {

	public void persist(Object entity);
    public <T> T merge(T entity);
    public void remove(Object entity);
    public <T> T find(Class<T> entityClass, Object primaryKey);
   	// more ....
}

```

JPA자체가 특정 기능을 하는것이 아닌 구현체가 존재한다.

## Hibernate(JPA의 구현체)

- JPA구현체의 한종류
- JPA가 DB와 객체를 매핑하기위한 인터페이스를 제공하고 Hibernate가 이 인터페이스를 구현한것

![https://images.velog.io/images/dudwls0505/post/cc6136f1-3b05-4236-8b1d-cde495a52546/image.png](https://images.velog.io/images/dudwls0505/post/cc6136f1-3b05-4236-8b1d-cde495a52546/image.png)

### 동작

```java
...

Member member = new Member();
jpa.find(memberId) // 조회
jpa.persist(member)  // 저장
member.setName("변경할 이름")// 수정
jpa.remove(member)//삭제

```

JDBC API를 사용하여 SQL을 호출해서 DB와 통신

## 성능

- 1차캐시의 존재로인해 약간의 성능 향상
- 트랜잭션을 지원하는 쓰기지연
- 지연로딩, 즉시로딩

옵션에 따라서 쿼리가 바뀐다.
중간에 쿼리문을 변경해야될 상황이 생겨도 옵션하나로 바꿀수있다.

### 데이터베이스 방언

특정 데이터베이스에 종족되지않음
페이징쿼리 (LIMIT, ROWNUM) ,
VARCHAR,VARCHAR2

의존성만 변경해주면 JPA가 알아서 번역

## JPA 구동방식

![https://images.velog.io/images/dudwls0505/post/a3949a64-4555-498a-8cc7-ecbb2111afff/image.png](https://images.velog.io/images/dudwls0505/post/a3949a64-4555-498a-8cc7-ecbb2111afff/image.png)

Persistence 라는 클래스가 META_INF/.xml 설정정보를 읽어서 EntityManagerFactory를 만들고, 팩토리에서 필요할떄마다 EntityManager를 만든다.

### EntityManagerFactory

DB당 1개가 생성된다.
생성비용이 커서 한번만 생성하고 공유해서 사용한다.
JPA에 따라서 커넥션풀도 생성한다. -> 생성비용이크다. 한번만생성하고 공유해서사용한다

### EntityManager

팩토리에 의해서 생성되고 JPA의 기능 대부분을 제공한다.

- 엔티티 매니저는 쓰레드간에 공유X (사용하고 버려야 한다).
• JPA의 모든 데이터 변경은 트랜잭션 안에서 실행
