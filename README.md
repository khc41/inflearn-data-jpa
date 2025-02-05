# inflearn-data-jpa

## 커리큘럼 
### 섹션 1. 스프링 데이터 JPA 소개
- 소개
- 강의 자료
---
### 섹션 2. 프로젝트 환경설정
- 프로젝트 생성
- 라이브러리 살펴보기
- H2 데이터베이스 설치
- 스프링 데이터 JPA와 DB 설정, 동작확인
---
### 섹션 3. 예제 도메인 모델
- 예제 도메인 모델과 동작확인
---
### 섹션 4. 공통 인터페이스 기능
- 순수 JPA 기반 리포지토리 만들기
- 공통 인터페이스 설정
- 공통 인터페이스 적용
- 공통 인터페이스 분석
---
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
  - JPA Hint
    - 진짜 성능 최적화가 중요한 것에만 넣는다. (성능 테스트 해보고 넣는 것 권장)
    - 진짜 조회 성능이 낮으면 캐시를 사용해야 한다.
      ```java
        @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
        Member findReadOnlyByUsername(String username);
      ```
  - Lock
    - 실시간 트래픽이 많은 서비스에서는 Lock을 걸면 안된다.
    - 걸려면 Optimistic Lock이나 다른 방법으로 해결하는 것을 추천
---
### 섹션 6. 확장 기능
- 사용자 정의 리포지토리 구현
  - repository 인터페이스 이름 + Impl 만 맞춰주면 Spring Data Jpa 가 인식해서 스프링 빈으로 등록
  - 실무에서는 QueryDSL 이나 Jdbc Template 사용할 때 많이 사용
  - 항상 사용하는 것은 아님
    - 핵심 비즈니스 로직과 화면에 맞춘 쿼리를 분리하는 편이 좋다.
- Auditing
  - 순수 JPA
    ```java
      @MappedSuperclass
      public class JpaBaseEntity {
  
        @Column(updatable = false)
        private LocalDateTime createdDate;
        private LocalDateTime updatedDate;
  
        @PrePersist
        public void prePersist() {
            LocalDateTime now = LocalDateTime.now();
            createdDate = now;
            updatedDate = now;
        }
  
        @PreUpdate
        public void preUpdate() {
            updatedDate = LocalDateTime.now();
        }
      }
    ```
  - 스프링 데이터 JPA
    ```java
      @EntityListeners(AuditingEntityListener.class)
      @Getter
      @MappedSuperclass
      public class BaseEntity {

          @CreatedDate
          @Column(updatable = false)
          private LocalDateTime createdDate;
          @LastModifiedDate
          private LocalDateTime lastModifiedDate;

          @CreatedBy
          @Column(updatable = false)
          private String createdBy;

          @LastModifiedBy
          private String lastModifiedBy;
      }
    ```
      ```java
        @Bean
        public AuditorAware<String> auditorProvider() {
          //SecurityContextHolder 등 에서 세션 정보 가져와서 사용
          //createdBy, lastModifiedBy 정보를 채워줌
          return () -> Optional.of(UUID.randomUUID().toString());
        }
      ```
    - Date 관련 Column의 경우는 모든 테이블에서 사용하지만, By 관련 Column은 사용 안하는 경우도 있기에 아래와 같이 분리해서 사용 권장
    ```java
      @EntityListeners(AuditingEntityListener.class)
      @Getter
      @MappedSuperclass
      public class BaseTimeEntity {
        @CreatedDate
        @Column(updatable = false)
        private LocalDateTime createdDate;
        @LastModifiedDate
        private LocalDateTime lastModifiedDate;
      }
    
      @EntityListeners(AuditingEntityListener.class)
      @Getter
      @MappedSuperclass
      public class BaseEntity extends BaseTimeEntity {

        @CreatedBy
        @Column(updatable = false)
        private String createdBy;

        @LastModifiedBy
        private String lastModifiedBy;
      }
    ```
- Web 확장 - 도메인 클래스 컨버터
  - 간단한 경우 조회용으로만 사용
  ```java
    @GetMapping("/members2/{id}")
    public String findMember2(@PathVariable("id") Member member) {
      return member.getUsername();
    }
  ```
- Web 확장 - 페이징과 정렬
  ```java
    @GetMapping("/members")
    public Page<MemberDto> list(@PageableDefault(size = 5, sort = "username") Pageable pageable) {
      // entity 노출하지 않게 page.map을 사용해 dto로 반환
      return memberRepository.findAll(pageable)
          .map(m ->new MemberDto(m.getId(), m.getUsername(), null));
    }
  ```
  - /members?page=1&size=3&sort=id,desc&sort=username,desc
    - 위와 같이 요청을 보내면 자동으로 페이징 처리 가능
  - default page setting 은 application.yml 파일에 아래와 같이 넣거나 @PageableDefault 어노테이션 사용
    ```yaml
      spring:
        data:
          web:
            pageable:
              default-page-size: 10
              max-page-size: 2000
    ```
  - Page를 1부터 시작하려면?
    - 별도의 Paging 클래스 정의 후 사용
    - `spring.data.web.pageable.one-indexed-parameters=true` 사용 
      - Page 객체 내 파라미터가 안맞아서 권장 x

---
### 섹션 7. 스프링 데이터 JPA 분석
- 스프링 데이터 JPA 구현체 분석
  - @Repository
    - JPA 예외를 스프링이 추상화한 예외로 변환
  - @Transactional은 Spring Data Jpa 내부에서 자동으로 적용
  - @Transactional(readOnly = true) 는 내부적으로 flush를 생략해서 약간의 성능 향상 가능
  - save() 메서드
    - 새로운 entity면 persist, 새로운 entity가 아니면 merge 
    - merge는 값이 전부 대체되기 때문에 되도록이면 dirty checking 사용해서 update
- 새로운 엔티티를 구별하는 방법
  - id가 GeneratedValue가 아닌 경우 내부 코드 상 isNew()는 false로 되어 merge() 동작
    - select 실행 후 update나 insert 실행
    - Persistable을 상속 받아 해결 가능
      - isNew() 메소드를 return createdDate == null 로 반환하면 새로운 엔티티인지 판단 가능
---
### 섹션 8. 나머지 기능들
- Specifications (명세)
  - JPA Criteria를 사용하므로 실무 사용 x
- Query By Example
  - Example 만들어서 select 할때 파라미터로 사용 (객체로 탐색)
  - primitive 타입인 경우 ExampleMatcher의 withIgnorePaths 사용해서 무시해줘야함
  - outer join 사용 안되는 이슈가 있어 실무 사용 고려해봐야함 
- Projections
  - 프로젝션 대상이 root 엔티티면 JPQA select문 최적화 가능 
  - root가 아닌 경우 모든 필드를 select 하므로 최적화가 안된다.
  - dto를 사용하는 경우 파라미터 이름 인식문제로 gradle로 빌드해야 한다.
- 네이티브 쿼리
  - 가급적 사용하지 않는게 좋음
    - dto로 반환하기 애매
    - JPQL 처럼 로딩 시점에 문법 확인 불가
    - 동적쿼리 불가
  - Projections을 이용하면 제약을 어느정도 해결할 수 있다.