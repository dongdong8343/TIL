## EC2 CPU 사용률 폭발

ec2에서 스프링부트 프로젝트를 클론해서 아래 명령어를 통해 테스트를 진행하고 있었다.

`./gradlew test`

하지만 테스트를 진행하는데 85%에서 도저히 오르지않고 먹통이 되는 사건이 발생했다.

그래서 검색해보니 인스턴스의 CPU 사용률이 높아서 중단 현상이 발생했다고 한다.

나도 CPU 사용률을 확인해봤다. 확인해보니 거의 100%를 찍었더라..

<center>
  <img
    src="https://github.com/user-attachments/assets/0123d241-8885-4532-941e-f4fd8e4ed41a"
    width="50%"
  />
</center>

## 메모리 스왑

이 문제를 해결하기 위해서는 메모리 스왑이라는 것을 적용해야한다.

메모리 스왑이라는 것은 RAM이 부족한 경우가 발생할 수 있는데 HDD의 일정공간을 RAM처럼 사용하는 것을 말한다.

방법은 아래와 같다.

1. dd 명령어를 통해 swap 메모리 할당

   `sudo dd if=/dev/zero of=/swapfile bs=128M count=16`

2. 스왑 파일에 대한 읽기 및 쓰기 권한 업데이트

   `sudo chmod 600 /swapfile`

3. Linux 스왑 영역 설정

   `sudo mkswap /swapfile`

4. 스왑 공간에 스왑 파일 추가해 스왑 파일 즉시 사용할 수 있도록 만든다.

   `sudo swapon /swapfile`

5. 절차가 성공했는지 확인

   `sudo swapon -s`

6. /etc/fstab 파일을 편집하여 부팅 시 스왑 파일을 활성화

   `sudo vi /etc/fstab`

   파일 끝에 아래 줄 추가하고 저장한 후 종료

   `/swapfile swap swap defaults 0 0`

   `:wq` 명령어를 통해 저장하고 종료 가능

7. free 명령어로 잘 적용됐는지 확인

<center>
  <img
    src="https://github.com/user-attachments/assets/77d32ef5-2f46-42e8-8630-1fd0a3d66d3e"
    width="50%"
  />
</center>

위 과정을 거치고 테스트를 다시 진행하기 정상적으로 동작했다.
