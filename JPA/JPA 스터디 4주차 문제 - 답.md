## JPA 스터디 4주차 문제 - 답

> 다롬

1. 아래 SQL문은 JPQL, Criteria, QueryDSL 의 결과로 나타난 SQL문이다. 아래 SQL문을 보고 JPQL, Criteria, QueryDSL로 각각 변환하시오.

### SQL

```sql
// 팀의 평균나이가 20살 이상인 팀의 이름과, 평균 나이를 평균나이 내림차순으로 정렬하고, 평균 나이가 같으면 팀 이름을 기준으로 오름차순 정렬한다.

SELECT t.name, AVG(m.age)
FROM MEMBER m JOIN m.team t ON m.team_id = t.id
GROUP BY t.name
HAVING AVG(m.age) >= 20
ORDER BY m.age DESC, t.name;
```



### jpql

```java
String jpql = "SELECT t.name, AVG(m.age) FROM Member m JOIN m.team t" 
    + "GROUT BY t.name HAVING AVG(m.age) >= 20" 
    + "ORDER BY m.age DESC, t.name" // -1
    
List<Object[]> resultList = em.createQuery(query).getResultList();

for(Object[] result : resultList){
    ...
}
```

- JOIN FETCH - 페치 조인할 경우 조인 대상에 별칭을 줄수 없어 위와 같이 t를 쓸 수 없다.

```java
String jpql = "SELECT t.name, AVG(m.age) FROM Member m JOIN FETCH m.team" 
    + "GROUT BY m.team.name HAVING AVG(m.age) >= 20" 
    + "ORDER BY m.age DESC, m.team.name" // -1
    
List<Object[]> resultList = em.createQuery(query).getResultList();

for(Object[] result : resultList){
    ...
}
```



### Criteria

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
EntityManager em = emf.createEntityManager();

CriteriaBuilder cb = em.getCriteriaBuilder(); // Criteria 쿼리 빌더

CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
Root<Member> m = cq.from(Member.class); // 조회 시작점
m.fetch("team", JoinTpe.LEFT); // 페치 조인 : -2

// 정렬 조건 정의 -3
javax.persistence.criteria.Order ageDesc = cb.desc(cb.avg(m.get("age")))
javax.persistence.criteria.Order teamNameAsc = cb.asc(m.get("team").get("name"));

// 쿼리문 작성 -4
cq.multiselect(m.get("team").get("name"), cb.avg(m.get("age")))
  .groupBy(m.get("team").get("name"))
  .having(cb.ge(cb.avg(m.get("age")), 20))
  .orderBy(ageDesc, teamNameAsc);

// Object[]
List<Object[]> = resultList = em.createQuery(cq).getResultList();
for(Object[] result : resultList){
    ...
}
```

```java
CriteriaBuilder cb = em.getCriteriaBuilder(); // Criteria 쿼리 빌더

CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
Root<Member> m = cq.from(Member.class); // 조회 시작점
Join<Member, Team> t = m.join("team", JoinType.INNER); // 내부 조인 - 별칭 설정 : -2

// 정렬 조건 정의 -3
javax.persistence.criteria.Order ageDesc = cb.desc(cb.avg(m.get("age")))
javax.persistence.criteria.Order teamNameAsc = cb.asc(t.get("name"));

// 쿼리문 작성 -4
cq.multiselect(t.get("name").alias("name"), cb.avg(m.get("age")).alias("ageAvg"))
  .groupBy(t.get("name"))
  .having(cb.ge(cb.avg(m.get("age")), 20))
  .orderBy(ageDesc, teamNameAsc);

// 튜플
TypedQuery<Tuple> query = em.createQuery(cq);
List<Tuple> = resultList = query.getResultList();
for(Tuple tuple : resultList){
    String name = tuple.get("name", String.class);
    Double ageAcg = tuple.get("ageAvg", Double.class);
    ...
}
```



### QueryDSL

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
EntityManager em = emf.createEntityManager();

JPAQuery query = new JPAQuery(em); 
QMember member = QMember.member;
QTeam team = QTeam.team;

// 쿼리문 작성 -5
List<Tuple> = result = query.from(member)
                            .join(member.team, team)
                            .groutpBy(team.name)
                            .having(member.age.avg().gt(20))
                            .list(team.name, member.age.avg());

for(Tuple tuple : result){
    tuple.get(member.age.avg())
    ...
}
```





> 경훈

## O.X 문제
1) 벌크 연산을 사용할 때 벌크 연산은 영속성 컨텍스트를 반영하여 결과 값을 반환한다. 

- X 바로 DB에 쿼리 날림

