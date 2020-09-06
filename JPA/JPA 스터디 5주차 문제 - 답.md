## JPA 스터디 5주차 문제 - 답

> 승현

1. 아래와 같은 JPA named 쿼리가 있을 때, 스프링 데이터 JPA에서 named 쿼리를 호출하는 코드를 작성

```java
@Entity
@NamedQuery(
  name="Member.findByUsername",
  query="select m from member m where m.username =: username"
)
public class Member {
  ...
}
```

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  // 코드 작성
    List<Member> findByUserName(@Param("username") String username);
}
```



2. 스프링 데이터 JPA를 사용하면서 사용자 정의 인터페이스를 구현할 때 구현한 클래스 이름은 어떻게 작성해야 할까?

   - 스프링 데이터 JPA를 사용하면서 사용자 정의 인터페이스를 구현한 클래스를 작성할 때에는 클래스 이름을 짓는 규칙이 있다. 
   - 리포지토리 인터페이스 이름 + Impl로 지어야 한다.
   -  이렇게 하면 스프링 데이터 JPA가 사용자 정의 구현 클래스로 인식한다.

   ```java
   public class MemberRepositoryImpl implements MemberRepositoryCustom {
     @Override
     public List<Member> findMemberCustom() {
       ...
     }
   }
   ```

   

3. 트랜잭션 범위의 영속성 컨텍스트 전략은 트랙잭션이 같으면 항상 같은 영속성 컨텍스트를 사용한다. (**O**/X)
   - 트랜잭션 범위의 영속성 컨텍스트 전략은 다양한 위치에서 엔티티 매니저를 주입받아 사용해도 트랜잭션이 같으면 항상 같은 영속성 컨텍스트를 사용한다. 
   - 트랜잭션이 다르면 다른 영속성 컨텍스트를 사용한다.



4. 요청 당 트랜잭션 방식의 OSIV의 문제점은 무엇일까?
   - 요청 당 트랜잭션 방식의 OSIV는 컨트롤러나 뷰 같은 프레젠데이션 계층이 엔티티를 변경할 수 있다는 문제점이 있다. 
   - 이를 막기 위해 프레젠테이션 계층에서 엔티티를 수정하지 못하도록 
     - 엔티티를 읽기 전용 인터페이스로 제공하거나 
     - 엔티티 래핑을 하거나 
     - DTO만 반환하도록 할 수 있다.



5. 스프링 OSIV를 사용할 때 프레젠테이션 계층에서 `em.flush()`를 호출해서 강제로 플러시하면 플러시가 일어날 수 있다. (O/**X**)
   - 스프링 OSIV를 사용할 때 프레젠테이션 계층에서 `em.flush()`를 호출해서 강제로 플러시해도 트랜잭션 범위밖이므로 데이터를 수정할 수 없다는 예외를 만난다. 
   - 스프링이 제공하는 OSIV 서블릿 필터나 OSIV 스프링 인터셉터는 요청이 끝나면 플러시를 호출하지 않고 `em.close()`로 영속성 컨텍스트만 종료해 버리므로 플러시가 일어나지 않는다.



6. 다음 테스트의 결과와 그러한 결과가 나타나는 이유는?

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:appConfig.xml")
public class MemberServiceTest {
  
  @Autowired MemberService memberService;
  @Autowired MemberRepository memberRepository;
  
  @Test
  public void 회원가입() throws Exception {
    
    //Given
    Member member = new Member("Kim");
    
    //When
    Long saveId = memberService.join(member);
    
    //Then
    Member findMember = memberRepository.findOne(saveId);
    
    assertTrue(member == findMember);
  }
}

@Transactional
public class MemberService {
  
  @Autowired MemberRepository memberRepository;
  
  public Long join(Member member) {
    ...
    memberRepository.save(member);
    return member.getId();
  }
}

@Repository
@Transactional
public class MemberRepository {
  
  @PersistenceContext
  EntityManager em;
  
  public void save(Member member) {
    em.persist(member);
  }
  
  public Member findOne(Long id) {
    return em.find(Member.class, id);
  }
}
```

- 테스트는 실패한다. 
- 코드에서 `memberService.join(member)`를 호출해서 회원 가입을 시도하면 서비스 계층에서 트랜잭션이 시작되고 영속성 컨텍스트가 하나 만들어진다. `memberRepository`에서 `em.persist()`를 호출해서 엔티티를 영속화하고 서비스 계층이 끝날 때 트랜잭션이 커밋되면서 영속성 컨텍스트가 플러시된다. 따라서 `member` 엔티티 인스턴스는 준영속 상태가 된다. 
- 테스트 코드에서 `memberRepository.findOne(saveId)`를 호출해서 저장한 엔티티를 조회하면 리포지토리 계층에서 새로운 트랜잭션이 시작되면서 새로운 영속성 컨텍스트가 생성되고, 이 컨텍스트에는 찾는 회원이 존재하지 않으므로 데이터베이스에서 회원을 찾아온다. 데이터베이스에서 조회된 회원 엔티티를 영속성 컨텍스트에 보관하고 반환하고, 메소드가 끝나면서 트랜잭션이 종료되고 새로운 영속성 컨텍스트도 종료된다. 
- `member`와 `findMember`는 각각 다른 영속성 컨텍스트에서 관리되었기 때문에 다른 인스턴스이므로 동일성 비교가 실패한다.



7. 프록시의 멤버변수에 직접 접근하지 않고 접근자 메소드를 쓰는 이유는 무엇일까?

   - 프록시는 실제 데이터를 가지고 있지 않으므로 프록시의 멤버변수에 직접 접근하면 아무 값도 조회할 수 없다. 
   - 특히, 일반적인 상황에서는 프록시의 멤버변수에 직접 접근하는 문제가 발생하지 않지만 `equals()` 메소드는 자신을 비교하기 때문에 `private` 멤버변수에도 접근할 수 있다.
   - 따라서, 프록시의 데이터를 조회할 때는 접근자를 사용해야 한다.

   

