### JPA 2주차 문제

> 승현

1. JPA가 데이터베이스 스키마를 자동으로 생성하도록 하기 위해 `persistence.xml`에 추가해야 할 설정은?

   ```
   <property name="hibernate.hbm2ddl.auto" value="create" />
   ```

   

2. 키, 이름, 몸무게 필드를 Person 클래스를 테이블과 매핑하되, 매핑할 테이블 이름은 PERSON으로 하고 기본 키 매핑은 SEQUENCE 전략을 사용하도록 코드 작성



3. 2번에 사용된 클래스를 테이블과 매핑하되, 아래 테이블을 키 생성 용도(seq_name 컬럼은 시퀀스 이름, next_val 컬럼은 시퀀스 값)로 사용하는 TABLE 전략을 사용하도록 코드 작성

```sql
create table MY_SEQ (
	seq_name varchar(255) not null, 
  next_val bigint,
  primary key ( seq_name )
)
```





4. 연관관계의 주인이 아닌 곳에 값을 입력하는 경우 데이터베이스에 영향을 미치지 않는다. (**O**/X)



5. 이름, 학번과 참조로 반을 가지는 클래스 Student와 이름 필드를 가지는 반 클래스 Class가 있을 때 양방향 연관관계로 매핑하는 코드를 작성(외래 키는 CLASS_ID)

   

6. 위의 클래스와 JPA를 사용해서 양방향 모두 관계를 설정하는 메소드를 test라는 이름으로 작성

   

7. 실전 예제(235p~241p)를 참고해서 상품과 카테고리를 매핑하기 위한 연결 엔티티 CategoryItem 작성

   

8. 주 테이블에 외래 키가 있는 양방향 매핑을 적용해서 회원 엔티티(이름 필드 존재)와 지갑 엔티티(돈 필드 존재) 작성

   

9. 다대다 단방향 관계인 사람(이름 필드 존재)과 일기 엔티티(제목, 내용 필드 존재) 작성