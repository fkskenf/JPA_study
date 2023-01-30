## 지연로딩이 필요한 이유
> Member 엔티티와 Team 엔티티는 N : 1 연관관계 매핑으로 이루어져 있다. <br>
> em.find를 하는 순간 연관관계 매핑이 설정되어있는 Member, Team 엔티티의 데이터를 조인하여 가져온다.
>> 이때, Team 테이블이 아닌 Member의 테이블의 데이터만 필요하다면, <br>
>> 두 테이블이 조회되는 것은 성능 상 매우 비효율적이다. JPA는 이런 상황을 대비하여 지연 로딩이라는 기능을 제공한다.


##  프록시(Proxy)
- EntityManger의 getReference() 메서드를 호출하면 해당 엔티티를 바로 조회하지 않고, 실제 사용하는 시점에 조회해올 수 있다. <br>
- 이러한 지연로딩을 사용하기 위해 프록시 객체를 사용한다.
```java
// ex 01. 일반적인 조회
// MemberTest findMember = em.find(MemberTest.class, member.getId());
// System.out.println("findMember = " + findMember.getId());
// System.out.println("findMember = " + findMember.getUsername());

// ex 02. 프록시를 통한 조회
MemberTest findMember = em.getReference(MemberTest.class, member.getId()); // em.getReference => 프록시
System.out.println("findMember = " + findMember.getId()); // id는 위에서 넣어주어서 상관 없음
System.out.println("findMember = " + findMember.getUsername()); // username은 DB에 있는 데이터
```
1. em.getReference를 하는 경우에는 쿼리를 날리지 않는다.
2. 해당 값이 실제로 사용이 되는 시점에 쿼리가 날라간다.
> 실제 날라가는 시점 -> getUsername()을 호출하는 경우

- 하이버네이트가 만든 가짜 클래스(proxy class)

## 프록시 특징
1. 특징(1)
> 1. 실제 클래스를 상속받아 만들어짐
>> 하이버네이트 내부적으로 상속을 수행한다
> 2. 실제 클래스와 겉 모양이 같다
> 3. 사용하는 입장에서의 관점
>> 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 됨(이론상)

2. 특징(2)
> 1. 프록시 객체는 실제 객체의 참조(target)를 보관한다
> 2. 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출 

### 프록시 식별자
1. 프록시 객체는 target 변수만 가지고 있는것이 아니라, 전달받은 식별자 값도 같이 저장한다. 
2. 그러므로 아래와 같이 식별자 값(pk)만 조회할 경우 직접적인 데이터베이스 조회가 일어나지 않는다. 
> (이러한 부분이 getUserName()의 데이터베이스 조회가 일어난 이유와 연관이 있음)

## 프록시 객체 특징 정리
1. 프록시 초기화 관련
> 프록시 객체는 처음 사용할 때 최초로 한번 초기화 된다. <br>
> 또한 프록시 객체를 초기화 하는 경우, 프록시 객체가 실제 엔티티로 바뀌는 것이 아닌 프록시 객체를 통해 실제 엔티티에 접근하는 방식으로 사용이 된다

2. 프록시 타입 체크시 주의점
> 프록시 객체는 원본 엔티티를 상속 받는데, 타입 체크시 ‘==’ 이 아닌 instance of 메서드를 사용을 지향해

3. 프록시 엔티티 반환 여부
> 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출 하여도 실제 엔티티를 반환한다. <br>
> 중요한 부분은 하나의 트랜잭션 단위 안에서 '=='을 사용하는 경우 영속성 컨텍스트에 엔티티가 존재 한다면 True가 반환 되어야 한다

4. 프록시 준영속 상태인 경우
> 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태인 경우, 프록시를 초기화하면 문제발생 위 부분은 실무에서 자주 발생하는 예외로 숙지하고 넘어가야 한다

## 프록시 확인
1. 프록시 인스턴스의 초기화 여부 확인
>  EntityMangerFactory.getPersistenceUnitUtil().isLoaded() 메서드를 호출하여 해당 프록시가 초기화 되었는지 확인이 가능하다.

2. 프록시 클래스 확인 방법
> entity.getClass().getName();

3. 프록시 강제 초기화
> JPA 표준은 강제 초기화가 존재하지 않는다. 그러므로 강제 호출 member.getName()을 통해 강제 초기화를 해야한다.



