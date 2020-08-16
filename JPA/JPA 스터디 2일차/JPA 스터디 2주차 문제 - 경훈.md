# JPA

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

두 결과의 차이점과 그 이유를 말하시오.

- Question1의 결과를 조회해보면 team필드에 null이 들어가게 된다.
- 이유는 연관관계 주인이 아닌 곳에서 입력된 값은 외래 키에 영향을 주지 않기 때문에, 데이터 베이스에 저장할 때 무시된다.

이 문제를 낼 때는 객체에서 어떤 객체가 연관관계의 주인인지 나타내는 엔티티 코드가 먼저 명시되어야 할듯!



문제 3.

다대다 관계에서 중간에 연결 테이블을 추가해야되는 이유가 무엇인가?

- 추가적인 필드가 필요할 때 다대다 매핑으로 생성되는 테이블로는 해결할 수 없기 때문에 새로운 엔티티를 생성해야 한다.