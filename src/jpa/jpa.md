## JPA 기본 개념

### 등장 배경

객체지향적으로 설계를 할수록 관계형데이터베이스에 데이터를 저장하고 가져왔을 때 개발자가 설계한대로 객체 지향적으로 코드가 동작하기 위해서는 수 많은 sql문을 
작성해야 했었다.(객체와 관계형데이터베이스간의 패러다임의 불일치) 그래서 대부분이 SQL 중심적으로 개발을 했었고 이를 해결하고 싶어 나온것이 바로 JPA 
(Java Collection을 다루는 것 처럼 Entity를 다루자!)

기존 이를 해결하기 위해 자바진영에서 나왔던 EJB가 있었지만 오류도 많고 동작도 제대로 하지 않아 개빈 킹이라는 개발자가 하이버네이트를 만들었고 이를 많은 개발자들이 
동참해 오픈소스화가 된 이후 다시 자바 진영에서 하이버네이트에서 조금 더 다듬고 표준으로 JPA를 들고나왔다.

`EJB -> 하이버네이트 -> JPA`


- JPA (Java Persistence API)
- 자바 진영의 ORM (Object relational mapping) 기술 표준
  - ORM : 객체와 관계형 데이터 베이스간의 데이터를 주고 받을 때 자동으로 매핑해주는 것
- 애플리케이션과 JDBC 사이에서 동작
- 인터페이스이며 특정 RDB에 종속되지 않음

---
### 영속성 컨텍스트

- Entity를 영구 저장하는 환경
- EntityManager와 PersistenceContext는 1:1 


#### 영속성 컨텍스트의 장점

- 캐시
  - 1차 캐시 : Db에 들리기 전 1차 캐시에 있는지 먼저 조회, DbTransaction 단위가 끝나면 삭제되기 때문에 Db 전체에서 공유하는 캐시가 아님
- 영속된 엔티티의 동일성 보장 : `==`비교 시 같은 주소를 가진 객체를 참고한 것 처럼 true, 1차 캐싱 때문에 가능
```java
User user1 = em.find(User.class, 1L);
User user2 = em.find(User.class, 1L);
System.out.println(user1 == user2); // 결과 값 true 반환
```
- Entity를 저장할 때 Transaction을 지원하는 쓰기 지연 : `entityManager.persist(user);`를 하게 되면 insert 
  query를 바로 Db에 보내지 않고 차곡 차곡  쓰기 지연 SQL 저장소에 저장하고 `transaction.commit();`하면 저장해뒀던 query가 
  Db에 전송
- 변경 감지 : `transaction.commit();`시 jpa 내부적으로 `flush()`가 호출되고 1차 캐시에 저장돼있는 스냅샷과 Entity를 비교해 바뀐 
  부분이 있다면 update query를 쓰기 지연 SQL 저장소에 저장 하고 Db에 쓰기 지연 SQL저장소에 있는 query를 Db에 전송
  - Entity가 최초로 영속성 컨텍스트에 들어온 시점에 스냅샷을 뜨게된다. (Db에서 읽어오거나 persist()로 영속화하는 등)
    - `flush();` : 영속성 컨텍스트의 상태와 데이터베이스의 상태를 맞추는 것(변경 내용 동기화)
    - 직접 `flush();` 호출시 캐시는 지워지지 않고 쓰기 지연 SQL 저장소에 있는 query만 전송 됌 (영속성 컨텍스트가 지워지는게 아님)
    - 기본적으로 `flush()` 옵션은 `transaction.commit();`과 JPQL 쿼리에서는 자동 호출되고 `em.flush();`는 직접 호출을 
      할 수 있으며 `em.setFlushMode(FlushModeType.COMMIT)`으로 커밋할 때만 자동 호출 되도록 할 수 있다.


---
### Entity 생명주기

