# Jenkins를 이용해서 배포하기

- 젠킨스의 기능
    - build & deploy
    - batch
        - 실시간이 아니라 모아서 또는 주기적으로 처리하는 것
- 이전에 손수 빌드, 배포했던 작업을 젠킨스로 자동화 한다.

## 인스턴스 생성

- 젠킨스 인스턴스 or 젠킨스
    - 젠킨스가 실행되고 있는 인스턴스
    - 하나만 생성할 예정
- 워커 인스턴스 or 워커
    - 배포 대상이 되는 인스턴스
    - 개발한 애플리케이션을 실행시키는 인스턴스

```commandline
# jenkins 인스턴스에서 실행하는 명령어 (한 줄씩 실행하면서 정상적으로 실행이 되고 있는지 꼭 확인해보세요)
sudo yum install wget
sudo yum install maven
sudo yum install git
sudo yum install docker

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
# 여기까지 실행하면 설치는 완료
```

![](../../.gitbook/assets/backend-system-practice/01/screenshot%202022-04-03%20오후%208.01.13.png)

- 웹 브라우저로 젠킨스에 접속할 것이다.
- 근데 젠킨스는 8080 포트로 떠있다.
- 우리가 만든 인스턴스는 80 포트를 사용하는데 젠킨스를 80 포트로 바꾸는 건 까다롭다.
- 방화벽 규칙 설정에서 8080 포트를 허용하는 규칙을 추가한다.
    - 0.0.0.0/0
        - 모든 ip 요청을 허용한다.
    - tcp:8080
        - 8080 포트를 허용한다.

## 젠킨스 설정

```text
http://00.00.000.00:8080/
```

- 인스턴스 ip + 8080 포트로 접속하면 젠킨스 설정 화면이 뜬다.

```commandline
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

1. 해당 명령어로 파일을 읽어 비밀번호를 알아낸다.
2. 추천 플러그인 버튼을 클릭한다.
3. 어드민 계정으로 사용할 정보를 입력한다.

## 플러그인 설치

- 플러그인 관리에서 Publish Over SSH를 재시작 없이 설치한다.

## 젠킨스로 배포

- 배포에는 여러 방법이 있지만 SSH를 통해 배포하는 것이 보편적이다.
- 젠킨스가 워커로 접속해 도커 이미지를 풀 받고 런 시키는 것이다.
- 이때, 젠킨스의 워커만 SSH로 접속할 수 있게 만들어야 한다.
    - 해커 등 원하지 않는 접속이 있을 위험이 있기 때문이다.
- 워커가 젠킨스 인스턴스라고 증명할 수 있는 SSH 연결만 허용하도록 만든다.

### 대칭키와 비대칭키

- 대칭 키
    - 암호화와 복호화에 같은 키 사용
- 비대칭 키
    - 암호화와 복호화에 다른 키 사용
    - 둘 사이에 관계는 있지만 키가 같지는 않다는 뜻이다.
- 공개 키로 암호화한 것은 개인 키로 복호화 할 수 있다.
- 개인 키로 암호화한 것은 공개 키로 복호화 할 수 있다.
    - 서명이라는 개념
    - A의 공개 키로 서명할 수 있는 건 A 뿐이다.
    - A의 공개 키로 복호화할 수 있는 메시지를 만드는 건 A 뿐이다.

## 젠킨스 인스턴스에서 개인 키와 공개 키 생성

```commandline
ssh-keygen -t rsa -f ~/.ssh/id_rsa
```

- 젠킨스에서 젠킨스의 개인키와 공개키 쌍을 만든다.
    - Enter passphrase는 계속 엔티를 누른다.

```commandline
cd .ssh
ls
```

```text
id_rsa  id_rsa.pub
```

- 개인 키와 공개 키 쌍이 생성된 걸 확인한다.
    - pub가 공개키를 나타낸다.

## 공개 키 등록

```commandline
vi id_rsa.pub
```

- 공개 키 내용을 복사한다.
- GCP - 메타데이터 - SSH 키에 등록한다.
    - 공개 키가 워커에게 등록된다.

## 워커 내부 폴더 권한 변경

```commandline
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

