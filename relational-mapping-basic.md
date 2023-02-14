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