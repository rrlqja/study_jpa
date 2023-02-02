## 엔티티 매핑  
객체와 테이블 매핑    
- 객체와 테이블 매핑: @Entity, @Table   
    1. @Entity: @Entity가 붙은 클래스는 jpa가 관리하는 엔티티라고 한다. 
    2. jpa를 사용해서 테이블과 매핑할 클래스는 @Entity 필수.    
    3. 기본 생성자 필수(public 또는 protected)  
    4. final, enum, interface, inner 클래스 X    
    5. 저장할 필드에 final 사용 X   
    ```java
    @Entity
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
    - @Id   
    - @GeneratedValue: 기본키 생성을 데이터베이스에 위임.   
        1. Identity: 데이터베이스에 위임. Mysql     
        2. Sequence: 데이터베이스 시퀀스 오브젝트 사용. Oracle      
            @SequenceGenerator 필요.    
        3. Table: 키 생성용 테이블 사용. 모든 DB에서 사용.  
            @TableGenerator 필요.   
        4. Auto: 데이터베이스 방언에 따라 자동 지정. 기본값     
        ```java
        @Entity
        public class ExEntity{
            @Id
            @GeneratedValue(strategy = GenerationType.Auto)
            private Long id;
        }
        ```
- 연관관계 매핑: @ManyToOne, @JoinColumn    