2) JDBC는 이름 기준 파라미터 바인딩을 제공하지만 JDQL은 위치 기준 파라미터 바인딩도 지원한다.

- X. JDBC는 위치 기준 파라미터만, JPQL은 이름 기준, 위치 기준 파라미터 바인딩 모두 지원.

3) 튜플은 튜플의 검색 키로 사용할 튜플 전용 별칭을 필수로 할당해야한다.

- O

 

### 1. 페치 조인의 한계를 쓰시오.

- **페치 조인 대상에는 별칭을 줄 수 없다**. SELECT, WHERE 절, 서브 쿼리에 페치 조인 대상을 사용할 수 없다.
  - JPA 표준은 아니지만 하이버네이트를 포함한 몇몇 구현체들은 페치 조인에 별칭을 지원한다. 잘못 사용하면 연관된 데이터 수가 달라져서 데이터 무결성이 깨질 수 있으므로 조심해서 사용해야 한다. 특히 2차 캐시와 함께 사용할 때 조심해야 하는데, 연관된 데이터 수가 달라진 상태에서 2차 캐시에 저장되면 다른 곳에서 조회할 때도 연관된 데이터 수가 달라지는 문제가 발생할 수 있다.
- **둘 이상의 컬렉션을 페치할 수 없다.**
  -  컬렉션 * 컬렉션의 **카테시안 곱**이 만들어지므로 주의해야 한다. 
  - 하이버네이트를 사용하면 MultipleGabFetchException예외가 발생한다.
- 컬렉션을 페치 조인하면 **페이징 API**(setFirstResult, setMaxResults)**를 사용할 수 없다**.
  - 컬렉션(일대다)이 아닌 단일 값 연관필드(일대일, 다대일)들은 페치 조인을 사용해도 페이징 API를 사용할 수 있다.
  - 하이버네이트에서 컬렉션을 페치 조인하고 페이징 API를 사용하면 경고 로그를 남기면서 메모리에 페이징 처리를 한다. 데이터가 많으면 **성능 이슈와 메모리 초과 예외**가 발생할 수 있어서 위험하다.

### 2. JPQL 쿼리는 2가지로 나뉘게 되는데 2가지의 차이점을 쓰시오

- 동적쿼리 : em.createQuery("select ...") 처럼 JPQL을 문자로 완성해서 직접 넘기는 것을 동적쿼리라 한다. 런타임에 특정 조건에 따라 JPQL을 동적으로 구성할 수 있다.
- 정적 쿼리 : 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용할 수 있는데 이것을 Named 쿼리라 한다. 이 쿼리는 한번 정의하면 변경할 수 없는 정적인 쿼리다.

### 3. 데이터를 조회할때 find() 와 JPQL 의 차이점을 쓰시오

- em.find()
  - 엔티티를 영속성 컨텍스트에서 먼저 찾고 없으면 DB 조회
- JPQL
  - 영속성 컨텍스트를 거치지 않고 바로 DB에서 조회





> 선재

### 1. 다음 중 JPQL 쿼리가 올바른지 알려주세요.

1. select m, t from Member m join fetch m.team t ( O / **X** )
2. Select m.age from Member m join Team t ( O / **X** )
   - Team → m.team
3. Select t.name, COUNT(m.age), SUM(m.age), AVG(m.age), MAX(m.age), MIN(m.age) from Member m LEFT JOIN m.team t GROUP by t.name HAVING AVG(m.age) >= 10 ( **O** / X )



### 2. 컬렉션 페치 조인할 때 주의사항, 단점을 이야기해주세요.

- **Distinct**를 추가해주지 않으면 객체가 연관된 **객체의 수 만큼 생성**된다.
- **Distinct**를 추가해주면, **애플리케이션에서 한 번 더 중복을 제거**해준다.

+

- 페치 조인 대상에는 별칭을 줄 수 없다. SELECT, WHERE 절, 서브 쿼리에 페치 조인 대상을 사용할 수 없다.
- 둘 이상의 컬렉션을 페치할 수 없다.
- 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.



### 3. 페치 조인과 일반 조인의 차이점을 이야기해주세요.

- JPQL은 결과를 반활할 때 연관관계까지 고려하지 않는다. 단지 SELECT 절에 지정한 엔티티만 조회할 뿐이다.
  - **지연 로딩**으로 설정했을 때 **프록시**나 아직 **초기화하지 않은 컬렉션 래퍼**를 반환한다.
  - **즉시 로딩**으로 설정했을 때 **쿼리**를 한 번 더 실행한다.
- 반면에 페치 조인을 사용하면 연관된 엔티티도 함께 조회한다.