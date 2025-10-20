Github Actions는 일종의 로직을 수행하는 컴퓨터다.

빌드, 테스트, 배포에 대한 우리가 작성한 시나리오대로 수행하게된다.

1. 개발자가 commit하고 github에 push
2. Push를 감지한 Github가 github actions에 작성한 로직을 실행시킨다.
3. 배포(EC2에) 완료 후 서버 재실행
