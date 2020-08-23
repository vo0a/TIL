## JPA 스터디 2주차 문제 - 답

> 다롬

1. 
   객체 연관관계 vs 테이블 연관관계 정리
   - 참조를 사용하는 객체의 연관관계는 (단)방향이다.
   - 외래 키를 사용하는 테이블의 연관관계는 (양)방향이다.
2. 객체 연관관계에서, 객체는 참조를 사용해서 연관관계를 탐색할 수 있는데 이것을 (객체 그래프 탐색)이라고 한다.
3. JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 (영속) 상태여야 한다.
4. 연관관계의 주인은 테이블에 (외래) 키가 있는 곳으로 정해야 한다. 
   - 주인만 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있다.
   - 주인이 아닌 반대편은 읽기만 가능하고 외래 키를 변경하지는 못한다.



> 선재

1. 일대다와 다대일의 차이점은 무엇인가?
   - 관리해야 할 외래키가 있는 장소의 차이. 관리하는 쪽에 외래키가 있다.
   - 먼저 오는 쪽이 연관관계의 주인.  
     

2. 일대다 단방향의 단점은 무엇인가?
   - 일대다인 경우 외래키 관리를 연관관계의 주인이 아닌 쪽에서 하기 때문에 추가적인 update문이 필요, 따라서 양방향매핑을 해주는 것이 좋다.
     

3. 양방향일 때의 주의점은 무엇이 있는가?
   - 연관관계의 주인에는 값을 입력하지 않고, 주인이 아닌 곳에만 값을 입력하는 것
     - null이 저장 됨
   - 무한루프에 빠지지 않도록 주의. (p. 195)
     - Meber.toString()에서 getTeam()을 호출하고 Team.toString에서 getMember()를 호출하면 무한루프에 빠질 수 있다.
     - 무한루프에 빠지지 않도록 하는 어노테이션이나 기능이 있다.



4. 일대일 관계에서 주 테이블에 외래 키를 가지고 있을 때와 대상 데이블에 외래 키를 가지고 있을 때의 차이점은 무엇인가?
   - 주 테이블이 외래키를 갖고 있으므로 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있다.
   - 대상 테이블에 외래키가 있으면, 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다.(JPA에서 지원하지 않는다. 그래서 보통 일대일 매핑에서 대상 테이블에 외래 키를 두고 싶으면 양방향으로 매핑한다.)



5. 다대다 매핑에서 복합 기본 키를 이용한 경우와 새로운 기본 키 사용한 경우와 어떤 차이점이 있는가?
   - 복합 기본 키를 사용한 경우 식별자 클래스를 사용해야 했고, 새로운 기본 키를 사용한 경우에는 식별자 클래스를 사용하지 않아도 되기 때문에 단순하고 편리하게 ORM 매핑을 할 수 있다.



> 경훈

질문 1.

```jsx
@GeneratedValue(startegy = GenerationType.IDENTITY)
```

가 뜻하는 바는 무엇인가?

- 기본키 생성을 데이터베이스에 위임하는 전략이다.



질문 2.

```jsx
public void Question() {

	Team team1 = new Team("team1", "팀1");
	Member member1 = new Member("member1", "회원1");
	Member member2 = new Member("member2", "회원2");
	
	member1.setTeam(team1);
	member2.setTeam(team1);

	List<Member> members = team.getMembers();
	System.out.println("members.size = " + members.size());

}
```

```jsx
public void Question1() {

	Team team1 = new Team("team1", "팀1");
	Member member1 = new Member("member1", "회원1");
	Member member2 = new Member("member2", "회원2");
	
	member1.setTeam(team1);
	team1.getMembers().add(member1);

	member2.setTeam(team1);
	team1.getMembers().add(member2);

	List<Member> members = team.getMembers();
	System.out.println("members.size = " + members.size());

}
```

Member가 연관관계의 주인 일 때, 두 결과의 차이점과 그 이유를 말하시오.

- Question1의 결과를 조회해보면 team필드에 null이 들어가게 된다.
- 이유는 연관관계 주인이 아닌 곳에서 입력된 값은 외래 키에 영향을 주지 않기 때문에, 데이터 베이스에 저장할 때 무시된다.



문제 3.

다대다 관계에서 중간에 연결 테이블을 추가해야되는 이유가 무엇인가?

- 추가적인 필드가 필요할 때 다대다 매핑으로 생성되는 테이블로는 해결할 수 없기 때문에 새로운 엔티티를 생성해야 한다.



> 승현

1. JPA가 데이터베이스 스키마를 자동으로 생성하도록 하기 위해 `persistence.xml`에 추가해야 할 설정은?

   ```
   <property name="hibernate.hbm2ddl.auto" value="create" />
   ```

   

2. 키, 이름, 몸무게 필드를 Person 클래스를 테이블과 매핑하되, 매핑할 테이블 이름은 PERSON으로 하고 기본 키 매핑은 SEQUENCE 전략을 사용하도록 코드 작성

   ```java
   @Entity
   @SequenceGenerator (
       name = "PERSON_SEQ_GENERATOR",
       sequenceName = "PERSON_SEQ",
       initialValue = 1, allocationSize = 1)
   public class Person {
       
       @Id
       @GenerateValue(strategy = GenerationType.SEQUENCE,
                      generator = "PERSION_SEQ_GENERATOR")
       private Long id;
       
       private int weight;
       private int height;
   }
   ```

   



