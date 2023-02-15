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

