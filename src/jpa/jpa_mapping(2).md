# Jpa 매핑

## 다양한 연관관계 매핑

연관관계 매팽시 고려 사항

- 1:1, 1:N, N:1, N:N (다중성)
- 단방향인지 양방향인지
- 연관관계의 주인 설정

### N : 1 (ManyToOne)

가장 많이 사용하는 연관관계로 단방향에서는 N쪽에서 참조필드를 만들어주면 되고 양방향으로 참조하고 싶을 경우 1쪽에서도 참조 필드를 만들어줘야한다. 연관관계의 주인을
N쪽으로 설정, 1쪽에서는 참조만 하고 영향을 주지 않아야 함

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
    private String name;
}

@Entity
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    private LocalDateTime orderDate;
    @Enumerated(EnumType.STRING)
    private OrderStatus status;

}
```

### 1 : N (OneToMany)

연관관계의 주인을 1쪽으로 설정, `@JoinColumn`을 사용하거나 중간에 조인테이블을 추가하는 방식으로 사용. Member 객체가 바뀔경우 다른 테이블인 Order
테이블도 업데이트 쿼리가 추가적으로 나간다. 연관관계 주인이 아닌쪽에서도 참조하고 싶을
경우 `@ManyToOne` `@JoinColumn(name = "MEMBER_ID", insertable = false, updatable = false)`어노테이션을
참조 필드에 달아줘야 한다. 하지만 권장하지 않는 방법이므로 되도록 N:1을 사용하도록 하자

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    @OneToMany
    @JoinColumn(name = "MEMBER_ID")
    private List<Order> orders = new ArrayList<>();
    private String name;
}

@Entity
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID", insertable = false, updatable = false)
    private Member member;
    private LocalDateTime orderDate;
    @Enumerated(EnumType.STRING)
    private OrderStatus status;

}
```

### 1 : 1 (OneToOne)

테이블이 A, B가 있다면 외래키를 어디에 설정하던 상관이 없고 외래키를 설정한 테이블에 Unique 제약 조건을 걸어야 한다. 양방향으로 설정하려면 참조 객체
쪽에서도 `@OneToOne` mappedBy를 설정해주면 된다.

- 주 테이블에 외래키를 설정하는 경우
    - 매핑이 편리해지고 주 테이블만 조회해도 대상 테이블 데이터를 확인할 수 있으나 값이 없을 경우 주 테이블에 null을 허용해야 한다.
- 대상 테이블에 외래키를 설정하는 경우
    - 주 테이블과 대상 테이블을 1:N 관계로 변경할 때 테이블 구조를 유지할 수 있으나 항상 즉시 로딩됨

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    @OneToOne
    @JoinColumn(name = "ORDER_ID")
    private Order order;
    private String name;
}

@Entity
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;
    @OneToOne(mappedBy = "order")
    private Member member;
    private LocalDateTime orderDate;
    @Enumerated(EnumType.STRING)
    private OrderStatus status;

}
```

### N : N

실무에서는 정말 쓰지 않는것을 권장한다고 한다... 관계형 Db는 정규화된 테이블 2개로 N:N 관계를 표현할 수 없다!
두 테이블을 연결하는 테이블을 만들어 N:1, 1:N관계로 풀어내야 함!
객체의 세상에서는 추가적인 객체 없이 객체 2개만으로 N:N 관계가 가능하다. 중간테이블에 추가적인 정보를 넣기 위해 중간 테이블은 Entity로 승격시켜 추가적인
데이터들을 넣어을 수 있고 Db와 맞춰 한계점을 극복할 수는 있다.

```java

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    @ManyToMany
    @JoinTable(name = "TEAM_COLOR")
    private List<Color> colors = new ArrayList<>();
    private LocalDateTime createDate;

}

@Entity
public class Color {

    @Id
    @GeneratedValue
    @Column(name = "COLOR_ID")
    private Long id;
    private String name;
    @ManyToMany(mappedBy = "colors")
    private List<Team> teams = new ArrayList<>();

}

```

- 한계 극복

```java

@Entity
public class TeamColor {

    @Id
    @GeneratedValue
    private Long id;
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    @ManyToOne
    @JoinColumn(name = "COLOR_ID")
    private Color color;

}

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    @OneToMany(mappedBy = "team")
    private List<TeamColor> TeamColors = new ArrayList<>();
    private LocalDateTime createDate;

}

@Entity
public class Color {

    @Id
    @GeneratedValue
    @Column(name = "COLOR_ID")
    private Long id;
    private String name;
    @OneToMany(mappedBy = "color")
    private List<TeamColors> TeamColors = new ArrayList<>();

}

```

## 상속관계 매핑

관계형 Db는 상속관계가 존재하지 않으나 슈퍼타입 서브타입 관계 모델링 기법이 객체의 상속과 유사해 이를 관계지어 매핑할 수 있다.

- 슈퍼 타입 테이블 밑에 각각의 테이블로 나누는 조인 전략
- 장점
  - 테이블 정규화
  - 외래 키 참조 무결성 제약조건 활용 가능
  - 저장공간 효율화
- 단점
  - 조인 쿼리가 나가 성능이 저하될 수 있음
  - 조회 쿼리가 복잡
  - 데이터 저장시 insert Sql 2번 호출

```java

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Item {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private Long price;

}

@Entity
public class Snack extends Item {

    private String brand;

}

@Entity
public class health extends Item {

    private String type;
  
}
``` 

- 하나의 테이블에 다 넣고 구분할 수 있는 컬럼을 추가하는 단일 테이블 전략
- 장점
  - 조인을 하지 않아 일반적으로 조회가 빠름
  - 조회 쿼리 단순
- 단점
  - null 데이터를 허용해야 함
  - 하나의 테이블에 저장하기 때문에 테이블 볼륨이 무척 커질 수 있고 상황에 따라 성능이 저하될 수 있음

```java

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public abstract class Item {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private Long price;

}

@Entity
public class Snack extends Item {

    private String brand;

}

@Entity
public class health extends Item {

    private String type;
  
}
```

- 슈퍼 타입 객체를 구한하지 않고 서브타입 객체를 테이블로 따로 만들어 하나씩 관리하는 테이블 전략
- Db 설계자와 ORM 전문가 둘 다 권장하지 않는 방법
- 장점
  - 서브 타입을 명확히 구분해 처리할 때 효과적
  - not null 제약조건 사용 가능
- 단점
  - 자식 테이블을 함께 조회할 때 성능이 느림 (자식 전체를 다 확인 함)
  - 자식 테이블을 통합해서 쿼리하기 어렵다.

```java

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private Long price;

}

@Entity
public class Snack extends Item {

    private String brand;

}

@Entity
public class health extends Item {

    private String type;
  
}
```

### 매핑 정보 상속

- 공통적인 매핑 정보가 필요한 경우 사용하면 좋다.(id, name, createDate, updateDate 등등)
- BaseEntity를 상속받아 사용 이 때 BaseEntity는 실제 Entity가 아니고, 조회, 검색이 불가능하다
- 상속받는 자식클래스에 매핑 정보만 제공, 직접 생성해 사용할 일이 없기 때문에 추상 클래스로 만드는 것을 권장함


```java

@MappedSuperclass
public abstract class BaseEntity {

  private String createBy;
  private LocalDateTime createDate;
  private String lastModifiedBy;
  private LocalDateTime lastModifiedDate;

}

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Item extends BaseEntity {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private Long price;

}

@Entity
public class Snack extends Item {

    private String brand;

}

@Entity
public class health extends Item {

    private String type;
  
}
``` 