3. 2번에 사용된 클래스를 테이블과 매핑하되, 아래 테이블을 키 생성 용도(seq_name 컬럼은 시퀀스 이름, next_val 컬럼은 시퀀스 값)로 사용하는 TABLE 전략을 사용하도록 코드 작성

   ```sql
   create table MY_SEQ (
   	seq_name varchar(255) not null, 
    	next_val bigint,
    	primary key ( seq_name )
   )
   ```

   ```java
   @Entity
   @Table(name="PERSON")
   @TableGernerator(
   	name = "PERSON_SEQ_GENERATOR"
   	table = "MY_SEQ",
   	initialValue = 1, allocationSize = 1)
   public class Person {
       
       @Id
       @GeneratedValue(strategy = GenerationType.TABLE,
                      	generator = "PERSON_SEQ_GENERATOR")
       private Long id;
       
       private int weight;
       private int height;
   }
   ```

   

4. 연관관계의 주인이 아닌 곳에 값을 입력하는 경우 데이터베이스에 영향을 미치지 않는다. (**O**/X)



5. 이름, 학번과 참조로 반을 가지는 클래스 Student와 이름 필드를 가지는 반 클래스 Class가 있을 때 양방향 연관관계로 매핑하는 코드를 작성(외래 키는 CLASS_ID)

   ```java
   @Entity
   public class Student {
       
       @Id
       @Column(name = "STUDENT_ID")
       private Long id;
       
       private String name;
       
       @ManyToOne
       private MyClass myClass;
       
       public void setClass(MyClass myClass){
           this.myClass = myClass;
       }
       ...
   }
   
   @Entity
   public class myClass {
       
       @Id
       @Column(name = "CLASS_ID")
       private Long id;
       
       private String name;
       
       @OneToMany(mappedBy = "myClass")
       private List<Student> students = new ArrayList<Student>();
   }
   
   ```

   

   

6. 위의 클래스와 JPA를 사용해서 양방향 모두 관계를 설정하는 메소드를 test라는 이름으로 작성

   ```java
   public void test() {
       
       EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
       EntityManager em = emf.createEntityManager();
       
       MyClass myClass1 = new MyClass("class1", "반1");
       em.persist(myClass1);
       
       Student student1 = new Student("student1", "학생1");
       
       student1.setClass(myClass1);
       myClass.getStudents().add(student1);
       em.persist(student1);
       
       Student student2 = new Student("student2", "학생2");
       
       student2.setClass(myClass1);
       myClass1.getStudents().add(student2);
       em.persist(student2);
   }
   ```

   

   

7. 실전 예제(235p~241p)를 참고해서 상품과 카테고리를 매핑하기 위한 연결 엔티티 CategoryItem 작성

   ```java
   @Entity
   @Table(name = "CATEGORY_ITEM")
   public class CategoryItem {
       
       @Id @GeneratedValue
       @Column(name = "CATEGORY_ITEM_ID")
       private Long id;
       
       @ManyToOne
       @JoinColumn(name = "CATEGORY_ID")
       private Category category;
       
       @ManyToOne
       @JoinColumn(name = "ITEM_ID")
       private Item item;
       
       ...
       
   }
   ```

   

   

8. 주 테이블에 외래 키가 있는 양방향 매핑을 적용해서 회원 엔티티(이름 필드 존재)와 지갑 엔티티(돈 필드 존재) 작성

   ```java
   @Entity
   @Table(name = "MEMBER")
   public class Member {
       
       @Id @GeneratedValue
       @Column(name = "MEMBER_ID")
       private Long id;
       
       private String name;
       
       @OneToOne
       @JoinColumn(name = "WALLET_ID")
       private Wallet wallet;
       
       ...
   }
   
   @Entity
   @Table(name = "WALLET")
   public class Wallet {
       
       @Id @GeneratedValue
       @Column(name = "WALLET_ID")
       private Long id;
       
       private int money;
       
       @OneToOne(mappedBy = "wallet")
       private Member member;
       
       ...
   }
   ```

   

   

9. 다대다 단방향 관계인 사람(이름 필드 존재)과 일기 엔티티(제목, 내용 필드 존재) 작성

   ```java
   @Entity
   public class Person {
       
       @Id @Column(name = "PERSON_ID")
       private Long id;
       
       private String name;
       
       @ManyToMany
       @JoinTable(name = "PERSON_DIARY",
                  joinColumns = @JoinColumn(name = "PERSON_ID"),
                  inverseJoinColumns = @JoinColumn(name = "DIARY_ID"))
       private List<Diary> diaries = new ArrayList<Diary>();
       
       ...
   }
   
   @Entity
   public class Diary {
       
       @Id @Column(name = "DIARY_ID")
       private Long id;
       
       private String title;
       private String content;
   }
   ```

   