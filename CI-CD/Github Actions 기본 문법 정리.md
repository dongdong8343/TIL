### 폴더 구조

Github Actions를 사용하기 위해서는 프로젝트 내부에 아래와 같이(꼭!!) 폴더 구조를 가져야한다.

.github - workflows - ~~.yml 파일

<center>
  <img
    src="https://github.com/user-attachments/assets/8c2a112b-7ad8-4b31-8e5e-21902696a724"
    width="50%"
  />
</center>

### ~~.yml 파일 내부 구성

이 문서 자체가 하나의 단위.

즉 workflow다.

```yaml
# workflow의 이름
name: Github Actions 연습

# Event: 어느 시점에 이 로직을 실행시킬 것인가?
# 아래는 main 브랜치에 push가 발생하면 실행시키겠다는 의미
on:
  push:
    branches:
      - main

# push가 발생하면 아래 로직을 실행시킬게요~
# 하나의 workflow는 1개 이상의 Job으로 구성된다. 여러 job으로 구성되면 병렬적으로 수행된다.
jobs:
  # Job을 식별하기 위한 id - 내 마음대로 작성 가능
  My-Deploy-job:
    # ubuntu 환경 중 최신 버전을 선택 (운영체제 고름)
    runs-on: ubuntu-latest

    # Step : 특정 작업을 수행하는 가장 작은 단위
    # Job은 여러 Step들로 구성됨
    steps:
      - name: Hello world 찍기
        run: echo "Hello World"

      # step 추가!
      - name: step 더 찍어보자
        run: |
          echo "꼭"
          echo "성공하고싶다."

      - name: Github Actions 자체에 저장된 변수 사용해보기
        # $GITHUB_SHA - 지금 해당하는 commit의 id
        run: |
          echo $GITHUB_SHA
          echo $GITHUB_REPOSITORY

      # Secret 값 같은 것은 막 외부에 노출되거나 찍히면 안됨.
      - name: 아무한테 노출이 안되는 값
        run: |
          echo $ {{ secrets.MY_NAME }}
          echo $ {{ secrets.MY_HOBBY }}
```

추가로 Secret 값 같은 것은 막 외부에 노출되거나 찍히면 안된다.

그렇기 때문에 Secret 값은 github 내부에 저장해놓고 사용할 수 있고 내부에 저장하면 외부에서도 볼 수 없고 초대된 사람도 볼 수 없게된다.

1. github - settings 클릭

<center>
  <img
    src="https://github.com/user-attachments/assets/85fb6e2b-31fc-4f6c-8efc-51bc807842da"
    width="50%"
  />
</center>

2. Secret 값을 등록

<center>
  <img
    src="https://github.com/user-attachments/assets/3cf58cd6-7457-4d47-8650-d1560ca2be79"
    width="50%"
  />
</center>

<center>
  <img
    src="https://github.com/user-attachments/assets/c8942238-89e9-4d0a-81ad-1b589c98447a"
    width="50%"
  />
</center>

위와 같이 등록하면 초대된 사람도 볼 수 없게 된다.

<center>
  <img
    src="https://github.com/user-attachments/assets/538b6e08-dd85-42fe-8d91-8eac03d4620b"
    width="50%"
  />
</center>

3. push가 발생하면 secret 값을 출력하라는 명령이 있었으니까 확인을 해보면 잘 가려져서 나오는 것을 확인할 수 있다.

<center>
  <img
    src="https://github.com/user-attachments/assets/bafef06e-2e34-4dac-8fe4-49ac2f3e1d8e"
    width="50%"
  />
</center>
