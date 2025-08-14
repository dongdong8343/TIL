구글 인증 플랫폼에서 승인된 도메인을 입력하는데 EC2의 퍼블릭 DNS를 입력해줬다.

그랬더니 아래 사진처럼 잘못된 도메인이라고 뜨는 것이다.

<center>
    <img
            src="https://github.com/user-attachments/assets/8ba4f99b-018d-4a77-ba00-69ce2a7d98a4"
            width="50%"
    />
</center>

이유를 알아보니 내 소유 도메인이 아니라서 요건을 충족할 수 없다는 것이였다.

그래서 이 참에 도메인을 하나 구매해서 EC2 서버와 연결시키는 작업을 해봤다.

순서는 아래와 같다.

1. Route 53의 호스팅 영역에서 호스팅 영역 생성을 클릭

<center>
    <img
            src="https://github.com/user-attachments/assets/4b007e8b-d2e3-4205-9e67-355edf42646f"
            width="50%"
    />
</center>

2. 아래와 같이 입력을 하는데 이 때 도메인 이름은 구매한 도메인을 입력해야한다.

- 나는 도메인을 가비아에서 구매함

<center>
    <img
            src="https://github.com/user-attachments/assets/53c87ea8-dde2-4295-8296-6619bff1fef7"
            width="50%"
    />
</center>

3. 레코드 생성 버튼을 클릭

<center>
    <img
            src="https://github.com/user-attachments/assets/9aaf8080-27a5-464e-96ec-16605f9ca807"
            width="50%"
    />
</center>

4. 연결할 EC2의 ip 주소를 입력

<center>
    <img
            src="https://github.com/user-attachments/assets/79d28715-884f-4288-b67d-f479411f8ba0"
            width="50%"
    />
</center>

- 가비아 연결을 위해 아래 4개의 값을 사용할 예정이다.

<center>
    <img
            src="https://github.com/user-attachments/assets/8bdb6935-3f63-4a5e-b036-ea8d7181d030"
            width="50%"
    />
</center>

5. 가비아로 이동 > 내 도메인 > 도메인 정보 변경 > 네임 서버 클릭

<center>
    <img
            src="https://github.com/user-attachments/assets/2dabe1ba-ba42-4773-b890-a3336aefc34b"
            width="50%"
    />
</center>

6. 창 안에 아까 4가지 주소를 입력한다.

- '.'은 빼고 입력할 것!

<center>
    <img
            src="https://github.com/user-attachments/assets/b435ac65-0167-43bb-a648-042540d166d2"
            width="50%"
    />
</center>

<center>
    <img
            src="https://github.com/user-attachments/assets/f7675098-68cb-4068-b01d-e1b0846b3fdc"
            width="50%"
    />
</center>

이렇게 연결을 하고 시간이 지나고 확인해보면 제대로 연결이 된 것을 확인할 수 있다. 로그인도 잘 된다^^

<center>
    <img
            src="https://github.com/user-attachments/assets/b654cba8-f982-45c7-ae74-c629f6dc73ca"
            width="50%"
    />
</center>
