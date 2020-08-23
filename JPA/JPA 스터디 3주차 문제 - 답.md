## JPA 스터디 3주차 문제 - 답

> 다롬

1. 상속 관계를 매핑하기 위한 전략 3가지
   1. 조인 전략
   2. 단일 테이블 전략
   3. 구현 클래스마다 테이블 전략
2. @MappedSuperclass로 지정한 클래스는 엔티티인가? (O / **X**)
3. 부모 테이블의 기본키를 내려받아서 자식 테이블의 기본키 + 외래키로 사용하는 관계는 비식별 관계이다. (O / **X**)
4. 선택적 비식별 관계보다는 필수적 비식별 관계를 사용하면 좋은 이유는?
   - 선택적 비식별 관계는 외래키에 null 을 허용하므로 조인할 때 외부 조인을 사용해야 한다. 
   - 반면에, 필수적 비식별 관계는 not null로 항상 관계가 있다는 것을 보장하므로 내부 조인만 사용해도 된다.



> 경훈

질문 1. 

![image](https://user-images.githubusercontent.com/52434820/90972581-4a509f80-e555-11ea-9aa9-13a15caaf414.png)

일대다 단방향 조인 테이블 맵핑을 코드로 구현하시오.(중요 부분만 구현해도 괜찮)

```java
@Entity
public class Parent {
    
    @Id @GenerateValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
    
    @OneToMany
    @JoinTable(name = "PARENT_CHILD",
               joinColumns = @JoinColumn(name = "PARENT_ID"),
               inverseJoinColumns = joinColumn(name = "CHILD_ID"))
    private List<Child> child = new ArrayList<Child>();
    
    ...
}

@Entity
public class Child {
    
    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    private String name;
}
```





질문 2.

즉시 로딩과 지연 로딩의 차이점을 쓰시오.

- 즉시 로딩 
  - 연관된 엔티티를 즉시 조회한다. 하이버네이트는 가능하면 SQL 조인을 사용해서 한 번에 조회한다.
  - FetchType.EAGER
- 지연 로딩 
  - 연관된 엔티티를 프록시로 조회한다. 프록시를 실제 사용할 때 초기화하면서 데이터베이스를 조회한다.
  - FetchType.LAZY



질문 3.

![KakaoTalk_20200823_152931866](https://user-images.githubusercontent.com/44438366/90976127-711ece00-e575-11ea-9379-483badd577dd.jpg)

임베디드 타입과 연관관계를 가진 그림이다. 이를 코드로 표현하라. p.326

```java
@Entity
public class Member {
    
    @Embedded Address address;
    @Embedded PhoneNumber phoneNumber;
}

@Embeddable
public class Address {
    String street;
    String city;
    String state;
    @Embedded Zipcode zipcode;
}

@Embeddable
public class Zipcode {
    String zip;
    String plusFour;
}

@Embeddable
public class PhoneNumber {
    String areaCode;
    String localNumber;
    @ManyToOne PhoneServiceProvider provider;
    ...
}

@Entity
public class PhoneServiceProvider {
    @Id String name;
    ...
}
```





> 선재

1. 슈퍼타입 서브타입 논리 모델을 실제 물리 모델인 테이블로 구현할 때의 세 가지 전략 중 어떤 전략이 좋다고 생각하는지 설명해주세요.
   - 조인 전략
     - 장점
       - 단일 테이블 전략, 구현 클래스마다 테이블 전략에 비해서 테이블이 정규화되고
       - 외래 키 참조 무결성 제약 조건을 활용할 수 있다.
       - 저장공간을 효율적으로 사용할 수 있다.
     - 단점
       - 조인이 많이 사용되므로 성능이 저하될 수 있다.
       - 조회 쿼리가 복잡하다.
       - 데이터를 등록할 INSERT SQL을 두 번 실행한다.
         

2. 복합 키를 매핑할 때의 두 방법을 설명하고 차이점을 설명해주세요.

   - @IdClass

     - 식별자 클래스를 별도로 만들어 엔티티 클래스의 아이디 컬럼과 매핑시킨다. 엔티티 클래스에 식별자 클래스를 매핑시킨 @IdClass 어노테이션을 붙인다.

   - @EmbeddedId

     - 식별자 클래스에서 기본 키를 직접 매핑한다.
       


     @IdClass 는 식별자 클래스를 몰라도 식별자 필드에 직접 매핑해서 저장할 수 있다. 영속성 컨텍스트에 엔티티를 등록하기 직전에 내부에서 식별자 클래스를 생성해 영속성 컨텍스트의 키로 사용한다. 복합 키로 조회할 때는 식별자 클래스를 알아야 한다. 데이터베이스 친화적이다. 식별관계 매핑을 여러 테이블에서 사용할 때, 객체 연관관계를 단순하게 유지할 수 있다.


     @EmbeddedId 는 엔티티를 생성할 때 @Embeddable 어노테이션을 가진 식별자 클래스를 직접 생성해서 사용해야 한다. @IdClass와 비교해서 더 객체지향적이고 중복도 없다. 그러나 특정 상황에 JPQL이 조금 더 길어질 수 있다. 복합키 구조가 2개 이상 테이블에 식별관계로 매핑될 때 복잡도가 증가한다.
     [참고 링크](https://woowabros.github.io/experience/2019/01/04/composit-key-jpa.html)



3. @IdClass이 @EmbeddedId 와 비교해서 JPQL이 더 짧은 이유를 설명해 주세요.

   - IdClass 는 엔티티에 기본 키 컬럼이 있고, 식별자 클래스를 **기본 키 컬럼에** 매핑 시킨 것이다.

   - 반면에, @EmbeddedId 는 **식별자 클래스에서** 직접 기본 키 컬럼을 매핑시킨다. 따라서 기본키를 가져오려면, 식별자 객체를 통해서 가져와야 한다.

     이런 이유로 기본키를 가져오는 @IdClass 의 JPQL 문이 더 짧다.
     

5. 프록시 객체의 식별자 조회 메소드를 호출해도 프록시를 초기화하지 않는 이유는?

   - 엔티티 접근 방식 프로퍼티가 `@Access(AccessType.PROPERTY)`로 설정한 경우에만 초기화하지 않는다.
   - `AccessType.FIELD`로 설정하면 id만 조회하는 메소드인지 다른 필드까지 활용해서 어떤 일을 하는 메소드인지 알지 못하므로 프록시 객체를 초기화한다.

6. 값 타입을 여러 엔티티에서 공유하면 어떤 문제가 발생하는지, 어떻게 해결하는지 설명해 주세요.

   - 한 엔티티에서 공유한 값을 변경하면, 의도치 않게 같은 인스턴스를 참조하고 있던 다른 엔티티들의 값도 변경되어  각각 UPDATE SQL을 실행한다.

   - 그래서 값 타입 복사(clone)를 하거나 불변 객체로 만들어 사용한다.
   - 값 타입 복사의 경우, 복사하지 않고 원본의 참조 값을 직접 넘기는 것을 막을 방법이 없다는 문제가 있기 때문에(C++에서는 커스텀 가능하지만 java에서는 불가) 값 타입 부작용 걱정없이 객체를 불변하게 만들어 사용한다.
   - 불변 객체의 가장 간단한 방법은 생성자로만 값을 설정하고 수정자를 만들지 않는 것이다. 참고로 Integer, String은 자바가 제공하는 대표적인 불변 객체다. 




### 문제. OX 퀴즈

1. 복합 키를 구성하는 여러 칼럼에 @GeneratedValue를 사용할 수 있다. ( O / **X** )
2. 엔티티 클래스는 같은 엔티티 클래스만 상속받을 수 있다. ( O / **X** )
   - MappedSuperClass도 가능
3. 일대일 조인 테이블을 만드려면, 조인 테이블에 외래 키 칼럼 중 하나만 유니크 제약 조건을 걸어야 한다. ( O / **X** )
4. 데이터베이스를 조회할 필요가 없다면 em.getReference()를 호출 결과는 실제 엔티티를 반환한다. (**O** / X ) 
   - 영속성 컨텍스트에 있다면 O, 없었다면 프록시객체 반환으로 X
5. PersistenceUnitUtil.isLoaded(entity) 호출 시 프록시 인스턴스가 아니라면 false 를 반환한다. ( O / **X** )
6. JPA는 즉시 로딩을 최적화하기 위해 가능하면 조인 쿼리를 사용한다. ( **O** / X )
7. JPA 기본 페치 전략 중 @OneToMany, @ManyToMany 는 즉시 로딩 전략이다. ( O / **X** )



> 승현

1. 이름 필드를 가지는 Animal 클래스와 Dog 클래스를 조인 테이블을 사용해서 단방향으로 일대일 매핑하는 코드 작성 

   ```java
   @Entity
   public abstract class Animal{
       
       @ID @GeneratedValue
       @Column(name = "ANIMAL_ID")
       private Long id;
       private String name;
       
       @OneToOne
       @JoinTable(name = "ANIMAL_DOG",
                  joinColumns = @JoinColumn(name = "ANIMAL_ID"),
                  inverseJoinColumns = @JoinColumn(name = "DOG_ID"))
       private Dog dog;
   }
   
   @Entity
   public class Dog {
       
       @ID @GeneratedValue
       @Column(name = "DOG_ID")
       private Long id;
       private String name;
       
       ...
   }
   ```

   

2. 이름, 가격 필드를 가지는 Equipment 클래스를 상속하는 Keyboard, Laptop, Phone 클래스를 구현 클래스마다 테이블 전략으로 작성

   ```java
   @Entity
   @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
   public abstract class Equipment {
       
       @ID @GeneratedValue
       @Column(name = "EQUIPMENT_ID")
       private Long id;
       
       private String name;
       private int price;
   }
   
   @Entity
   public class Keyboard extends Equipment { ... }
   
   @Entity
   public class Laptop extends Equipment { ... }
   
   @Entity
   public class Phone extends Equipment { ... }
   ```

   

3. 프록시 객체는 식별자 값을 가지고 있으므로 JPA는 식별자 값을 조회하는 메소드를 호출해도 항상 프록시를 초기화하지 않는다. (O/**X**) 

   - 엔티티 접근 방식을  `@Access(AccessType.PROPERTY)`로 설정한 경우에만 초기화하지 않고, 
   - `@Access(AccessType.FIELD)`로 설정한 경우에는 getId 메소드가 id만 조회하는 메소드인지 다른 메소드 까지 활용하는지 알지 못하므로 프록시 객체를 초기화 한다.

4. 즉시 로딩을 사용하는 방법

   - @ManyToOne(fetch = FetchType.EAGER) 
   - @OneToOne(fetch = FetchType.EAGER)
     

5. 연관관계별 `FetchType.EAGER` 설정과 조인 전략 

   - @ManyToOne, @OneToOne
     - (optional = false): 내부 조인
     - (optional = true): 외부 조인
   - @OneToMany, @ManyToMany
     - (optional = false): 외부 조인
     - (optional = true): 외부 조인
       

6. 몇 번의 INSERT SQL이 실행될까?

 ```java
@Entity
public class Member {
  
  @Id @GeneratedValue
  private Long id;
  
  @Embedded
  private Addess homeAddress;
  
  @ElementCollection
  @CollectionTable(name = "FAVORITE_FOODS",
                  joinColumns = @JoinColumn(name = "MEMBER_ID"))
  @Column(name = "FOOD_NAME")
  private Set<String> favoriteFoods = new HashSet<String>();
  
  @ElementCollection
  @CollectionTable(name = "ADDRESS",
                  joinColumns = @JoinColumn(name = "MEMBER_ID"))
  private List<Address> addressHistory = new ArrayList<Address>();
  ...
}
 ```

```java
Member member = new Member();

member.setHomeAddress(new Address("서울", "백범로", "12345")); // 컬렉션이 아닌 임베디드 값 타입이므로 회원 테이블을 저장하는 SQL에 포함된다.

member.getFavoriteFoods().add("아메리카노");
member.getFavoriteFoods().add("와플");
member.getFavoriteFoods().add("파스타");
member.getFavoriteFoods().add("샌드위치");
member.getFavoriteFoods().add("김밥");

member.getAddressHistory().add(new Address("서울", "양화로", "34555"));
member.getAddressHistory().add(new Address("서울", "서강대길", "33455"));

em.persist(member);
```

- 멤버 추가 1번, 값 타입 컬렉션에 추가될 때마다 INSERT SQL 7번, 총 8번