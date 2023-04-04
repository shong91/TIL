# 컴퓨터 공학 CS 지식 모음

- [프로그래밍 기본](./basic.md)
- [아키텍쳐 & 디자인 패턴](./architecture-design-pattern.md)
- [네트워크/스토리지](./network-storage.md)
- [데이터베이스](./database.md)
- [보안](./security.md)

### 유용한 링크 모음

1. 어떻게 준비하면 좋을까?

- [쩜튜브(유투브 영상)](https://www.youtube.com/channel/UCz6z9z5wrOd2aB090WwYc8Q)

2. CS 지식, 기술면접 질문

- [한재엽 님 github](https://github.com/JaeYeopHan/Interview_Question_for_Beginner)
- [망나니 개발자 님 tistory](https://mangkyu.tistory.com/91)
- [gyoogle 님 tech interview 블로그](https://gyoogle.dev/blog/)

3. [SW 직군 면접 사이트, 주니어 공부 방법 사이트 모음](https://garden1500.tistory.com/2)

| API URI         | HttpMethod | API 명      |
| --------------- | ---------- | ----------- |
| /event          | POST       | 이벤트 작성 |
| /event/{userId} | GET        | 포인트 조회 |

### 응답코드

| 코드명               | 응답코드 | 상세                                       |
| -------------------- | -------- | ------------------------------------------ |
| OK                   | 200      | 정상적으로 처리되었습니다.                 |
| REVIEW_NOT_FOUND     | 401      | 이미 삭제되었거나 존재하지 않는 리뷰입니다 |
| USER_NOT_FOUND       | 402      | 존재하지 않는 사용자입니다                 |
| PLACE_NOT_FOUND      | 403      | 존재하지 않는 장소입니다                   |
| NOT_ENOUGH_POINT     | 404      | 포인트가 부족합니다                        |
| OBJECT_ALREADY_EXIST | 500      | 이미 존재하는 데이터입니다                 |
| ERROR                | 500      | 수행 중 오류가 발생하였습니다.             |
