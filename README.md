### jpa     
- [JPA(Java Persistence Api)](#jpa---java-persistence-api) - [링크](https://github.com/rrlqja/study_jpa/blob/master/jpa.md)       
- [영속성 컨텍스트(Persistence Context)](#영속성-컨텍스트-persistence-context) - [링크](https://github.com/rrlqja/study_jpa/blob/master/persistence-context.md)       
    - 엔티티의 생명주기     
    - 영속성 컨텍스트의 이점        
    - 플러시 (flush)     
    - 준영속 상태       
- [엔티티 매핑](#엔티티-매핑) - [링크](https://github.com/rrlqja/study_jpa/blob/master/entity-mapping.md)   
- [연관관계 매핑 기초](#연관관계-매핑-기초) - [링크](https://github.com/rrlqja/study_jpa/blob/master/relational-mapping-basic.md)    
    - 단방향 연관관계       
    - 양방향 연관관계
    - 양방향 연관관계 정리
- [다양한 연관관계 매핑](#다양한-연관관계-매핑) - [링크](https://github.com/rrlqja/study_jpa/blob/master/relational-mapping.md)     
- [연관관계 매핑 고급](#상속관계) - [링크](https://github.com/rrlqja/study_jpa/blob/master/relational-mapping-advanced.md)    
    - 상속관계      
- [프록시](#프록시) - [링크](https://github.com/rrlqja/study_jpa/blob/master/proxy.md)    
    - 프록시    
    - 즉시 로딩과 지연 로딩     
    - 영속성 전이   
    - 고아 객체     
    - 영속성 전이 + 고아 객체 - 생명주기

___     

## JPA - Java Persistence Api  
Java Orm 표준  
Java Application과 JDBC 중간에서 동작한다.  
> __Orm__ (Object Relational Mapping)  
객체 관계 매핑  
객체는 객체대로 관계형DB는 관계형DB대로 설계, Orm 프레임워크가 중간에서 매핑  

<br><br>   

## Jpa 사용 이유   
* sql 중심적 개발에서 객체 중심 개발 가능   
* 패러다임의 불일치 해결    

<br><br>   

## 패러다임의 불일치   
객체지향 프로그래밍의 경우 추상화, 캡슐화, 상속, 다형성 등 다양한 객체 제어 방법이 존재하지만 관계형 데이터베이스에는 존재하지않는다.     
jpa는 이러한 패러다임의 불일치를 해결할 수 있다.    

* ### jpa 상속  
    객체는 상속이 있지만 관계형 데이터베이스는 상속개념이 없다. 상속관계의 객체를 테이블에 저장, 조회하려면 관련 쿼리 여러개를 보내야한다. jpa는 이를 알아서 처리해줌.    

* ### jpa 연관관계, 객체 그래프 탐색    
    객체는 참조값을 이용하여, 테이블은 외래키 값을 이용하여 관련 객체, 테이블을 조회함.     
jpa는 <b>entity의 필드에 다른 entity의 참조값을 저장</b>하여 연관관계매핑이 가능하다.  

* ### jpa 비교  
    동일한 트랜잭션에서 조회한 entity는 같음(== 비교)을 보장함.    
* ### jpa 성능 최적화   
    #### 1. 1차 채시와 동일성 보장   
    >같은 트랜잭션 안에서느 같은 entity를 반환.  
    DB Islocation Level이 Read Commit이어도 애플리케이션 Repeatable Read 보장.      
    #### 2. 트랜잭션을 지원하는 쓰기 지연    
    >트랜잭션을 커밋할때까지 Insert Sql을 모아놓음.  
    Jdbc Batch Sql 기능을 이용하여 한번에 sql을 전송함.     
    #### 3. 지연 로딩과 즉시 로딩    
    >지연 로딩: 객체가 실제 사용될 때 로딩   
    즉시 로딩: join sql로 한번에 연관된 객체를 미리 조회    


___     

## 영속성 컨텍스트 (Persistence Context)    
- 엔티티(entity)를 영구 저장하는 환경   
- 눈에 보이지 않는 논리적인 개념    
- 엔티티 매니저(entity manager)를 통해 영속성 컨텍스트에 접근   

<br><br>

### 엔티티의 생명주기   
- <b>비영속(new/transient)</b>: 영속성 컨텍스트와 전혀 관계가 없는 상태, 새로운 객체를 생성만 한 상태    
- <b>영속(managed)</b>: 영속성 컨텍스트에 관리되는 상태. 새로운객체를 엔티티 매니저를 통해 저장, 조회 등을 한 상태   
- <b>준영속(detached)</b>: 영속성 컨텍스트에 저장되었다가 분리된 상태  
    ```java     
    entitymanager.detach(entity);   
    ```    
- <b>삭제(removed)</b>: 영속성 컨텍스트에 삭제된 상태.      
    ```java     
    entitymanger.remove(entity);    
    ```     

<br><br>

### 영속성 컨텍스트의 이점  
- <b>1차 캐시</b>  
    - 엔티티 저장시 1차 캐시에 먼저 저장.   
    - 엔티티 조회시 1차 캐시에서 먼저 조회 후 캐시에 없으면 쿼리를 날려 조회.   
- <b>동일성 보장</b>   
    - 영속 엔티티의 동일성을 보장함.    
    ```java 
    member1 = entitymanger.find(member.class, memberID1);
    member2 = entitymanger.find(member.class, memberID1);       
    member1 == member2; // true
    ```     
- <b>트랜잭션을 지원하는 쓰기 지연</b>     
    - 엔티티 매니저에 persist를 호출해도 sql이 데이터베이스로 전송되지 않고, 쓰기 지연 sql 저장소에 sql을 저장함.   
    - 트랜잭션 커밋시점에 쓰기 지연 sql 저장소의 쿼리를 데이터베이스에 전송함.      
    ```java 
    entitymanager.persist(entityA);
    entitymanager.persist(entityB);
    //sql을 전송하지 않음

    //커밋하는 순간에 sql전송
    transaction.commit()//커밋    
    ```     
- <b>변경 감지(dirty checking)</b>     
    - 엔티티를 조회 후 필드값을 변경한 후에 따로 update 쿼리를 만들지 않아도 jpa가 알아서 update 쿼리르 보냄.   
    ```java    
    member = entitymanger.find(member.class, memberID);
    member.setName("홍길동");

    //entitymanager.update(member) 이러한 코드를 작성하지 않아도 됨.

    transaction.commit()// 자동으로 update 쿼리를 보냄.     
    ```     
    > 트랜잭션이 커밋되면 플러시가 발생함. 플러시가 발생되면 1차 캐시 내부의 스냅샷과 엔티티를 비교 후 update sql을 생성후 데이터베이스에 보냄.     
    
- <b>지연 로딩(lazy loading)</b>   

<br><br>

### 플러시(flush)    
- <b>영속성 컨텍스트의 변경 내용을 데이터베이스에 반영.</b>    
- <b>플러시 발생시: 변경 감지, 수정된 엔티티를 쓰기 지연 sql 저장소에 저장. 쓰기 지연 sql 저장소의 쿼리를 데이터베이스에 전송.</b>     
- 플러시하는 방법   
    1. entitymanager.flush() - 플러시 직접 호출   
    2. 트랜잭션 커밋 - 플러시 자동 호출
    3. jpql 쿼리 실행 - 플러시 자동 호출   
    ```java     
    entitymanager.persist(memberA)
    entitymanager.persist(memberB)
    entitymanager.persist(memberC)

    //중간에 jpql 실행
    entitymanager.createQuery(query)//멤버 엔티티가 저장이 안된 상태에서 쿼리를 보내면 원치않는 결과가 나올 수 있음.
                                    //이러한 결과를 방지하고자 jpql 실행시 플러시가 자동으로 호출됨.
    ```     
<br><br>

### 준영속 상태     
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태.   
- 영속성 컨텍스트가 제공하는 기능을 사용 못함.
- 준영속 상태를 만드는 방법.  
    1. entitymanger.detach(entity) - 특정 엔티티만 준영속 상태로 만듬.  
    2. entitymanger.clear() - 영속성 컨텍스트를 완전 초기화 
    3. entitymanger.close() - 영속성 컨텍스트를 종료    


___     

## 엔티티 매핑  
객체와 테이블 매핑    
- 객체와 테이블 매핑: <b>@Entity, @Table</b>   
    - @Entity: @Entity가 붙은 클래스는 jpa가 관리하는 엔티티라고 한다. 
    - jpa를 사용해서 테이블과 매핑할 클래스는 @Entity 필수.    
    - 기본 생성자 필수(public 또는 protected)  
    - final, enum, interface, inner 클래스 X    
    - 저장할 필드에 final 사용 X   
        ```java
        @Entity //jpa entity
        public class ExEntity{
            @Id
            private Long Id;

            public ExEntity(){
            }
        }
        ```
- 필드와 컬럼 매핑  
    ```java
    @Entity
    public class ExEntity{
        @Id
        private Long Id;

        @Column(name="username")// 컬럼 매핑
        private String userName;

        @Enumerated(EnumType.String)// enum 타입 매핑
        private EnumClass enumClass;

        @Temporal(TemporalType.TIMESTAMP)// 날짜 타입 매핑
        private Date createDate;

        @Lob// BLOB, CLOB 매핑
        private String description;

        @Transient// 필드를 컬럼에 매핑하지 않을때
        private int temp;
    }
    ```
- 기본 키 매핑     
    - <b>@Id</b>   
    - <b>@GeneratedValue: 기본키 생성을 데이터베이스에 위임.</b>   
        - Identity: 데이터베이스에 위임. Mysql     
        - Sequence: 데이터베이스 시퀀스 오브젝트 사용. Oracle      
            @SequenceGenerator 필요.    
        - Table: 키 생성용 테이블 사용. 모든 DB에서 사용.  
            @TableGenerator 필요.   
        - Auto: 데이터베이스 방언에 따라 자동 지정. 기본값     
            ```java
            @Entity
            public class ExEntity{
                @Id
                @GeneratedValue(strategy = GenerationType.Auto)
                private Long id;
            }
            ```
- 연관관계 매핑: @ManyToOne, @JoinColumn    

___         

## 연관관계 매핑 기초   
- 객체와 테이블의 연관관계의 차이   
    1. 테이블은 외래키를 조인하여 연관된 테이블을 찾는다.   
    2. 객체는 참조값을 사용하여 연관된 객체를 찾는다.     
- 객체의 참조와 테이블의 외래키를 매핑함.   
- 방향(direction): 단방향, 양방향   
- 다중성(multiplicity): 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)  
- 연관관계 주인(owner): 객체 양방향 연관관계는 관리 주인이 필요함.       

<br><br>

## 단방향 연관관계  
Member -> Team  
```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;

    @ManyToOne // Member 입장에서 Team은 N:1 관계
    @JoinColumn(name = "TEAM_ID") // name 속성: Member 테이블의 join 컬럼의 컬럼명     
                                  // referencedColumnName = xxx: 연관관계를 맺을 테이블의 컬럼을 지정가능     
                                  // 생략하면 자동으로 pk값으로 연관관계를 맺음
    private Team team;
}

@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;

    private String name;
}

public static void main(String[] args){
    Team teamA = new Team();
    teamA.setName("teamA");
    entityManager.persist(teamA);

    Member memberA = new Member();
    memberA.setName("memberA");
    memberA.setTeam(team); // 단방향 연관관계 설정. 참조값 저장
    entityManager.persist(memberA);

    entityManager.flush();
    entityManager.clear();

    Member findMember = entityManager.find(Member.class, memberId);
    Team findTeam = findMember.getTeam()// team 값 가져올 수 있음
}
```     

<br><br>

## 양방향 연관관계      
> 데이터베이스 테이블은 외래키를 통해 연관된 테이블을 모두 조회 가능.   
> 객체는 객체마다 다른 객체의 참조값을 가지고 있어야함.     

```java
@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany( mappedBy = "team")// Team 입장에서 Member는 1:N 관계
                                  // mappedBy = "xxx" 연관관계인 객체의 변수명 
    private List<Member> members = new ArrayList<>();
}
```     
- 연관관계 주인(mappedBy)   
    - 객체와 테이블이 관계를 맺는 차이      
        - 객체 연관관계 = 2개      
            1. 회원 -> 팀 연관관계 1개(단방향)      
            2. 팀 -> 회원 연관관계 1개(단방향)      
            > 객체의 양방향 관계는 양방향 관계가 아닌 서로 다른 단방향 관계 2개인 것.     
        - 테이블 연관관계 1개   
            1. 회원 - 팀 연관관계 1개 (양방향)      
            > 테이블 연관관계는 외래키 하나로 두 테이블의 연관관계를 가진다.    

    테이블은 외래키 하나로 연관관계를 맺고 객체는 각 객체 필드의 참조값으로 연관관계를 맺는다.      
    이 차이로 인해 연관관계가 맺어진 두 객체중 어떤 객체의 참조값을 변경했을때 테이블의 값이 변경돼야 하는지를 지정해야된다.    
    즉 <b>연관관계를 맺은 객체중 하나를 연관관계의 주인으로 지정해야된다.</b>       
    <b>연관관계 주인만이 외래키를 관리(등록, 수정)한다.</b>     
    <b>주인이 아닌쪽은 읽기만 가능.</b>    
    주인은 mappedBy 속성 사용 X.        
    주인이 아니면 mappedBy 속성으로 주인 지정.      

    - <b>누구를 주인으로?</b>      
        외래키가 있는 곳을 주인으로 지정해라.( 무조건 적인 강요는 아니지만 외래키가 있는곳을 주인으로 지정해야 개발하기 수월하다.)      
        ex) 외래키가 없는 곳(여기선 Team)을 주인으로 지정했을때.    
        Team의 members를 수정했을 때 Team관련 sql이 생성되는게 아닌 member와 관련된 sql이 생성된다.    

    - 연관관계 주의     
        1. 항상 양쪽에 값을 설정하자.( member.team = xxx, team.members().add(xxx) )     
        2. 연관관계 편의 매서드를 생성, 사용해라.   
            ```java
            @Entity
            public class Member{
                @Id @GeneratedValue
                private Long id;

                @ManyToOne
                @JoinColumn(name = "TEAM_ID")
                public Team team;

                // 연관관계 편의 매서드
                public void changeTeam(Team team){
                    this.team = team;
                    team.getMembers.add(this);
                }
            }
            ```     
        3. 양방향 매핑시 무한 루프를 조심하자 ex) toString()     

<br><br>

## 양방향 매핑 정리     
- 단방향 매핑만으로 연관관계 매핑을 완료해라    
- 양방향 매핑은 조회(객체 그래프 탐색) 기능이 추가되는 것 뿐        
- 단방향 매핑을 잘 하고 양방향 매핑은 필요할 때 추가해라    

___             

## 다양한 연관관계 매핑     
- 고려사항      
    1. 다중성   
        - 다대일(N:1): @ManyToOne           
        - 일대다(1:N): @OneToMany        
        - 일대일(1:1): @OneToOne        
        - 다대다(N:M): @ManyToMany    
        > 다중성이 헷갈릴 때는 데이터베이스 테이블의 관점에서 생각해보자
    2. 단방향, 양방향       
        - 테이블: 외래키 하나로 양쪽 조인 가능. 방향이라는 개념이 없음      
        - 객체: 참조용 필드가 있는 쪽으로만 참조 가능. 한쪽만 참조하면 단방향, 양쪽이 서로 참조하면 양방향      
    3. <b>연관관계 주인</b>        
        테이블과 객체가 연관관계를 맺는 방법의 차이가 있음.     
        객체는 참조가 두군데있기 때문에 둘중 테이블의 외래키를 관리할 곳을 지정해야함.      
        - 연관관계 주인: 외래키를 관리하는 참조     
        - 주인의 반대편: 외래키에 영향을 주지 않음. <b>조회만 가능</b>

- <b>다대일(N:1) 양방향 연관관계</b>   
    ![N1_1](https://user-images.githubusercontent.com/59528611/218443122-65126329-3355-4cdc-ba87-918ca9637708.jpeg)     
    - 외래키가 있는 쪽이 연관관계 주인      
    - 양쪽을 서로 참조하도록 개발
        ```java     
        @Entity
        public class Member{
            @Id @GeneratedValue
            @Column(name = "MEMBER_ID")
            private Long id;

            @Column(name = "USERNAME")
            private String username;

            @ManyToOne
            @JoinColumn(neme = "TEAM_ID")
            private Team team;
        }

        @Entity
        public class Team{
            @Id @GeneratedValue
            @Column(name = "TEAM_ID")
            private Long id;
            private String name;

            @OneToMany(mappedBy = "team")
            private List<Member> members = new ArrayList<>();	
        }
        ```         
- 일대다(1:N) 단방향 연관관계        
    ![1N_1](https://user-images.githubusercontent.com/59528611/218448668-40b0ccf5-d9ac-4d10-bea8-508d5a161d87.jpeg)     
    - 권장하지 않는 연관관계 방식       
    ```java
    @Entity
	public class Member{
		@Id @GeneratedValue
		@Column(name = "MEMBER_ID")
		private Long id;

		@Column(name = "USERNAME")
		private String username;
	}

	@Entity
	public class Team{
		@Id @GeneratedValue
		@Column(name = "TEAM_ID")
		private Long id;
		private String name;

		@OneToMany
		@JoinColumn(name = "TEAM_ID") 
		private List<Member> members = new ArrayList<>();	
	}
    ```
    - 1쪽에서 외래키를 관리하기때문에 team에서 외래키를 관리함.     
    - 데이터베이스 설계상 항상 다(N)쪽에 외래키가 존재함.       
    - 객체와 테이블의 차이 때문에 반대편 테이블의 외래키를 관리하는 특이한 구조.     
    - @JoinColumn을 사용하지 않으면 조인 테이블 방식을 사용함(중간에 테이블을 추가하는 방식)    
    > 단점   
        - 엔티티가 관리하는 외래키가 다른 테이블에 있음.    
        - 연관관계 관리를 위해 추가로 update sql실행    
    - <b>다대일 양방향 매핑을 사용하자</b>      
    
- 일대다(1:N) 양방향 연관관계   
    - 이런 매핑은 공식적으로 존재하지않음.
    - 다대일 양방향 연관관계를 사용하자.      

- 일대일(1:1) 양방향 연관관계       
    - 주 테이블이나 대상 테이블 중에 외래키 선택 가능
        - 주 테이블에 외래키        
            ![11_1](https://user-images.githubusercontent.com/59528611/218453365-2dc8d958-ea09-48a9-90f2-ca4bcece7da0.jpeg)     
            ```java
            @Entity
            public class Member{
                @Id @GeneratedValue
                @Column(name = "MEMBER_ID")
                private Long id;

                @Column(name = "USERNAME")
                private String username;

                @OneToOne
                @JoinColumn(name = "LOCKER_ID")
                private Locker locker;
            }

            @Entity
            public class Locker{
                @Id @GeneratedValue
                @Column(name = "LOCKER_ID")
                private Long id;

                @Column(name = "LOCKERNAME")
                private String username;		

                @OneToOne(mappedBy = "locker")
                private Member member;
            }
            ```     
            - 외래키가 있는 곳이 연관관계 주인. 반대편은 mappedBy 사용.
            - 주 객체가 대상 객체의 참조를 가지는 것 처럼 주 테이블에 외래키를 두고 대상 테이블을 찾음.     
            - 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능.     
            - 값이 없으면 외래 키에 null 허용
        - 대상 테이블에 외래키            
            ![11_2](https://user-images.githubusercontent.com/59528611/218453488-1e4655ab-245f-43a1-a89a-8d3427a3e33b.jpeg)     
            ```java
            @Entity
            public class Member{
                @Id @GeneratedValue
                @Column(name = "MEMBER_ID")
                private Long id;

                @Column(name = "USERNAME")
                private String username;

                @OneToOne(mappedBy = "member")
                private Locker locker;
            }

            @Entity
            public class Locker{
                @Id @GeneratedValue
                @Column(name = "LOCKER_ID")
                private Long id;

                @Column(name = "LOCKERNAME")
                private String username;		

                @OneToOne
                @JoinColumn(name = "MEMBER_ID")
                private Member member;
            }
            ```
            > 주 테이블에 외래키가 있는 상황에서 연관관계 주인만 바꾸면 됨.     
            - 대상 테이블에 외래키가 존재       
            - 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지 가능        
            - 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨.     
- 다대다(N:M) 양방향 연관관계        
    - 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음.     
    - 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어야됨.        
    ![NM_1](https://user-images.githubusercontent.com/59528611/218476175-619d283f-29fd-4017-b812-e182700e64f3.jpeg)     
    - 객체는 컬렉션을 사용해서 객체 2개로 다대다 가능       
    ![NM_2](https://user-images.githubusercontent.com/59528611/218498094-8abeb410-c092-4942-818f-4e27c703a075.jpeg)     
    - @ManyToMany 사용, @JoinTable로 연결 테이블 지정.      
    - 편리해 보이지만 실무에서 사용X    
    - 다대다 한계 극복      
        - 연결 테이블용 엔티티 추가(연결 테이블을 엔티티로 승격함)      
        - @ManyToMany -> @OneToMany, @ManyToOne     
        ![NM_3](https://user-images.githubusercontent.com/59528611/218498676-dbe7f1d8-9804-421b-abc3-b076063643ee.jpeg)     

___         

## 상속관계     
- 상속관계 매핑     
    - 관계형 데이터베이스는 상속 관계 X     
    - 슈퍼타입 서브타입이라는 모델링 기법이 상속관계와 유사     
        - 테이블의 논리 모델, 물리 모델 구현        
            1. 조인 전략    
            ![join_st](https://user-images.githubusercontent.com/59528611/219011535-4c719b4b-c9d7-46a7-be16-4ea98f882cc9.jpeg)      
            2. 단일 테이블 전략     
            ![singleTable_st](https://user-images.githubusercontent.com/59528611/219011754-5c7183d8-a6a2-4d99-8d9f-45cedcf47477.jpeg)       
            3. 구현 클래스마다 테이블 전략      
            ![tablePer_st](https://user-images.githubusercontent.com/59528611/219011961-87eb5eb2-c1b3-403e-b9cf-bddc4b4506ba.jpeg)
            ```java     
            @Entity
            @Inheritance(strategy = InheritanceType.JOINED)//디폴트값은 SINGLE_TABLE
            @DiscriminatorColumn // 컬럼에 서브타입 타입용 컬럼 추가
            public class Item{
                @Id	@GeneratedValue
                private Long id;				

                private String name;
                private int price;
            }

            @Entity
            @DiscriminatorValue("Ab") // 슈퍼타입 테이블의 컬럼에 등록될 이름
            public class Album extends Item{
                private String artist;
            }

            @Entity
            public class Movie extends Item{
                private String director;
                private String actor;
            }

            @Entity
            public class Book extends Item{
                private String author;
                private String isbn;
            }
            ```     
    1. <b>조인 전략</b>    
        - 장점      
            - 테이블이 정규화 되어있음      
            - 외래키 참조 무결성 제약조건 활용 가능     
            - 저장공간 효율화됨
        - 단점      
            - 조회시 조인을 많이 사용함. 성능 저하 가능성     
            - <b>조회 쿼리가 복잡함</b>     
            - 데이터 저장시 isert sql이 여러번(상속 관계만큼) 호출      
    2. <b>단일 테이블 전략</b>     
        - 장점      
            - 조인이 필요 없음. 조회 성능 빠름      
            - 조회 쿼리가 단순함        
        - 단점      
            - 자식 엔티티가 매핑한 칼럼은 모두 null을 허용해야 함       
            - 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있음      
    3. 구현 클래스마다 테이블 전략      
    이 전략은 추천 X    
        - 장점      
            - 서브 타입을 명확하게 구분할 때 효과적     
        - 단점  
            - 여러 테이블을 조회시 성능이 느림(union)   
            - 자식 테이블을 통합해서 쿼리하기 어려움        
    - <b> 조인 전략과 단일 테이블 전략 둘 중 고민해서 사용하자</b>       

- MappedSuperClass      
공통 매핑 정보가 필요할 때 사용
```java     
    @MappedSuperclass
	public abstract class BaseEntity{
		private LocalDateTime createDate;
		private LocalDateTime lastModifiedDate;
	}

	@Entity
	public class Member extends BaseEntity{
		@Id @GeneratedValue
		private Long id;
	}

	@Entity
	public class Team extends BaseEntity{
		@Id @GeneratedValue
		private Long id;
	}
```     
- 상속 관계 X, 엔티티 X, 테이블과 매핑 X        
- 상속받는 자식 클래스에 매핑 정보만 제공       
- 조회, 검색 불가능     
- 직접 사용할 일이 없으므로 <b>추상 클래스 권장</b>     

___         

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