- 권한을 변경해준다.

## 젠킨스만 워커에 접속할 수 있게 설정

- 젠킨스 관리 - 시스템 설정

### Publish over SSH

```commandline
vi ~/.ssh/id_rsa
```

- key
    - 젠킨스의 개인 키를 입력한다.

### SSH Servers

- Name
    - 서버를 다양하게 연결 할 수 있기 때문에 구분할 수 있는 이름으로 짓는다.
- HostName
    - 젠킨스와 워커는 같은 네트워크 상에 있기 때문에 내부 IP를 입력한다.
- UserName
    - 젠킨스 계정 userName를 넣는다.
    - 젠킨스 인스턴스에 설치된 계정과 맞지 않으면 연결 실패한다.
- remote directory
    - 이번 프로젝트에서는 홈 디렉토리를 사용한다.
    - pwd 커맨드의 결과를 넣는다.

## 배포 스크립트 작성

1. 새로운 Item을 만든다.
    - item name
        - cpu-worker-instance-1 deploy
    - Freestyle project
2. 빌드 후 조치
3. Send build artifacts over SSH
4. SSH Server에 워커 인스턴스를 연결한다.
5. 고급 - Verbose output in console을 체크한다.
    - 로그를 자세히 출력해준다.

```commandline
sudo docker run -p 80:80 user-name/spring-boot-cpu-bound
```

6. Transfers - Exec command
    - 도커 run 명령어를 입력한다.

## SSH로 접속해 docker run 명령어 실행

앞서 했던 설정으로 build를 해보면 실패한다. sudo를 사용할 수 없다고 나올 것이다.

```commandline
docker run -p 8080:80 user-name/spring-boot-cpu-bound
```

- 이전에 sudo를 붙였던 이유는 80번 포트를 사용하기 위해서였다.
    - 8080으로 포트를 변경하면 sudo가 필요없다.
- 어차피 나중에 nginx가 앞에서 요청을 받게 되는데, nginx만 80번 포트이면 되므로 문제가 없다.

다시 해보면 이번엔 docker: command not found 에러가 뜬다.

```commandline
sudo yum install docker
sudo systemctl start docker
```

- 워크 인스턴스에 docker를 설치하고 docker 데몬을 실행한다.

```commandline
/usr/bin/docker-current: Got permission denied while trying to connect to the Docker daemon socket 
at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.26/containers/create: 
dial unix /var/run/docker.sock: connect: permission denied.
```

```commandline
sudo chmod 666 /var/run/docker.sock
```

- 권한 에러가 뜨면 docker.sock의 권한을 변경한다.

이제 애플리케이션이 잘 뜬다. 문제는 로딩 중인 로그 때문에 젠킨스가 배포가 끝나지 않았다고 인식해 빨간색이 된다.

```commandline
nohup docker run -p 8080:80 redthink90/spring-boot-cpu-bound > /dev/null 2>&1 &
```

- 구성 - SSH Server - Transfers - Exec command의 명령어를 수정한다.

```commandline
nohup ... &
```

- 명령을 백그라운드로 실행시키겠다.

```commandline
> /dev/null 2>&1
```

- 표준 에러를 표준 출력으로 리다이렉션 하겠다.

## 확인

```text
http://{워커의 외부 ip}:8080/hash/123
```

- 워커의 외부 IP로 접속해서 응답이 나오는지 본다.

## 정리

1. 젠킨스와 워커의 인스턴스를 생성한다.
2. 젠킨스의 개인 키, 공개 키 쌍을 만든다.
3. 공개 키를 워커에 등록한다.
4. 젠킨스에서 워커 인스턴스로 배포하도록 설정한다.
5. 워커 인스턴스에서 도커 이미지를 pull 받고 run 하게 한다.
    - 이 과정에서 로그를 보며 삽질한다.