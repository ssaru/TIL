# 완벽한 IT 인프라 구축을 위한 Docker

업무로 인해서 급하게 Docker를 학습하려고 [완벽한 IT 인프라 구축을 위한 Docker](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956747903&orderClick=LEA&Kc=)를 구매해서 다 읽었다. 꼼꼼히 읽은 것은 아니고 정말 필요한 부분만 열심히 읽었다. 

- 도입부

- Docker 기본
- Dockerfile
- Docker Compose

이렇게만 읽었는데, 빠르게 읽다보니 정리도 안되고 실제로 명령어를 즉각즉각 쓸 수 있는 것도 아니고 해서 Cheet Sheet 겸 TIL에 기록한다.

*(설치법은 생략한다.)*

​    

## Dockerfile

Dockerfile은 Docker 상에서 작동시킬 컨테이너의 구성 정보를 기술하기 위한 파일

​     

### Dockerfile 기본 구문

| 명령        | 설명                       |
| ----------- | -------------------------- |
| FROM        | 베이스 이미지 지정         |
| RUN         | 명령 실행                  |
| CMD         | 컨테이너 실행 명령         |
| LABEL       | 라벨 설정                  |
| EXPOSE      | 포트 익스포트              |
| ENV         | 환경 변수                  |
| ADD         | 파일/디렉토리 추가         |
| COPY        | 파일 복사                  |
| ENTRYPOINT  | 컨테이너 실행 명령         |
| VOLUME      | 불륨 마운트                |
| USER        | 사용자 지정                |
| WORKDIR     | 작업 디렉토리              |
| ARG         | Dockerfile 안의 변수       |
| ONBUILD     | 빌드 완료 후 실행되는 명령 |
| STOPSIGNAL  | 시스템 콜 시그널 설정      |
| HEALTHCHECK | 컨테이너의 헬스 체크       |
| SHELL       | 기본 쉘 설정               |

​    

### Dockerfile의 주석 서식

```dockerfile
# comment
some command # comment
```

​    

### Dockerfile 작성

#### FROM

```dockerfile
FROM [이미지명]
FROM [이미지명]:[태그명]
FROM [이미지명]@[다이제스트]
```

​    

#### RUN

Docker 이미지를 생성할 때, 실행되는 명령어

```dockerfile
RUN [실행하고 싶은 명령]

ex)
RUN apt-get install -y nginx
```

​    

##### Exec 형식

```dockerfile
# Nginx의 설치
RUN ["bin/bash", "-c", "apt-get install -y nginx"]
```

​    

#### CMD

`RUN`명령은 이미지를 작성하기 위해 실행하는 명령을 기술하지만, `CMD`는 이미지를 바탕으로 생성된 컨테이너 안에서 명령을 실행할 때, 사용한다.

Dockerfile에는 하나의 `CMD`명령을 기술 할 수 있다. 만약 여러개를 지정하면 마지막 명령만 유효하다.

```dockerfile
CMD [실행하고 싶은 명령]
```

​     

##### Exec 형식

```dockerfile
CMD ["nginx", "-g", "daemon ooff;"]
```

​    

#####     Shell 형식으로 기술

```dockerfile
CMD nginx -g 'daemon off;'
```

​    

#### ENTRYPOINT; 데몬 실행

`ENTRYPOINT`명령에서 지정한 명령은 `docker container run`명령을 실행했을 때, 실행된다.

```dockerfile
ENTRYPOINT [실행하고 싶은 명령]
```

​    

##### Exec 형식으로 기술

```dockerfile
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

​    

##### Shell 형식으로 기술

```dockerfile
ENTRYPOINT nginx -g "daemon off;"
```

​     

#### ONBUILD

`ONBUILD` 명령은 그 다음 빌드에서 실행할 명령을 이미지 안에 설정하기 위한 명령.

```dockerfile
ONBUILD [실행하고 싶은 명령]
```

​    

#### STOPSIGNAL

컨테이너가 종료할 때, 송신하는 시그널을 설정    

```dockerfile
STOPSIGNAL [시그널]
```

​    

#### HEALTHCHECK

컨테이너 안의 프로세스가 정상적으로 작동하고 있는지를 체크하고 싶을 때 사용

```dockerfile
HEALTHCHECK [옵션] CMD 실행할 명령
```

​    

#### ENV

Dockerfile안에서 환경변수를 설정하고 싶을 때, ENV 명령 사용

```dockerfile
ENV [key] [value]

OR

ENV [key]=[value]
```

​    

#### WORKDIR

작업용 디렉토리 지정

```docker
WORKDIR [작업 디렉토리 경로]
```

​    

#### USER

이미지 실행이나, Dockerfile의 다음과 같은 명령을 실행하기 위한 사용자 지정

```dockerfile
USER [사용자명/UID]
```

​     

#### LABEL

이미지에 버전 정보나 작성자 정보, 코멘트 등과 같은 정보를 제공할 때, 사용

```dockerfile
LABEL <키 명>=<값>
```

​     

#### 포트 설정(EXPOSE 명령)

컨테이너의 공개 포트 번호를 지정

```dockerfile
EXPOSE <포트 번호>
```

​    

#### ARG

Dockerfile안에서 사용할 변수 정의

```dockerfile
ARG <이름>[=기본값]
```

​    

#### SHELL

쉘 형식으로 명령을 실행할 때, 기본 쉘을 설정하려면 `SHELL`명령 사용

```dockerfile
SHELL ["쉘의 경로", "파라미터"]
```

​    

#### ADD

이미지에 호스트상의 파일이나 디렉토리를 추가할 때는 ADD 명령을 사용한다.

```dockerfile
ADD [호스트의 파일 경로] [Docker 이미지의 파일 경로]

OR

ADD ["호스트의 파일 경로", "Docker 이미지의 파일 경로"]
```

​    

#### COPY

이미지에 호스트상의 파일이나 디렉토리를 복사할 때는 COPY 명령 사용

```dockerfile
COPY [호스트의 파일 경로] [Docker 이미지의 파일 경로]
```

​     

#### VOLUME

이미지에 볼륨을 할당하려면 VOLUME 명령어 사용

```dockerfile
VOLUME ["/마운트 포인트"]
```

​    

### Dockerfile로부터 Docker 이미지 만들기

#### Docker build 명령 서식

```bash
$ docker build -t [생성할 이미지명]:[태그명] [Dockerfile의 위치]
```

​    

#### 이미지 생성시 명령어 실행 확인

```bash
$ docker history [이미지]
```

