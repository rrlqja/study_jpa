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
    1. #### 조인 전략    
        - 장점      
            - 테이블이 정규화 되어있음      
            - 외래키 참조 무결성 제약조건 활용 가능     
            - 저장공간 효율화됨
        - 단점      
            - 조회시 조인을 많이 사용함. 성능 저하 가능성     
            - <b>조회 쿼리가 복잡함</b>     
            - 데이터 저장시 isert sql이 여러번(상속 관계만큼) 호출      
    2. #### 단일 테이블 전략     
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
    - <b> 조인 전략과 단일 테이블 둘 중 고민해서 사용하자</b>       

- MappedSuperClass      
공통 매핑 정보가 필요할 떄 사용
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