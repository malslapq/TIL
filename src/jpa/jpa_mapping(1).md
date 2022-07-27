# 연관관계 매핑 기초

Java의 객체들을 Db 테이블에 맞춰서 모델링을 하고 개발을 하다 보면 객체들간의 연관관계는 없고, Db 테이블간의 연관관계만 존재하게 된다. 이런 부분의 문제점은

1. 객체의 필드로 연관 관계 테이블의 PK 값을 직접 들고 있다.
2. 해당 테이블과 연관관계를 맺고 있는 다른 테이블을 가져오기 위해 다시 직접 select 하는 코드를 추가해야 한다.
3. 저장할 때 해당 객체를 그대로 저장하는게 아닌 N : 1 관계 일 때 1쪽의 PK를 가져와 N쪽에 FK로 초기화 시켜줘야 하는 이게 맞나?😕 싶은 부분이 발생한다.

그렇다면 객체 참조와 Db FK를 어떻게 해결하냐?

바로 객체들에게도 **연관관계를 매핑**하는 것으로 해결할 수 있다! 😲

### 단방향 연관관계

- 학교와 학생으로 표현, 갈 수 있는 학교가 단한곳뿐이라고 가정하자.
- N : 1 관계에서 School class는 1, Student class는 N에 해당하고 아래처럼 표현할 수 있다.
- 여기서 중요한 것은 어느 객체가 참조하는 필드를 가지냐인데 N의 입장인 객체에서 가지고 있어야 한다.

```java

@Entity
public class School {

    @Id
    @GeneratedValue
    @Column(name = "SCHOOL_ID")
    private Long Id;
    private String name;

}
```

```java

@Entity
public class Student {

    @Id
    @GeneratedValue
    @Column(name = "STUDENT_ID")
    private Long id;
    private String name;
    // N : 1 , N쪽 객체 필드로 참조
    @ManyToOne
    // 참조하는 테이블의 PK 컬럼을 작성해줘야 한다.
    @JoinColumn(name = "SCHOOL_ID")
    private School school;

}
```

### 양방향 연관관계

단방향 연관관계를 맺는 것 만으로도 사실 Db 관점에서는 양방향 연관관계를 맺는것과 다를게 없지만, 
객체에 입장에서는 참조용 필드가 있는 쪽으로만 참조가 가능하기 때문에 다르다.

위 단방향 연관관계에서 School에서 School의 id를 FK로 가지고 있는 Student를 찾을 수 있을까? Db에서는 가능하지만 객체에서 참조는 불가능하다. 그래서
객체에서도 School에서도 Student를 찾는것이 가능하게 양방향 연관관계를 맺는 것이다.(사실 단반향으로 두개가 서로 참조하고 있는 것)

#### 양방향 매핑 규칙

- 객체의 두 관계 중 하나를 연관관계의 주인으로 지정
- 연관관계의 주인만 외래 키를 관리(등록, 수정)
- 주인이 아닌 쪽은 읽기만 가능
- 주인은 mappedBy 속성 사용 금지
- 주인이 아니면 mappedBy 속성으로 주인을 지정해야함

#### 주의점

- 비즈니스 로직 기준으로 연관관계의 주인을 정하는게 아닌 FK를 가지고 있는 객체를 주인으로!
- N 쪽에서 참조하는 필드가 변경됐을 경우 해당 내용을 1 쪽에도 넣어주는것이 좋다.
    - 이를 위해서 간단하게 연관관계 편의 메소드를 만드는 것이 좋음, 순수 객체 상태를 유지하기 위함!
    - 넣어주지 않을 경우에는 직접 `flush();` 와 `clear();`를 해서 db에 저장하고 가져와야 하는 번거로움이 발생된다. 
    - 연관관계 주인이 아닌 쪽에 변경된 값을 넣었을 경우 실제 Db에는 업데이트 되지 않는다. 메모리상에만 떠있음
- `toString();` 같이 서로를 계속 참조하여 무한루프를 발생시키지 말자!
- 최대한 처음부터 설계할 때 단방향으로 끝낸다는 마인드로..!

```java

@Entity
public class School {

    @Id
    @GeneratedValue
    @Column(name = "SCHOOL_ID")
    private Long Id;
    private String name;
    // 읽기만 가능 수정 불가 
    // mappedBy 속성으로 Student에 있는 필드인 school에 의해 관리
    @OneToMany(mappedBy = "school")
    private List<Student> students = new ArrayList<Student>();
}
```

```java

@Entity
public class Student {

    @Id
    @GeneratedValue
    @Column(name = "STUDENT_ID")
    private Long id;
    private String name;
    @ManyToOne
    @JoinColumn(name = "SCHOOL_ID")
    private School school;

    // 연관관계 편의 메소드
    public void changeSchool(School school) {
        this.school = school;
        school.getStudents.add(this);

    }

}
```