8. JPA 낙관적 락의 OPTIMISTIC 옵션을 적용하면 어떤 문제들이 방지될까?

   - JPA 낙관적 락의 OPTIMISTIC 옵션을 적용하면 엔티티를 조회만 해도 버전을 체크한다. 
   - 한 번 조회한 엔티티는 트랜잭션을 종료할 때까지 다른 트랜잭션에서 변경하지 않음을 보장한다. 이로 인해 DIRTY READ와 NON-REPEATABLE READ를 방지한다.

   

9. 엔티티 캐시를 사용해서 엔티티를 캐시하면 (**엔티티 정보**)을/를 모두 캐시하지만 쿼리 캐시와 컬렉션 캐시는 (**결과 집합의 식별자 값**)만 캐시한다.





> 선재

1. 준영속 상태의 지연 로딩 문제를 해결하는 방법 중 뷰가 필요한 엔티티를 미리 로딩해두는 방법 세 가지를 명칭만 말씀해 주세요.
   - 글로벌 페치 전략 수정
   - JPQL 페치 조인
   - 강제로 초기화
     

2. N+1 문제가 무엇인지, 어떻게 해결하는지 설명해 주세요.

   - 연관된 엔티티를 로딩하는 전략이 즉시 로딩이면 데이터베이스에 JOIN 쿼리를 사용해 한 번에 연관된 엔티티까지 조회한다.

   - 문제는 JPQL을 사용할 때 발생하는데, 다음과 같은 JPQL로 조회할 시실행된 SQL은 다음과 같다.

   ```sql
   // JPQL
   List<Order> orders = em.createQuery("select o from Order o", Order.class).getResultList();
   // 연관된 모든 엔티티를 조회한다.
   ```

   ```sql
   // JPQL로 실행된 SQL
   select * from Order 
   select * from Member where id = ? // EAGER로 실행된 SQL
   select * from Member where id = ? // EAGER로 실행된 SQL
   select * from Member where id = ? // EAGER로 실행된 SQL
   select * from Member where id = ? // EAGER로 실행된 SQL
   ...
   ```

   - JPA가 JPQL을 분석해서 SQL을 생성할 때는 글로벌 페치 전략을 참고하지 않고 오직 JPQL 자체만 사용한다.

   

   **해결방안**

   - JPQL 페치 조인으로 해결할 수 있다.

     ```sql
     select o from Order o join fetch o.member
     select m from Member m join fetch m.orders
     ```

   - 하이버네이트 @BatchSize

     - 하이버네이트가 제공하는 @BatchSize 어노테이션을 사용하면 연관된 엔티티를 조회할 때 지정한 size 만큼 SQL의 IN 절을 사용해서 조회한다.
     - 만약 조회한 회원이 10명인데 size=5로 지정하면 2번의 SQL만 추가로 실행한다.

   - 하이버네이트 @Fetch(FetchMode.SUBSELECT) 

     - 연관된 데이터를 조회할 때 서브 쿼리를 사용해서 해결한다.

     ```java
     @org.hibernate.annotations.Fetch(FetchMode.SUBSELECT)
     ```

     

3. OSIV 초창기는 어떤 방식으로 동작되었으며, 어떤 문제점이 있었고 어떻게 해결하는가?

   - 초기에는 요청 당 트랜잭션 방식이다. 

     - 클라이언트의 요청이 들어오자마자 서블릿 필터나 스프링 인터셉터에서 트랜잭션을 시작하고 요청이 끝날 때 트랜잭션도 끝내는 것이다.
     - FACADE 계층이 없이도 뷰에 독립적인 서비스 계층을 유지할 수 있다.
     - 문제점은 컨트롤러나 뷰 같은 프레젠테이션 계층이 엔티티를 변경할 수 있다는 점이다.

   - 해결법은 다음과 같다.

     * 엔티티를 읽기 전용 인터페이스로 제공
     * 엔티티 레핑
     * DTO로 반환

     

4. 네 가지 트랜잭션 격리 수준을 설명하고, 각각 어떤 문제 현상이 발생할 수 있는지 설명하시오.

   - READ UNCOMMITED
     * 트랜잭션 1이 데이터를 수정하고 있는데 커밋하지 않아도 트랜잭션 2가 수정 중인 데이터를 조회할 수 있다. Dirty Read 라고 한다. 트랜잭션 2가 Dirty Read 한 데이터를 사용하는데 트랜잭션 1을 롤백하면 데이터 정합성에 문제를 일으킬 수 있다.
   - READ COMMITTED
     * NON-REPETABLE READ가 발생한다. 트랜잭션 1이 회원 A를 조회 중인데 갑자기 트랜잭션 2가 회원 A를 수정하고 커밋하면 트랜잭션 1이 다시 회원 A를 조회했을 때 수정된 데이터가 조회된다. 반복해서 같은 데이터를 읽을 수 없는 상태를 NON-REPEATABLE READ 라 한다.
   - REPEATABLE READ
     * PHANTOM READ 가 발생한다. 반복 조회 시 결과 집합이 달라지는 것을 뜻한다.
   - SERIALIZABLE
     * PHANTOM READ는 발생하지 않지만, 동시성 처리 성능이 급격히 떨어질 수 있다.

애플리케이션 대부분은 동시성 처리가 중요하므로 데이터베이스들은 보통 READ COMMITTED 격리 수준을 기본으로 사용한다. 일부 중요한 비즈니스 로직에 더 높은 격리 수준이 필요하면 데이터베이스 트랜잭션이 제공하는 잠금 기능을 사용하면 된다.