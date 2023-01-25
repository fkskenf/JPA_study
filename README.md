# Jpa_study

* JPA는 동일한 트랜잭션에서 조회한 엔티티는 같음을 보장해준다.
```java 
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2 //같다
```

* SQL의 경우에는 두번 쿼리가 날라가겠지만, JPA는 캐시를 통해 한번 날라간다
```java
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId); // SQL 날리고
Member member2 = jpa.find(Member.class, memberId); // 캐시에서 가져옴

println(m1==m2); //true
```
