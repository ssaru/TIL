# 완벽한 IT 인프라 구축을 위한 Docker

업무로 인해서 급하게 Docker를 학습하려고 [완벽한 IT 인프라 구축을 위한 Docker](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788956747903&orderClick=LEA&Kc=)를 구매해서 다 읽었다. 꼼꼼히 읽은 것은 아니고 정말 필요한 부분만 열심히 읽었다. 

- 도입부

- Docker 기본
- Dockerfile
- Docker Compose

이렇게만 읽었는데, 빠르게 읽다보니 정리도 안되고 실제로 명령어를 즉각즉각 쓸 수 있는 것도 아니고 해서 Cheet Sheet 겸 TIL에 기록한다.

*(설치법은 생략한다.)*

​    

## Docker 기본



### Docker 버전확인

docker 버전 확인

```bash
$ docker version
>
Client:
    Version: 18.02.0-ce-rc1
    API version: 1.35
    ...
```

​    

### Docker 실행 환경 확인

docker 실행 환경의 상세 설정 표시

```bash
$ docker system info
>
Containers: 3
    Running: 1
    Paused: 0
    Stopped: 2
    ...    
```

​    

### Docker 디스크 이용 상황

docker가 사용하고 있는 디스크 이용상황

```bash
$ docker system df
>
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              0                   0                   0B                  0B
Containers          0                   0                   0B                  0B
Local Volumes       0                   0                   0B                  0B
Build Cache         0                   0                   0B                  0B
```

​    

### Docker Image 다운로드

Docker Image를 다운로드. 별도의 주소 및 포트를 인자로 주지 않으면 default로 DockerHub에서 Docker Image를 다운로드한다.

```bash
$ docker pull [docker image name:tag]
```

​    

### Docker Image 확인

현재 보유하고 있는 Docker Image list를 확인한다.

```bash
$ docker image ls [옵션] [레파지토리명]
or
$ docker images
```

| -all, -a    | 모든 이미지 표시        |
| ----------- | ----------------------- |
| --digests   | 다이제스트를            |
| --no-trunc  | 결과를 모두 표시        |
| --quiet, -q | Docker 이미지 ID만 표시 |

​    

### Docker 이미지 상세정보 확인

docker 이미지에 대해서 자세히 보고싶다면 다음 명령어 사용

```bash
$ docker image inspect [docker image]
```

- 이미지 ID
- 작성일
- Docker 버전
- CPU 아키텍쳐

​    

### 이미지 태그 설정

docker 이미지에 태그를 붙이려면 다음 명령어 사용

```bash
$ docker image tag [기존 도커 이미지] [설정할 태그명]
```

일반적으로 tag는 `[DockerHub 사용자명]/[이미지명]:[버젼]` 형태를 따른다.

​    

### 이미지 검색

DockerHub에 공개되어있는 이미지 검색

```bash
$ docker search [옵션] [검색 키워드]
ex)
$ docker search nginx
```

​    

### 이미지 삭제

작성한 docker 이미지 삭제

```bash
$ docker image rm [옵션] [이미지명]
ex)
$ docker image rm nginx
or 
$ docker rmi nginx
```

​    

### DockerHub에 로그인

```bash
$ docker login [옵션] [서버]
```

​    

### Docker 이미지 업로드

```bash
$ docker image push [이미지명:버전]
```

​    

### DockerHub에서 로그아웃

```bash
$ docker logout [서버명]
```

​    

### 컨테이너 생성

```bash
$ docker container create [옵션] [이미지] [명령어] [파라미터]
```

​    

### 컨테이너 시작 / 중지 / 삭제

```bash
$ docker container start [옵션] [컨테이너]
$ docker container stop [옵션] [컨테이너]
$ docker container rm [옵션] [컨테이너]
```

​    

### 컨테이너 생성 및 시작

```bash
$ docker container run [옵션] [이미지] [명령어]
```

​    

### 컨테이너 가동 확인

```bash
$ docker container status [컨테이너 식별자]
```

​    

### Docker 네트워크 목록 확인

```bash
$ docker network ls [옵션]
```

​    

### Docker 네트워크 작성

```bash
$ docker network create [옵션] [네트워크]
```

​     

### Docker 네트워크 연결

```bash
$ docker network connect [옵션] [네트워크 컨테이너]
```

​    

### Docker 네트워크 상세 정보 확인

```bash
$ docker network inspect [옵션] [네트워크]
```

​    

### Docker 네트워크 삭제

```bash
$ docker network rm [옵션] [네트워크]
```

​     

### 가동 컨테이너 연결

```bash
$ docker container attach [컨테이너]
```

​    

### 가동 컨테이너에서 프로세스 실행

```bash
$ docker container exec [옵션] [컨테이너] [실행할 명령] [인수]
```

​    

### 가동 컨테이너의 프로세스 확인

```bash
$ docker container top [컨테이너]
```

​    

### 가동 컨테이너의 포트 전송 확인

```bash
$ docker container port [컨테이너]
```

​    

### 컨테이너 이름 변경

```bash
$ docker container remane [현재 컨테이너 이름] [변경할 컨테이너 이름]
```

​    

### 컨테이너 안의 파일을 복사

```bash
$ docker container cp [컨테이너 식별자]:[컨테이너 안의 파일 경로] [호스트 디렉토리 경로]
or
$ docker container cp [호스트 파일경로] [컨테이너 식별자]:[컨테이너 안의 디텍토리 경로] 
```

​    

### 콘테이너 변경내역 확인

```bash
$ docker container diff [컨테이너]
```

​    

### 컨테이너로부터 이미지 작성

```bash
$ docker container commit [옵션] [컨테이너 식별자] [이미지명:버전명]
```

​    

### 컨테이너를 tar파일로 출력

```bash
$ docker container export [컨테이너 식별자]
```

​    

### tar 파일로부터 이미지 작성

```bash
$ docker image import [파일 또는 URL] | - [이미지명:버전명]
```

​    

### 이미지 저장

```bash
$ docker image save [옵션] [저장 파일명] [이미지명]
```

​    

### 이미지 읽어 들이기

```bash
$ docker image load [옵션]
```

​     

### 불필요한 이미지/컨테이너 일괄 삭제

```bash
$ docker system prune [옵션]
```