- 비영속 (new/transient) : 영속성 컨텍스트와 관계가 아직 없는 새로운 상태, Entity 객체를 새로 생성한 상태
```java
User user = new User();
``` 
- 영속 (managed) : 영속성 컨텍스트에 관리되는 상태 
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("test");
EntityManager em = emf.createEntityManager();
User user = new User();
em.getTransaction().begin();
em.persist(user);
em.getTransaction().commit();
```
- 준영속 (detached) : 영속성 컨텍스트에 저장되었다 분리된 상태, 영속성 컨텍스트가 제공하는 기능 상실
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("test");
EntityManager em = emf.createEntityManager();
User user = new User();
em.getTransaction().begin();
em.persist(user);
em.detach(user);    // 특정 Entity만 준영속 상태로
em.clear();     // EntityManager에 있는 영속성 컨텍스트 전부를 준영속 상태로
em.close();     // 영속성 컨텍스트를 종료
em.getTransaction().commit();
```
- 삭제 (removed) : 삭제된 상태
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("test");
EntityManager em = emf.createEntityManager();
User user = new User();
em.getTransaction().begin();
em.persist(user);
em.remove(user);
em.getTransaction().commit();
```
---
### 객체와 테이블간의 매핑

- `@Entity` : JPA가 해당 class를 관리하도록 하는 어노테이션
  - 해당 클래스의 기본생성자는 필수적으로 있어야 함
  - final class, enum, interface, inner class는 사용할 수 없음
  - 속성 
    - `name` : 매핑할 테이블 이름을 지정할 수 있지만 일반적으로 class와 테이블 이름을 같도록 하기 때문에 기본값을 많이 사용함
    - `catalog` : Db catalog 매핑
    - `schema` : Db schema 매핑 
    - `uniqueConstraints` : DDL 생성 시 유니크 제약 조건 생성
```java
@Entity
public class User {

    @Id
    private Long id;
    private String name;
    
    public Long getId() {
      return id;
    }
    
    public String getName() {
      return name;
    }
}
```

#### schema 자동 생성 

- Application 실행 시점에 Db 방언을 활용해 Db에 맞는 적절한 DDL 자동 생성 이렇게 생성한 DDL은 개발시에만 사용하는 것은 권장한다! 
- 속성
  - `create` : application 실행 시 기존 테이블을 삭제 후 다시 생성
  - `create-drop` : `create`와는 조금 다르게 application 종료 시 테이블 삭제
  - `update` : 추가되는 사항들만 적용됨 삭제가 불가능
  - `validate` : Entity와 테이블이 정상적으로 매핑됐는지 확인 할 수 있음
  - `none` : 해당 기능을 사용하지 않음, 해당 속성을 주석해도 되지만 관례상 사용
  - DDL 생성 기능
    - DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않음
    - ex) : `@Column(length= 10)`
  

    운영 장비에서는 절대 네버 create, create-drop, update를 사용하면 안된다!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

#### 필드와 컬럼 매핑

- `@Id` : 유일한 값을 가지는 기본키인 PK와 필드를 매핑하는 어노테이션 
- `@Column` : 클래스 필드의 이름과 Db에서 사용하는 컬럼을 매핑하는 어노테이션
- `@Enumerated` : enum 타입을 필드로 사용할 경우 Db 컬럼에는 enum 타입이 대부분 없기 때문에 매핑해서 사용할 수 있음 기본값은 
  `EnumType.ORDINAL`이다. 
  - `@Enumerated(EnumType.ORDINAL)` : enum의 순서를 숫자형 타입으로 Db에 저장
  - `@Enumerated(EnumType.STRING)` : enum의 이름을 문자열 타입으로 Db에 저장, **권장하는 방법**
- `@Temporal(TemporalType.?)` : 날짜 타입으로 Java의 DateType은 날짜, 시간이 다 들어있지만 Db에서는 각 각 따로 구분해서 
  쓰기 때문에 맞춰주기 위함. (최신버전에서는 JPA가 필드 속성을 보고 맞춰줌)
  - `DATE` : 날짜 Db date 타입, `TIME` : 시간 Db time 타입, `TIMESTAMP` : 날짜와 시간을 Db timestamp 타입 3가지 
    타입이 있음
- `@Lob` : 큰 컨텐츠를 사용하고 싶을 때 LOB 타입의 컬럼을 사용할 수 있다.
  - 매핑하는 필드가 문자일 경우는 CLOB, 나머지는 BLOB으로 매핑이 됨
- `@Transient` : Db와는 연관없이 메모리에서만 사용하고 싶은 필드일 때 사용

_PK를 직접 할당하려면 `@Id` 어노테이션만 달아주면 되지만 대부분 `@GeneratedValue` 자동 생성을 많이 사용하기 때문에 이에 관한건 아래표를 
참고하자._
```java
@Entity
public class User {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  private Long id;
  @Column(name = "USER_NAME", length = 10)
  private String name;

