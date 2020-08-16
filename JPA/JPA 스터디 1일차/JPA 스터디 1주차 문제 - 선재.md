# JPA 스터디 1일차 정리
> made by [다람쥐](https://github.com/kor-Chipmunk)
>
> > 1~3단원


## 문제점

### 1. H2 Database 설치
Embedded 에서 접속 후 다시 Server 모드로 실행해야 하는데, 접속이 되지 않음.
심지어 Embedded 에서도 접속이 되지 않을 때도 있음

Embedded 에서 `testdb` 으로 변경해서 접속. Embedded 접속해도 Server 모드 접속이 안됨.. 이런저런 글을 찾아보니 `test.mv.db` 파일이 생성이 되었어야 했는데 생성이 안되어 있음

`Users/user/` 디렉토리에서 `vim test.mv.db` 으로 파일을 만들어야 접속이 됨.
맥 OS Catalina 이상 버전에서 생기는 오류인듯..




# JPA 스터디 문제
## 1. OX 퀴즈

1. 엔티티 매니저는 여러 스레드가 동시에 접근해도 안전하다. ( O / **X** )
2. 준영속 엔티티의 `memberA` 를 엔티티 매니저의 `merge()` 메소드로 영속 상태의 엔티티로 변경했다. `memberA`는 영속상태인가? ( O / **X** )

## 2. 다음 코드의 쿼리문은 몇 번 날리는가?
```java
// 트랜잭션 시작
tx.begin();

Member member = new Member();
member.setId(“id1”);
member.setUsername(“지한”);
member.setAge(2);

Member member2 = new Member();
member2.setId(“id2”);
member2.setUsername(“유한”);
member2.setAge(3);

em.persist(member);
em.persist(member2);

Member findMember = em.find(Member.class, “id1”);
Member findMember2 = em.find(Member.class, “id2”);
Member findMember3 = em.find(Member.class, “id3”); - 1

// 트랜잭션 커밋
tx.commit(); - 2

// 트랜잭션 시작
tx.begin();

em.detach(member);

Member findMember4 = em.find(Member.class, “id1”); - 1
Member findMember5 = em.find(Member.class, “id2”);
Member findMember6 = em.find(Member.class, “id3”); - 1

em.remove(findMember4);
em.remove(findMember5);

//트랜잭션 커밋
tx.commit(); - 2 (remove)
```

7번

## 3. 엔티티 매니저에 update() 메소드가 없는 이유는 무엇인가?

업데이트를 한 문장으로 처리하기위해서 이고, 이를 위해 스냅샷이라는 개념이 나왔다.

## 4. 수정 쿼리를 보낼 때 엔티티의 모든 필드를 업데이트 하는 방법의 장단점은 무엇인가?
장점

- 모든 필드의 수정 쿼리가 항상 같다. 따라서 애플리케이션 로딩 시점에 수정 쿼리를 미리 생성해두고 재사용할 수 있다.
- 데이터베이스에 동일한 쿼리를 보내면 데이터베이스는 이전에 한 번 파싱된 쿼리를 재사용할 수 있다.

단점

- 필드가 많거나 저장되는 내용이 크면 수정된 데이터만 사용해서 동적으로 update SQL을 생성하는 전략을 선택하면 된다.
    - 해결책
        - @DynamicUpdate
          - 컬럼이 대략 30개 이상이 되면, 기본 방법인 정적 수정 쿼리보다 동적 수정 쿼리가 빠르다고 한다.
          - 변경된 내용만 반영한다.