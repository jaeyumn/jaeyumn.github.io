---
layout: single
title: "Port 8080 was already in use"
---

작업을 하다가 인텔리제이가 갑자기 닫혀버리거나 서버를 종료하지 못한 경우에 다시 서버를 실행시키면 에러가 발생한다.

아마 비정상적으로 종료되면 포트가 닫히지 않고 다시 동일한 포트로 접근하면 '이미 사용중인 포트임' 에러가 발생하는 것 같다.

자주 발생하는 경우라 블로그에 정리해두고 보려고 한다.

![Alt text](image-1.png)


<br>

### 해결 방법 (Windows OS)

1. 관리자 권한으로 cmd 실행
2. netstat -p tcp -ano 를 입력하면 아래와 같이 8080 port가 사용중이라고 나온다.
![Alt text](image.png)


3. 위의 PID를 참고하여 taskkill /f /pid {PID} 값을 입력하면 된다.
![Alt text](image-2.png)

<br>

### 추가
EC2 환경에서 백그라운드로 서버를 실행시켰을 때 종료 방법 (bash)

1. 프로세스 id 확인: sudo lsof -t -i:[포트 번호]
2. 프로세스 종료 : kill -9 [프로세스 id]