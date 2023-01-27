# JPA_study

- Reference : https://ym1085.github.io/categories/jpa

1. JPA는 동일한 트랜잭션에서 조회한 엔티티는 같음을 보장해준다.
```java 
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2 //같다
```

2. SQL의 경우에는 두번 쿼리가 날라가겠지만, JPA는 캐시를 통해 한번 날라간다
```java
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId); // SQL 날리고
Member member2 = jpa.find(Member.class, memberId); // 캐시에서 가져옴

println(m1==m2); //true
```

3. 지연 로딩: (중요)
> 1. 특정 객체가 실제로 사용이 될 때 로딩
> 2. 지연 로딩은 쿼리가 두번 날라간다는 단점이 있다
```java 
// 지연로딩
Member member = memberDao.find(memberId);
Team team = member.getTeam(); // team값이 실제 필요할 경우 해당 쿼리 실행
String teamName = team.getName();

SELECT * FROM MEMBER // 먼저 실행
SELECT * FROM TEAM // team값이 실제 필요할 경우 해당 쿼리 실행
```

4. 즉시 로딩: (중요)
> 1. JOIN SQL로 한 번에 연관된 객체까지 미리 조회
> 2. JPA 옵션을 설정 후 즉시 로딩 설정 가능
> 3. Member랑 Team 조회 시 한 번에 가져오도록 설정 가능
```java
// 즉시로딩
Member member = memberDao.find(memberId);
Team team = member.getTeam();
String teamName = team.getName();

SELECT M.*, T.*
FROM MEMBER JOIN TEAM ....
```

5. 영속성 컨텍스트의 이점
> 1. 1차 캐시
> 2. 동일성 보장
> 3. 트랜잭션을 지원하는 쓰기 지연 ( transaction write-behind )
> 4. 변경 감지 ( Dirty Checking )
> 5. 지연 로딩 ( Lazy Loading )

6. 연관 관계 매핑 
```java
Team team = new Team();
team.setName("TeamA") //팀명 셋팅
em.persist(team);

MemberTest member = new MemberTest();
member.setUsername("member1");
member.setTeam(team.getTeamId()); //변경 전
```

7. 연관 관계 매핑 후
```java
Team team = new Team();
team.setName("TeamA") //팀명 셋팅
em.persist(team);

MemberTest member = new MemberTest();
member.setUsername("member1");
member.setTeam(team); //변경 후 -> 참조 변수 넣기
```
> 1. JPA는 기본적으로 영속성 컨텍스트의 특성을 가짐
> 2. 외래키(FK)를 넣어주는 것이 아닌 참조 변수(team)를 넣어준다
> 3. @ManyToOne, @JoinColumn(name = “TEAM_ID”)로 매핑을 해둔 상황
> 4. Member, Team 테이블에 자동으로 PK(기본키 값), FK(외래키 값)가 셋팅 된다

8. 양방향 매핑 규칙
> 1. 객체의 두 관계 중 하나를 연관관계의 주인으로 지정 (외래 키가 있는 곳을 주인 지정)
> 2. 연관관계의 주인만이 외래 키를 관리 - 등록, 수정
> 3. 주인이 아닌쪽은 읽기만 가능
> 4. 주인은 mappedBy 속성 사용을 하지 않는다
> 5. 주인이 아니면 mappedBy 속성으로 주인 지정

9. 양방향 매핑시 대처 방법
```java
@Entity
@Table(name = "TEAM")
public class Team {

  @Id
  @GeneratedValue
  @Column(name = "TEAM_ID")
  private Long id;
  private String name;

  @OneToMany(mappedBy = "team")
  private List<MemberTest> members = new ArrayList<MemberTest>();

  public List<MemberTest> getMembers() {
    return members;
  }
}
```
> 연관관계 편의 메서드를 생성한다.
```java
@Entity
@Table(name = "MEMBER")
public class Member {

  @Id
  @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  @Column(name = "USERNAME")
  private String username;

  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private TeamTest team;

  public void setTeam(Team team) {  // 편의 메서드
      this.team = team;
      team.getMember().add(this);
  }
}
```

9. 양방향 매핑시 연관관계의 주인에 값을 입력
```java
TeamTest team = new TeamTest();
team.setName("TeamA");
em.persist(team);

MemberTest member = new MemberTest();
member.setUsername("member1");
member.setTeam(team) // 연관관계의 주인 쪽에 값을 넣어준다
em.persist(member);
```
> 1. 단방향 매핑만으로도 이미 연관관계 매핑은 완료가 된 것이다.
> 2. 처음 설계 시 단방향 매핑으로 설계를 마무리 해야한다.
> 3. 양방향이 필요한 이유는 반대 방향으로 객체그래프를 탐색하기 위해서이다.
> 4. JPQL에서는 역방향으로 탐색할 일이 많다.
> 5. 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 된다.

