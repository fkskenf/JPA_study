## 영속성 전이 : CASCADE
- 영속성 전이는 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속상태로 만들고 싶을 때 사용이 된다. 
- 대표적인 예로는 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장하는 경우를 들 수 있다.

## 영속성 전이를 이해하기 위한 기본 예제
```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent")
    private List<Child> childList = new ArrayList<>();

    // 연관관계 편의 메서드
    public void addChild(Child child) {
        childList.add(child); // 부모에 자식 셋팅
        child.setParent(this); // 자식에 부모 셋팅
    }

    //.. Getter, Setter 중략
}
```
```java
@Entity
public class Child {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;

    //.. Getter, Setter 중략
}
```
하지만 여기서 Parent를 조작하는 것만으로도 같은 효과를 낼 수 있는 방법이 존재하는데 이 부분이 바로 영속성 전이 CASCADE다
``` java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
private List<Child> childList = new ArrayList<>();
```
- 위와 같이 cascade 속성을 CascadeType.ALL로 설정해준다.

## 영속성 전이와 연관관계 매핑의 상관관계
- 영속성 전이는 연관관계 매핑과는 아무 관련이 없다. 
- 즉, 엔티티를 영속화 할 때 연관된 엔티티도 함께 영속화 한다는 편리함을 제공할 뿐이다.

## CASCADE 종류
1. ALL : 모두 적용
2. PERSIST : 영속
3. REMOVE : 삭제
4. MERGE : 병합
5. REFRESH : REFRESH
6. DETACH : DETACH

- 영속성 전이를 언제 써야 하는가?
> 1. 소유자가 하나인 경우만 영속성 전이를 사용하자. 
> 2. 즉, 단일 엔티티에 완전히 종속적인 경우에는 라이프 사이클이 동일하게 적용되기 때문에 영속성 전이를 사용해도 무방하다.

## 고아 객체
```
