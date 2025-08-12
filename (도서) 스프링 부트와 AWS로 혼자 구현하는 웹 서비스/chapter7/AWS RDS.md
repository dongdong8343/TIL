## AWS RDS

- aws에서 제공하는 클라우드 기반 관계형 데이터베이스

### **RDS 인스턴스 생성 순서**

1. RDS에 가서 데이터 베이스 생성 클릭
<center>
  <img
    src="https://github.com/user-attachments/assets/2f411df3-c09b-48ed-ad85-96547b377eae"
    width="50%"
  />
</center>

2. DB 선택
<center>
  <img
    src="https://github.com/user-attachments/assets/6493a218-bc53-4bb9-a917-63d8d8679a7e"
    width="50%"
  />
</center>

3. 템플릿 - 프리티어 선택
<center>
  <img
    src="https://github.com/user-attachments/assets/2a10bad0-2273-4399-916d-2146805fc914"
    width="50%"
  />
</center>

4. DB 인스턴스, 마스터 사용자 정보 등록
<center>
  <img
    src="https://github.com/user-attachments/assets/fca7903e-a150-41fe-a945-98697408cbb4"
    width="50%"
  />
</center>

5. 마지막으로 내려서 DB 생성 버튼 클릭

### RDS 운영환경에 맞는 파라미터 설정

- 타임존
- Character Set
- Max Connection

1. 파라미터 탭 클릭
<center>
  <img
    src="https://github.com/user-attachments/assets/f63652e5-de2a-4659-8cea-67fe3bdc4e8f"
    width="50%"
  />
</center>

2. 파라미터 그룹 생성 버튼 클릭
<center>
  <img
    src="https://github.com/user-attachments/assets/c548b9fd-f5da-4174-9bd7-7e89fb6ee670"
    width="50%"
  />
</center>

3. 파라미터 그룹 패밀리는 DB 생성할 때 버전과 맞춰야 함.
<center>
  <img
    src="https://github.com/user-attachments/assets/c15b22cf-4a02-47e9-a889-925660f05ae5"
    width="50%"
  />
</center>

4. 생성된 그룹 클릭
<center>
  <img
    src="https://github.com/user-attachments/assets/5f47752e-8f3d-49f5-9638-5ea9d6546078"
    width="50%"
  />
</center>

5. 편집 버튼 클릭
<center>
  <img
    src="https://github.com/user-attachments/assets/e315bae3-9d3a-40f5-9663-b47f02d5b2fd"
    width="50%"
  />
</center>

6. time_zone 검색 후 Asia/Seoul 선택
7. character 항목들 utf8mb4로 변경 - 이모지 저장 가능

   collation 항목들은 utf8mb4_general_ci로 변경

<center>
  <img
    src="https://github.com/user-attachments/assets/a441d4e2-09a4-4b89-a3df-821885379fee"
    width="50%"
  />
</center>

8. max_connections 150으로 설정
<center>
  <img
    src="https://github.com/user-attachments/assets/59a14a28-cb14-40d4-8784-5971a82e8c6b"
    width="50%"
  />
</center>

9. 변경 사항 저장 클릭
10. 생성된 파라미터 그룹을 DB에 연결
<center>
  <img
    src="https://github.com/user-attachments/assets/49b66d30-b2f9-4ff2-853b-26bb5aa9935c"
    width="50%"
  />
</center>

11. 파라미터 그룹 변경 후 저장 클릭
<center>
  <img
    src="https://github.com/user-attachments/assets/09170d8c-a695-45ee-bfa7-0a5ab62bab41"
    width="50%"
  />
</center>

12. 제대로 반영되지 않는 경우가 있어서 재부팅 진행
<center>
  <img
    src="https://github.com/user-attachments/assets/96525c92-3849-4383-8924-3c965469b387"
    width="50%"
  />
</center>

### PC에서 RDS 접속해보기

1. RDS의 보안그룹에 본인 PC의 IP 추가 및 ec2에서 접근 가능하도록 ec2의 보안 ID 선택 (아래 사진 참고)
<center>
  <img
    src="https://github.com/user-attachments/assets/b095f360-f7a5-4f4b-a787-e39341093abc"
    width="50%"
  />
</center>

2. 인텔리제이에서 DB 연결해보기
3. character_set, collation 설정 확인

   `*show variables like* 'c%'`

4. 타임존 확인

   `*select* @@time_zone, now()`

5. 한글 데이터 잘 등록되는지 아래 쿼리를 통해 확인

```sql
CREATE table test(
    id      bigint(20) NOT NULL AUTO_INCREMENT,
    content varchar(255) DEFAULT NULL,
    PRIMARY KEY (id)
) ENGINE=InnoDB;

insert into test(content) values ('테스트')

select * from test
```

### EC2에서 RDS 접근 확인

1. ssh 접속 - putty
2. Mysql cli 설치
   - RPM 파일 다운로드
     `sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm`
   - GPG 퍼블릭 키 설정
     `sudo dnf install mysql80-community-release-el9-1.noarch.rpm -y`
   - mysql 설치를 위해서 퍼블릭키를 import 해야함
     `sudo dnf update -y`
   - mysql 설치
     `sudo dnf install mysql-community-client -y`
     `sudo dnf install mysql-community-server -y`
3. 설치 후 RDS 접속

   `mysql -u 계정 -p -h Host주소`

4. show databases 명령어 입력 후 생성한 RDS가 맞는지 확인
<center>
  <img
    src="https://github.com/user-attachments/assets/5a9f86c6-4378-448f-bf89-20cc47014e79"
    width="50%"
  />
</center>
