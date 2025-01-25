# inflearn-data-jpa

## 커리큘럼 
### 섹션 1. 스프링 데이터 JPA 소개
- 소개
- 강의 자료

### 섹션 2. 프로젝트 환경설정
- 프로젝트 생성
- 라이브러리 살펴보기
- H2 데이터베이스 설치
- 스프링 데이터 JPA와 DB 설정, 동작확인

### 섹션 3. 예제 도메인 모델
- 예제 도메인 모델과 동작확인

### 섹션 4. 공통 인터페이스 기능
- 순수 JPA 기반 리포지토리 만들기
- 공통 인터페이스 설정
- 공통 인터페이스 적용
- 공통 인터페이스 분석

### 섹션 5. 쿼리 메소드 기능
- 메소드 이름으로 쿼리 생성
- JPA NamedQuery
- @Query, 리포지토리 메소드에 쿼리 정의하기
- @Query, 값, DTO 조회하기
- 파라미터 바인딩
- 반환 타입
- 순수 JPA 페이징과 정렬
  ```java
  List<Member> findListByUsername(String username); // 컬렉션
  Member findMemberByUsername(String username); // 단건
  Optional<Member> findOptionalByUsername(String username); // 단건 Optional
  ```
- 스프링 데이터 JPA 페이징과 정렬
  - 슬라이스 (limit + 1 가져와서 앱에서 간단한 페이징 처리)
  - count query 분리 (join 등도 같이 Total count 쿼리에 걸리므로 분리)
     ```java 
     @Query(value = "select m from Member m left join m.team t",
        countQuery = "select count(m.username) from Member m") 
     ```
  - page.map() 으로 dto 변환
     ```java
       Page<MemberDto> map = page.map(m -> new MemberDto(m.getId(), m.getUsername(), null));
     ```
- 벌크성 수정 쿼리
  - @Modifying로 update문 명시
  - 벌크 연산 실행 후 다른 조회가 있으면 clearAutomatically = true 옵션으로 영속성 컨텍스트 클리어
    ```java
      @Modifying(clearAutomatically = true)
      @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
      int bulkAgePlus(@Param("age") int age);
    ```
- @EntityGraph
  - 간단한 fetch join의 경우 사용 (복잡할 땐 jpql fetch join 사용)
  ```java
    @EntityGraph(attributePaths = {"team"})
    @Query("select m from Member m")
    List<Member> findMemberEntityGraph();
  ```
- JPA Hint & Lock

### 섹션 6. 확장 기능
- 사용자 정의 리포지토리 구현
- Auditing
- Web 확장 - 도메인 클래스 컨버터
- Web 확장 - 페이징과 정렬

### 섹션 7. 스프링 데이터 JPA 분석
- 스프링 데이터 JPA 구현체 분석
- 새로운 엔티티를 구별하는 방법

### 섹션 8. 나머지 기능들
- Specifications (명세)
- Query By Example
- Projections
- 네이티브 쿼리