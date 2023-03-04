## 프록시   
EntityManager.find() vs EntityManager.getReference()    
- find(): 실제 엔티티 조회   
- getReference(): 가짜(프록시) 엔티티 객체 조회     
```java     
main(){
    Member member = new Member();
    entityManager.persist(member);

    entityManager.flush();
    entityManager.clear();

    Member findMember = entityManager.getReference(Member.class, member.getId());   
    // select 쿼리가 나가지 않음

    System.out.println("findMember.username = " + findMember.getUsername());
    // 엔티티를 사용할 때 쿼리가 나감
}
```     
특징    
    - 프록시 객체는 실제 객체의 참조(target)을 보관     
    - <b>프록시 객체는 처음 사용할 때 한번만 초기화</b>     
    - <b>프록시 객체가 실제 엔티티로 바뀌는 것이 아닌 프록시 객체를 통해 실제 엔티티에 접근</b>     
    - 프록시 객체는 원본 엔티티를 상속받음. 타입 체크시 주의 ( == 비교 실패, instanceof를 사용해라)   
    - 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출      
    - 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 getReference를 호출해도 실제 엔티티를 반환        
    - <b>준영속 상태일 때 프록시를 초기화하면 문제 발생( LazyInitializationException 발생 )</b>       

프록시 확인     
```java     
main(){
    Member findMember = entityManager.find(Member.class, member.getId());
    System.out.println("findMember = " + findMember.getClass());
    // findMember = class ...Member
}
```     
```java     
main(){
    Member findMember = entityManager.getReferenece(Member.class, member.getId());
    System.out.println("findMember = " + findMember.getClass());
    // findMember = class ...Member$HibernateProxy$xxxx
}
```     
- 프록시 인스턴스 초기화 여부 확인: PersistenceUnitUtill.isLoaded(entity)       
- 프록시 클래스 확인: entity.getClass()     
- 프록시 강제 초기화: Hibernate.initialize(entity)      

<br><br>

## 즉시 로딩과 지연 로딩    
Member와 Member의 Team을 동시에 조회해와야 할까?    
- 지연로딩     
    ```java     
    @Entity
	public class Member{
		@Id @GeneratedValue
		private Long id;

		private String name;

		@ManyToOne(fetch = FetchType.LAZY) // LAZY = 지연로딩
		@JoinColumn(name = "TEAM_ID")
		private Team team;
	}

	main(){
		Member findMember = entityManager.find(Member.class, memberId);
									// team관련 쿼리가 나가지않음

		System.out.println("team = " + findMember.getTeam().getClass());
	}
    ```     
    > team = class ...Team$HibernateProxy$xxx   
    
    프록시로 조회됨     
    ```java     
    main(){
		Member findMember = entityManager.find(Member.class, memberId);

		System.out.println("team = " + findMember.getTeam().getClass());

		System.out.println("======");
		String name = findMember.getTeam.getName(); // team을 사용할 때 team을 초기화(조회) 함
		System.out.println("======");
	}
    ```
- 즉시 로딩     
    ```java     
    @Entity
	public class Member{
		@Id @GeneratedValue
		private Long id;

		private String name;

		@ManyToOne(fetch = FetchType.EAGER) // EAGER = 즉시 로딩
		@JoinColumn(name = "TEAM_ID")
		private Team team;
	}

	main(){
		Member findMember = entityManager.find(Member.class, memberId);// member와 team을 join해서 조회     

		System.out.println("team = " + findMember.getTeam().getClass());
	}
    ```     
    > team = class ...Team      

    Team 클래스로 조회됨. 프록시 X

- 프록시와 즉시로딩 주의    
    - <b>되도록 지연로딩만 사용해라</b>     
    - 즉시 로딩은 예상하지 못한 sql이 발생함        
    - <b>즉시 로디은 JPQL에서 N+1문제*를 일으킨다</b>       
    - <b>@ManyToOne, @OneToOne은 즉시 로딩이 기본값. LAZY로 변경해라</b>        
    - @OneToMany, ManyToMany는 지연로딩이 기본값    

    N+1 문제
    ```java     
    main(){
		List<Member> members = entityManager.createQuery("select m 
		from Member m", Member.class).getResultList();
	}
    ```
    > select member ~, select team ~    

    Member쿼리와 Team쿼리 총 2개의 쿼리가 나감.
    - JPQL "select m from Member m" -> SQL로 번역 -> "select * from Member"쿼리가 나감      
    - 조회한 Member에 Team이 즉시 로딩으로 설정되어있어 Team을 다시 조회함
---
### 영속성 전이 CASCADE
특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때       
ex) 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장
```java     
@Entity
	public class Parent{
		@Id @GeneratedValue
		private Long id;

		private String name;

		@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL) //cascade
		private List<Child> childList = new ArrayList<>();

		public void addChild(Child child){
			childList.add(child);
			child.setParent(this);
		}
	}

	@Entity
	public class Child extends Parent{
		@Id @GeneratedValue
		private Long id;

		private String name;

		@ManyToOne
		@JoinColumn(name = "parent_id")
		private Parent parent;
	}

	main(){
		Child child1 = new Child();
		Child child2 = new Child();

		Parent parent = new Parent();
		Parent.addChild(child1);
		Parent.addChild(child2);

		//entityManager.persist(child1); // parent가 cascade all이여서 child도 영속 상태가 됨
		//entityManager.persist(child2);
		entityManager.persist(parent);
	}
```     
- <b>참조하는 곳이 하나일 때 사용해라(특정 엔티티가 개인 소유할 때)</b>     
- 영속성 전이는 연관관계 매핑과 아무 관련없음       
- 엔티티를 영속화할 떄 연관된 엔티티도 함꼐 영속화하는 편리함을 제공할 뿐       

---
### 고아 객체
- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제   
- orphanRemoval = true
```java
@Entity
	public class Parent{
		@Id @GeneratedValue
		private Long id;

		private String name;
		@OneToMany(mappedBy = "parent", orphanRemoval = true)
		private List<Child> childList = new ArrayList<>();
	}
```     

- parent.getChildList().remove(n) // 자식 엔티티를 컬렉션에서 제거      
- delete from child where ~ // delete 쿼리가 나감       

주의    
- <b>참조하는 곳이 하나일 때 사용해라(특정 엔티티가 개인 소유할 때)</b>     
- @OneToOne, @OneToMany만 가능  
- 부모엔티티를 제거하면 자식엔티티는 고아가 된다. cascadeType.REMOVE 처럼 동작      

<br><br>

## 영속성 전이 + 고아 객체, 생명주기    
- CascadeType.All + orphanRemoval=true      
- 스스로 생명주기를 관리하는 엔티티는 EntityManager로 직접 영속화(persist), 제거(remove)함      
- <b>두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식 엔티티의 생명 주기를 관리함(EntityManager를 통하지 않음)</b>      
- 도메인 주도 설계의(DDD) Aggregate Root 개념을 구현할 때 유용		