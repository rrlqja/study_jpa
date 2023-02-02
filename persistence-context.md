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