  public Long getId() {
    return id;
  }

  public String getName() {
    return name;
  }
}
```
| @GeneratedValue 속성 | 설명 | 
| :---: | :---: |
| IDENTITY | Db에 위임 Mysql AUTO_INCREMENT와 같음, 새 객체가 영속성 컨텍스트에 의해 관리되는 시점에 PK값을 확인하기 위해 insert query가 Db에 전송됨 | 
| SEQUENCE | Db sequence object 사용 `@SequenceGenerator()`을 이용해 직접 sequence을 설정할 수 있음, 새 객체가 영속성 컨텍스트에 의해 관리되는 시점에 PK값을 확인하기 위해 sequence call next value로 값을 가져와 객체의 Id를 초기화 한 후 영속성 컨텍스트에 저장 |
| TABLE | 키를 생성하는 테이블을 만들어 Db Sequence 따라하는 방법으로 모든 Db에 적용할 수 있으나 성능이 떨어짐 |
| AUTO | Db 방언에 맞춰 위 3개 속성 중 선택해 자동 생성 |

- `SEQUENCE` 속성
  - `name` : 식별자 생성기 이름으로 필수!
  - `sequenceName` : Db에 등록돼 있는 sequence 이름 기본값은 hibernate_sequence
  - `initialValue` : DDL 생성 시에만 사용됨, 처음 시작하는 수를 지정 기본값은 1
  - `allocationSize` : sequence를 한번 호출 할 때 증가하는 수를 지정(성능 최적화에 사용 call next value를 줄일 수 있음), Db 
    sequence 
    값이 1씩 
    증가하도록 
    설정되있는 경우 반드시 1로 설정해야 함! 기본값은 50
  - `catalog, schema` : Db의 catalog와 schema의 이름

- `Table` 속성
  - `name` : 식별자 생성기 이름 필수 값
  - `table` : 키 생성 테이블 명, 기본 값은 hibernate_sequences
  - `pkColumnName` : 시퀸스 컬럼 명, 기본 값은 sequence_name
  - `valueColumn` : 시퀸스 값 컬럼 명, 기본 값은 next_val
  - `pkColumnValue` : 키로 사용할 값 이름, 기본 값은 Entity의 이름
  - `initialValue` : 초기 값, 마지막으로 생성된 값이 기준, 기본 값은 0
  - `allocationSize` : sequence를 한번 호출 할 때 증가하는 수(성능최적화에 사용), 기본값은 50
  - `catalog, schema` : Db의 catalog와 schema의 이름
  - `uniqueConstraints(DDL)` : unique 제약 조건 설정

_기본 키는 유일하고 절대 변하면 안된다. 하지만 미래에 이 조건을 만족하는 자연키를 찾기는 어려우므로 대체키를 사용하는것이 좋다. 
ex) Long형 + 대체키 + Custom Key 생성 전략_

| @Column 속성 | 설명 | 기본값 | 
| :---: | :---: | :---: |
| name | 객체 필드명과 Db 테이블의 컬럼명이 다를 경우 컬럼명을 넣어서 매핑할 수 있음 | 객체의 필드 이름 |
| insertable, updatable |  해당 컬럼이 insert 되거나 update될 경우 이를 Db에 반영할 것인지 | TRUE |
| nullable(DDL) | 해당 컬럼에 null을 허용할 것인지 설정할 수 있음 | |
| unique(DDL) | 해당 컬럼에 Unique 제약 조건을 걸 수 있으나 이름이 실제 운영에선 사용하기 어렵게 생성됨 (@Table의 uniqueConstraints는 이름까지 설정 가능) | 
| length | 길이 제약 조건을 설정할 수 있고 String 타입에만 사용 | 255 |
| columnDefinition(DDL) | Db 컬럼을 직접 설정 | 필드의 타입과 방언 정보 사용 |
| precision | BicDecimal 타입에서 사용(BigInteger도 가능) 소수점을 포함한 전체 자릿수 (아주 큰 숫자 또는 정밀한 소수를 다룰 때 사용하기 때문에 double, float 타입에는 적용되지 않음) | 19 |
| scale(DDL) | BicDecimal 타입에서 사용(BigInteger도 가능) 소수의 자릿수 (아주 큰 숫자 또는 정밀한 소수를 다룰 때 사용하기 때문에 double, float 타입에는 적용되지 않음) | 2 |