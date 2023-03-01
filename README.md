### [jpa](https://github.com/rrlqja/study_jpa/blob/master/jpa.md)     
- [영속성 컨텍스트(Persistence Context)](#영속성-컨텍스트-persistence-context) [원문](https://github.com/rrlqja/study_jpa/blob/master/persistence-context.md)       
- [영속성 컨텍스트(Persistence Context)](https://github.com/rrlqja/study_jpa/blob/master/persistence-context.md)   
- [영속 컨](#영속성-컨텍스트-persistence-context)
- [엔티티 매핑](https://github.com/rrlqja/study_jpa/blob/master/entity-mapping.md)   
- [연관관계 매핑 기초](https://github.com/rrlqja/study_jpa/blob/master/relational-mapping-basic.md)    
- [다양한 연관관계 매핑](https://github.com/rrlqja/study_jpa/blob/master/relational-mapping.md)
- [연관관계 매핑 고급](https://github.com/rrlqja/study_jpa/blob/master/relational-mapping-advanced.md)    
- [프록시](https://github.com/rrlqja/study_jpa/blob/master/proxy.md)    

___     
## 영속성 컨텍스트 (Persistence Context)    
- 엔티티(entity)를 영구 저장하는 환경   
- 눈에 보이지 않는 논리적인 개념    
- 엔티티 매니저(entity manager)를 통해 영속성 컨텍스트에 접근   

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

## 양방향 매핑 정리     
- 단방향 매핑만으로 연관관계 매핑을 완료해라    
- 양방향 매핑은 조회(객체 그래프 탐색) 기능이 추가되는 것 뿐
- 단방향 매핑을 잘 하고 양방향 매핑은 필요할 때 추가해라        
