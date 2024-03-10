---
title: "[Docker] Docker Compose로 컨테이너 생성하기"
categories:
  - develop
---

저는 `Docker Compose`로 `MySQL`, `Redis` 등의 설정을 진행해봤습니다.

여기서는 `MySQL`을 간단하게 설정해보겠습니다.

공부 목적으로 작성되는 글이기에 틀린 부분이 있다면 댓글로 남겨주시면 감사하겠습니다. 🙇‍♂️

### docker-compose.yml

```yml
version: "3.8"

services:
  playground-db:
    container_name: playground-db ## 컨테이너 이름
    image: mysql:8.2.0 ## 사용할 Docker 이미지
    ports:
      - "3306:3306" ## 포트 번호
    volumes:
      - ./db/data:/var/lib/mysql ## 볼륨 마운트
    env_file:
      - .env ## 환경 변수 파일
    restart: on-failure
    networks:
      default:
```

- `version`으로 파일의 버전을 정의하고 `services` 섹션에서 서비스를 정의합니다.
- 3306을 포트번호로 지정했는데, 이는 `MySQL`의 기본 포트입니다.
- 볼륨 마운트(Volume Mount)는 Docker 컨테이너와 호스트 시스템 간에 파일 또는 디렉터리를 공유하는 메커니즘입니다.
- 노출되면 안되는 정보들은 환경 변수 파일로 저장했습니다.
- 마지막 네트워크 부분은 공부가 필요한 부분이라 기본값으로 설정했습니다.

<br>

```yml
command:
  - --character-set-server=utf8mb4
  - --collation-server=utf8mb4_unicode_ci
  - --skip-character-set-client-handshake
```

- 한글 깨짐 현상이 발생하여 위 옵션을 추가해줬습니다.

<br>

### .env

```
MYSQL_DATABASE= db
MYSQL_ROOT_HOST= root
MYSQL_ROOT_PASSWORD=1234
TZ=Asia/Seoul
```

- 같은 루트에 .env 파일을 작성했습니다.

<br>

`docker-compose.yml` 파일이 위치한 루트에서 다음 명령어를 통해 실행할 수 있습니다.

명령어: `docker-compose up -d`
