vs code를 통해 깃허브에 TIL을 작성하는데 commit과 push하는 방법을 까먹어서 기억하기위해 작성합니다.

### commit, push 순서

1. 터미널을 연다.
2. git add .
   - add 명령어는 commit을 하기 전에 준비 완료 상태로 보내기 위함입니다.
   - . 을 붙이면 변경 사항 있는 모든 파일을 commit 대상으로 하겠다는 뜻입니다. 특정 파일만 하고싶다면 특정 파일 이름만 작성하면 됩니다.
3. git commit -m "커밋 메시지"
4. git push

위 순서대로 진행하면 깃허브에 push를 할 수 있습니다.