10.  (N:1) 연관관게 매핑 예시 -> 주로 사용
```sql
SELECT
  A.member_no,
  B.member_name,
  B.member_address
FROM
  TEAM A
    JOIN
  MEMBER B
    ON
  A.member_no = B.member_no
```
```java
@Entity
@Table(name = "MEMBER")
public class Member {
    //..중략

    @ManyToOne
    @JoinColumn(name = "TEAM_ID") // pk
    private Team team

    //..중략
}
```
```java
@Entity
@Table(name = "TEAM")
public class Team {
    //..중략

    @OneToMany(mappedBy = "team")
    private List<MemberTest> members = new ArrayList<MemberTest>();

    //..중략
}
```
11. (1:N) 연관관계 매핑시 문제점 
> 1. 팀(1) 엔티티를 수정하였는데 맴버(N) 테이블에 업데이트 쿼리가 나가는 현상 발생.
> 2. 실무에서는 테이블이 수십개가 엮여서 돌아가는 상황, 위와 같은 상황은 운영을 힘들게 한다.
> -  해결 방안Permalink
>> 1. 객체지향적으로 조금 손해를 볼 수 있지만, 다대일 단방향, 양방향을 사용한다.
>> 2. 즉, 먼저 학습한 다대일 단방향 관계로 매핑하고, 필요한 경우 양방향 매핑을 통해 해결한다.
>> 3. 객체 입장에서 보면, 반대방향으로 참조할 필요가 없는데 관계를 하나 만드는 것이지만, DB의 입장으로 설계의
>> 4. 방향을 조금 더 맞춰서 운영상 유지보수하기 쉬운 쪽으로 선택할 수 있다.

```java
//수정 전: 이런식으로 매핑을 하면 연관관계의 주인이 2개가 된다.
@ManyToOne
@JoinColumn(name = "TEAM_ID")
private Team team;

//수정 후: 읽기 전용으로 변경하여, 양방향 매핑을 한 효과를 만들어낸다.
@ManyToOne
@JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
private Team team;
```
> - 일대다 단방향 매핑의 단점
>> 1. 엔티티가 관리하는 외래키가 다른 테이블에 있음
>> 2. 연관관계 관리를 위해 추가로 UPDATE SQL 실행. (성능 문제)
>> 3. 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자.
>> 4. 객체간의 참조를 하나 더 넣는 한이 있더라도 다대일 사용.

12. (1:1) 연관관계 매핑시
> 1. 주 테이블에 외래키 양방향 정리
>> 1. 다대일 양방향 매핑처럼 외래키가 있는곳이 연관관계의 주인
>> 2. 반대편은 mappedBy 적용을 해야한다.
> 2. DB관점에서 많이 조회되는 것을 기준으로 외래크를 적용해야 한번의 쿼리로 쉽게 조회 가능 
> 3. 테이블에 외래키
>> 1. 주 객체가 대상 객체의 참조를 가지는것처럼, 주 테이블에 외래키를 두고 대상 테이블을 찾는다.
>> 2. 객체지향 개발자가 선호한다
>> 3. JPA 매핑 편리
>>> 1. 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 있는 확인 가능
>>> 2. 단점: 값이 없으면 외래키에 NULL 허용
> 4. 대상 테이블에 외래키
>> 1. 대상 테이블에 외래키가 존재
>> 2. 전통적인 DB 개발자가 선호한다
>>> 1. 장점: 주 테이블, 대상 테이블을 일대일에서 일대다로 변경 시 테이블 구조 유지가능
>>> 2. 단점: 프록시 기능의 한계로 지연로딩 설정해도 항상 즉시 로딩됨

13. (N:N) 연관관계 매핑 
> 1. 다대다 관계는 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어야한다.
> 2. @ManyToMany
>> 1.  즉, 원하는 정보가 아닌 추가적인 정보가 들어갈 수 있다.
>> 2. 쿼리가 이상하게 나가는 현상이 발생할 수 있다.
> 3. @ManyToMany -> @OneToMany, @ManyToOne으로 변경
>> - 중간에 연결테이블을 활용하여 다대다를 일대다/다대일로 변경하여 사용한다.
```java 
@Entity
public class MemberProduct {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name="PRODUCT_ID")
    private Product product;

    // 추가적으로 컬럼을 설정할 수 있다.
    private int count;
    private int price;

    private LocalDateTime orderDateTime;
}
```

14. 슈퍼타입과 서브타입
> - 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법은 아래와 같다.
> 1. 각각 테이블로 변환
>> 조인 전략
> 2. 통합 테이블로 변환
>> 단일 테이블 전략
> 3. 서브타입 테이블로 변환
>> 구현 클래스마다 테이블 전략

15. @JoinColumn
> 외래키 매핑 시 사용이 되는 어노테이션.
```java
@Entity
public class Team {

    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "team") // Member Entity의 team 객체를 바라본다
    private List<Member> members = new ArrayList<Member>();
    //..중략
}
```
```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID") // 외래키로 매핑될 컬럼을 지정 (TEAM : PK)
    private Team team;
}
```

16. 상속관계매핑
> - 각각의 엔티티를 상속 관계로 연결한 후 실행하면, JPA에서는 기본적으로 단일 테이블 전략을 사용한다.
> - 해결 : 단일 테이블 전략 -> 조인 전략으로 변경
>> 기존 부모(상위)클래스에 @Inheritance 어노테이션을 추가한다.
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED) // 상속 관계 매핑 추가
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private int price;
    //..중략 Getter Setter
}
```
> - 부모 - 자식 관계에서 어떤 자식의 타입인지 구분하기 위해 사용.
>> @DiscriminatorColumn 어노테이션을 추가
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn // 해당 어노테이션 추가
// @DiscriminatorColumn(name = "DIS_TYPE") 이름을 지정할 수 있다
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private int price;
    //..중략 Getter Setter
}
```
```SQL
Hibernate:
/* insert com.hello.jpasample.Movie
    */ insert
    into
        Item
        (name, price, DTYPE, id)
    values
        (?, ?, 'Movie', ?)
```

