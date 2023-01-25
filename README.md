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
