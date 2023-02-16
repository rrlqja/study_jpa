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
    - 프록시 인스턴스 초기화 여부 확인: PersistenceUnitUtill.isLoaded(entity)       
    - 프록시 클래스 확인: entity.getClass()     
    - 프록시 강제 초기화: Hibernate.initialize(entity)      
