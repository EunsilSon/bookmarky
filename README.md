# 북마키 (BookMarky)
![bookmarky](https://github.com/user-attachments/assets/46b23ae1-0d8a-457d-827d-d78602d265ea)

# 목차
1. [프로젝트 소개](#프로젝트-소개)
2. [기술 스택](#기술-스택)
3. [주요 기능](#주요-기능)
4. [구현 과정](#구현-과정)
5. [API 명세](#API-명세)
6. [ERD](#ERD)

<br>

# 프로젝트 소개
**북마키**는 독서 중 마음에 드는 구절을 편리하게 기록하고 쉽게 접근할 수 있는 독서 도우미 서비스입니다.

- 책을 읽다가 마음에 드는 구절을 표시하고 싶지만, 책에 줄을 긋기는 꺼려질때
- 메모장에 적어두기엔 번거롭고 내용이 많아 찾기 힘들때
- 다른 장소에서 독서하기 위해 노트, 필기구 등 챙기기 번거로울때

북마키는 이러한 불편을 해소해줍니다.

<br>

> **“BookMarky”의 의미**  
책(Book)에서 마음에 드는 부분을 표시(Mark)하여 기억하고 보관할 수 있다는 뜻을 담은 이름입니다.  
"Marky"는 표시하다(Mark)와 친근한 느낌을 주는 접미사(-y)를 결합해, 사용자가 쉽게 기억하고 친숙하게 느낄 수 있도록 했습니다.
> 

<br>

# 기술 스택
※ 현재 사용 중인 기술입니다.

| 구분 | 기술 | 설명 |
| --- | --- | --- |
| Back-end | Spring Boot 3 | MVC 패턴의 REST API 서버 |
|  | Spring Data JPA |  |
|  | Spring Security | 사용자 인증 |
|  | Spring AOP | 호출된 API, 수행 시간 파악 |
|  | Spring Cache | 저장한 책 개수 조회 |
|  | JavaMailSender | 이메일 생성 및 전송 |
|  | Redis | 토큰 발급 및 관리 |
|  | MySQL |  |
|  | TypeScript | [[Web] TypeScript 이슈 모음](https://velog.io/@eunsilson/Web-TypeScript-%EC%9D%B4%EC%8A%88) |
|  | Axios | API 통신 라이브러리  |

<br>

# 주요 기능
- **Spring JPA를 활용한 모든 비즈니스 로직 구현**
    - DTO와 VO를 활용하여 실제 Entity에 영향이 미치지 않도록 함
    - JPA 페이징 기능으로 서버 부하 감소
    - Entity를 Optional로 래핑하여 NPE 발생 가능성을 줄임
    - @Value를 이용한 환경 변수 사용
- **Spring Security 폼 로그인 사용자 인증**
    - SecurityContext의 인증된 Authentication에서 username을 추출해 비즈니스 로직에 사용
- **비밀번호 변경을 위해 이메일 링크 전송 및 Redis 토큰 관리**
    - Spring SMTP를 적용하고 Mail Service와 Token Service 구현
    - Token 유효성 검증 후 비밀번호 변경 프로세스 실행
- **Validation을 통한 데이터의 제약 조건 검증**
    - @RestControllerAdvice 클래스를 구현하고 예외 처리 메소드를 생성해 명확한 검증 실패 원인 반환
- **최근 삭제한 구절 조회 및 복구 기능**
    - 해당 엔티티에 @SQLDelete를 적용하여 Delete 발생 시, Update를 발생시켜 isDeleted 컬럼을 변경해 Soft Delete 구현
    - Hibernate 동적 필터링을 적용해 삭제된 데이터 조회
    - @Scheduled를 적용하여 삭제한지 30일 지난 데이터를 자동으로 영구 삭제
- **AOP를 적용한 로깅**
    - 호출된 API를 파악하기 위한 로깅 작업에서 실수와 변경이 잦았음
    - 반복되는 로깅을 LogAspect 클래스로 분리해 관리하고, 메서드 실행 시간을 로깅하여 병목 현상 파악에 도움
- **Spring 내부 캐시를 활용해 책 개수 현황 API 구현**
    - 책 개수가 변동 가능성은 낮지만 자주 조회돼 캐시에 저장하고자 함
    - DB 부담 최소화 및 성능 약 98% 향상 (174ms → 3ms 감소)

<br>

# 구현 과정
※ ‘비고’에 기재된 포스팅에서 구현 과정을 확인할 수 있습니다.

| 기능 | 설명 | 비고 |
| --- | --- | --- |
| 사용자 인증 | 로그인 | [[Spring Security] Form Login 인증](https://velog.io/@eunsilson/Security-Form-Login-%EC%9D%B8%EC%A6%9D-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0) |
|  |  | [[Spring Security] Form Login 성공/실패 Handler](https://velog.io/@eunsilson/Security-Form-Login-%EC%9D%B8%EC%A6%9D-Handler-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0) |
|  |    | [[Security] SecurityContext의 사용자 인증 정보 가져오기](https://velog.io/@eunsilson/Spring-SecurityContext%EC%9D%98-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%9D%B8%EC%A6%9D-%EC%A0%95%EB%B3%B4-%EA%B0%80%EC%A0%B8%EC%98%A4%EA%B8%B0) |
|  | 회원가입 | [[Spring Boot] Validation+@Valid를 적용한 값의 유효성 검증](https://velog.io/@eunsilson/Spring-Boot-Validation-%EC%9C%A0%ED%9A%A8%EC%84%B1-%EA%B2%80%EC%A6%9D) |
|  | 비밀번호 변경 | [[Spring Boot] 비밀번호 재설정 링크를 이메일로 전송 (Redis 토큰)](https://velog.io/@eunsilson/Spring-Boot-%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8-%EC%9E%AC%EC%84%A4%EC%A0%95-%EB%A7%81%ED%81%AC%EB%A5%BC-%EC%9D%B4%EB%A9%94%EC%9D%BC%EB%A1%9C-%EC%A0%84%EC%86%A1%ED%95%98%EA%B8%B0-Redis-%ED%86%A0%ED%81%B0) |  
| 책 | 책 검색 | [[Java] JSON, XML을 파싱해 원하는 값만 추출하는 방법](https://velog.io/@eunsilson/Java-JSON-XML%EC%9D%84-%ED%8C%8C%EC%8B%B1%ED%95%B4-%EC%9B%90%ED%95%98%EB%8A%94-%EA%B0%92%EB%A7%8C-%EC%B6%94%EC%B6%9C%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-Open-API-%EA%B2%B0%EA%B3%BC%EA%B0%92) |
|  |  저장한 책 조회  | [[Spring Boot] JPA 페이징 (Pageable 객체)](https://velog.io/@eunsilson/Spring-Boot-JPA-%ED%8E%98%EC%9D%B4%EC%A7%95-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-Pageable-%EA%B0%9D%EC%B2%B4) |  
|  |  책 상세 정보 조회  |  |  
|  |  책 삭제  |  | 
|  |  구절 생성  |  |
|  |  구절 수정  |  |
|  |  구절 삭제  | [[Spring Boot] Soft Delete와 Hibernate 필터링으로 삭제된 데이터 조회 (@Where 이슈 해결)](https://velog.io/@eunsilson/Spring-Boot-Soft-Delete%EC%99%80-Hibernate-%EB%8F%99%EC%A0%81-%ED%95%84%ED%84%B0%EB%A7%81-Where-%EC%9D%B4%EC%8A%88-%ED%95%B4%EA%B2%B0) |
|  |  구절 상세 조회  |  |
|  |  구절 목록 조회  |  |
|  |  삭제된 구절 목록 조회  |  |
|  |  호출된 API, 메서드 실행 시간 로깅 | [[Spring Boot] AOP 기능으로 어떤 API가 호출됐는지 로그 생성하기](https://velog.io/@eunsilson/Spring-Boot-AOP-%EA%B8%B0%EB%8A%A5%EC%9C%BC%EB%A1%9C-%EC%96%B4%EB%96%A4-API%EA%B0%80-%ED%98%B8%EC%B6%9C%EB%90%90%EB%8A%94%EC%A7%80-%EB%A1%9C%EA%B7%B8-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0)

<br>

# API 명세
<img width="70%" alt="api" src="https://github.com/user-attachments/assets/a02ab04f-c35e-4b97-9418-78e5d084f446">

<br>

# ERD
<img width="100%" alt="erd" src="https://github.com/user-attachments/assets/64bc4b16-64b9-4617-847e-715a28c2e8b0">
