# Introduction to Docker - (KUBERNETES IN THE GOOGLE CLOUD 1주차)



Kubernetes 스터디잼을 진행하고 있다. 스터디잼이 해당 강의를 듣고, Blog글을 쓰는 것이라 간단하게 1강을 듣고 내용을 정리해본다.

​    

## Overview

Overview에서는 Docker에 대해서 간략하게 설명한다.



WIKI에서는 Docker에 대한 설명을 다음과 같이 하고있다.

> **Docker** is a [computer program](https://en.wikipedia.org/wiki/Computer_program) that performs [operating-system-level virtualization](https://en.wikipedia.org/wiki/Operating-system-level_virtualization), also known as "containerization".[[6\]](https://en.wikipedia.org/wiki/Docker_(software)#cite_note-SYS-CON_Media-7) It was first released in 2013 and is developed by [Docker, Inc.](https://en.wikipedia.org/wiki/Docker,_Inc.)[[7\]](https://en.wikipedia.org/wiki/Docker_(software)#cite_note-os4u-8)



개발자들이 협업을 통해 개발을 진행할 때, 제일 중요한 것은 같은 개발환경 및 배포환경이 같아야한다는 것이다. 예전에는 이를 수동으로 해주거나 가상머신(VM)을 사용했다.(그림 오른쪽) 수동으로 개발환경을 하나하나 맞추는 것은 시간적인 비용 / 에러 / 정확하게 같지 않은 개발&배포환경이라는 문제를 발생시킨다. VM의 경우에는 OS위에 Guest OS를 하나 더 올리는 것을 통해서 Overhead가 발생하여 Host OS보다 성능이 크게 떨어진다는 단점이 있다.

이를 보완하고자 나온 것이 Docker이다(그림 왼쪽) . Docker는 VM의 몇가지 단점들을 Linux Container라는 기술로 쉽게 풀어냈다고 이해하면 된다. Docker를 사용하는 것을 통해서 이점을 볼 수 있는 것들은 다음과 같다.



- 복잡한 리눅스 어플리케이션을 컨테이너로 묶어서 실행할 수 있음
- 개발, 테스트, 서비스 환경을 하나로 통일하여 효율적으로 관리할 수 있음



![vms-and-containers](https://user-images.githubusercontent.com/13328380/51082023-6ff2b180-1741-11e9-9f45-3f19ec71eb77.jpg)

해당 세션(Introduction to Docker)에서 배울 것들은 다음과 같다.

- Docker container들을 `build`/`run`/`debug`하는 방법
- Docker Hub나 Google Container Registry에서 `Dockere Image`들을 pull 하는 방법
- Google Container Registry에 Docker Image를 push하는 방법



(해당 포스트에서는 Docker를 어떻게 설치하는지 가이드하지는 않겠다. 해당 부분은 별도로 구글 검색을 통해서 설치를 진행하기를 추천한다.)

​     

## Hello World

자세하게 알기전에 일단 실행부터 해보자

```bash
$ docker run hello-world

>>
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9db2ca6ccae0: Pull complete
Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```



정확하게 `hello-world` container를 실행했다면 우리는 `Hello from Docker!`라는 return 값을 화면에서 확인할 수 있다.



출력된 메세지를 한번 자세히 확인해보자. 이를 확인해보면 간단한 `docker run hello-world`라는 명령을 입력했을 때,  Docker가 어떻게 작동하는지 확인할 수 있다.



1. `hello-world:latest`를 local에서 찾아본다.
2. `hello-world:latest`이미지가 없으니 DockerHub의 `library/hello-world`라는 곳에서 이미지를 pull한다.
3. 다운받은 이미지에서 컨테이너를 생성하고 실행한다.



## Docker Image list 확인하기

현재 내가 가지고 있는 Docker Image의 리스트를 확인해보고 싶다면, 다음 명령어로 확인한다.



```bash
$ docker images
>>
REPOSITORY     TAG      IMAGE ID       CREATED       SIZE
hello-world    latest   1815c82652c0   6 days ago    1.84 kB
```

- Docker Image ID는 [SHA256 hash](https://www.movable-type.co.uk/scripts/sha256.html) format으로 되어있다.
- `run`명령어 실행시 Docker daemon이 Docker Image를 local에서 찾지 못할 경우, 기본적으로 public registry인 Docker Hub에서 이를 찾아 pull한다.
- Docker daemon이 Docker Image를 local에서 찾을 경우는 그냥 이를 이용하여 container를 생성한다.

​    

## Docker container list 확인하기

현재 docker container의 list를 확인하고싶다면, 다음과ㅏ 같은 명령어를 이용한다.

```bash
$ docker ps
>>
CONTAINER ID        IMAGE               COMMAND             CREATED             
STATUS              PORTS               NAMES
```



위에서 보이는 것은 현재 실행되고있는 docker container의 list만 보여준다.

만약에 이전에 실행된 docker container의 list도 확인하고싶다면, 다음과 같은 명령어를 이용한다.



```bash
$ docker ps -a
>>
CONTAINER ID      IMAGE           COMMAND      ...     NAMES
6027ecba1c39      hello-world     "/hello"     ...     elated_knuth
358d709b8341      hello-world     "/hello"     ...     epic_lewin
```



해당 결과에는 컨테이너 ID, Docker가 컨테이너를 식별하기 위해 생성 한 UUID 및 실행에 대한 더 많은 메타 데이터가 표시된다. 일반적으로 컨테이너 이름은 무작위로 생성되지만 명시적으로 이를 입력할 수 있다.

​    

## Build

간단한 Node Application을 이용하여 Docker Image를 build해보자

간단한 폴더를 하나 만들어보자.

```bash
$ mkdir test && cd test
```



폴더를 만들었다면, 내부에 Docker image를 build하기 위한 `Dockerfile`을 작성해준다.

```dockerfile
# Use an official Node runtime as the parent image
FROM node:6

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
```



이제 해당 `Dockerfile`의 의미를 차례차례 분석해보자.

1. 상위 이미지로 `node:6`을 사용한다.
2. 컨테이너의 작업 디렉토리를 `/app`으로 설정한다.
3. 현재 디렉토리의 내용을 container의 `/app`으로 복사한다.
4. container의 port 80을 외부에 공개한다.
5. container가 시작될 때 `app.js`를 실행한다.



여기에서 `app.js`는 다음과 같이 구성되어있다.

```javascript
const http = require('http');

const hostname = '0.0.0.0';
const port = 80;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});

process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
```



`app.js`는 80번 포트를 갖고 `Hello World`를 return하는 간단한 HTTP 서버이다.



자 이제 다음 명령어를 이용하여 `Dockerfile`을 Docker Image로 build하자



```bash
$ docker build -t node-app:0.1 .
>>
Sending build context to Docker daemon 3.072 kB
Step 1 : FROM node:6
6: Pulling from library/node
...
...
...
Step 5 : CMD node app.js
 ---> Running in b677acd1edd9
 ---> f166cd2a9f10
Removing intermediate container b677acd1edd9
Successfully built f166cd2a9f10
```

- 마지막 `.`은 `Dockerfile`의 디렉토리를 나타낸다.

- `-t`옵션은 이미지의 name&tag를 `name:tag`형태의 인자를 받는 옵션이다.

  (tag는 Docker Images를 build할 때, 추천되는 항목이다. 만약에 tag를 안붇인다면, 해당 image의 tag는 자동적으로 `latest`결정될 것이다.)

- 해당 `build` 명령어를 실행했을 때, 메세지 출력을 자세히 보면, **Docker Image는 하나의 layer를 만들어서 이전 layer에 쌓아가는 식으로 Image를 만들고 있음**을 확인할 수 있다.

  (이러한 구성은 Docker Image의 크기를 작게만드는데 핵심이 된다.)



`build`가 완료되었다면 `docker images`명령어로 image가 잘 생성되었는지 확인할 수 있다.

```bash
$ docker images
>>
REPOSITORY     TAG      IMAGE ID        CREATED            SIZE
node-app       0.1      f166cd2a9f10    25 seconds ago     656.2 MB
node           6        5a767079e3df    15 hours ago       656.2 MB
hello-world    latest   1815c82652c0    6 days ago         1.84 kB
```

​    

## Run

위에서 Docker Image build를 성공적으로 진행했다면, 다음과 같은 `run`명령어로 container를 실행할 수 있다.

```bash
$ docker run -p 4000:80 --name my-app node-app:0.1
>>
Server running at http://0.0.0.0:80/
```

- `--name` flag는 container에 name을 만들어주는 옵션이다.

- `-p`옵션은 host의 `4000 port`와 container의 `80 port`를 서로 연결해주는 옵션이다.

  (이러한 port mapping없이는 host os에서 container의 localhost에 접근할 수 없다.)

- container는 초기에 터미널이 작동하는 시간동안만 작동을 한다. container가 background에서 작동하게끔 하고 싶다면 `-d`옵션을 사용하면 된다.



성공적으로 container를 run했다면, 다음과 같은 방법을 이용해서 container가 정확하게 작동하는지 확인해보자.

```bash
$ curl http://localhost:4000
>>
Hello World
```

​     

### Container run in the background

자 이제 `-d`옵션을 통해서 background에서 container를 작동시켜보자.

기존의 container를 먼저 종료한다.

```bash
$ docker stop my-app & docker rm my-app
```



container를 종료했다면, 다음과 같은 명령어를 이용하여 docker container를 background에서 작동시키고, container list를 확인하는 것을 통해서 잘 작동하는지 확인한다.

```bash
$ docker run -p 4000:80 --name my-app -d node-app:0.1
$ docker ps

>>
CONTAINER ID   IMAGE          COMMAND        CREATED         ...  NAMES
xxxxxxxxxxxx   node-app:0.1   "node app.js"  16 seconds ago  ...  my-app
```

​    

### Check Container Log

다음과 같은 명령어를 통해서 docker container의 log를 확인할 수 있다.

```bash
$ docker logs [container_id]
```

- 여기서 `[container_id]`라고 함은 `docker ps`를 입력했을 때, `CONTAINER ID`에 적힌 내용을 의미한다.



이제 위에서 background로 실행했던 container의 log를 확인해보자

```bash
docker logs xxxxxxxxxxxx
>>
Server running at http://0.0.0.0:80/
```

​    

### Change code

처음에 작성했던 `app.js`가 도중에 변경되어 재빌드되면 Docker에서 어떤일이 발생할까?

한번 알아보자.



처음에 만들었던 `test`폴더로 들어간다.

```bash
$ cd test
```



`test`폴더로 들어왔다면, `app.js`파일을 다음과 같이 부분적으로 수정한다.

```javascript
....
const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Welcome to Cloud\n');
});
....
```



수정이 완료되었다면, build명령어를 이용하여 `0.2` 버전의 Docker Image를 재빌드한다.

```bash
$ docker build -t node-app:0.2 .
>>
Step 1/5 : FROM node:6
 ---> 67ed1f028e71
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> a39c2d73c807
Step 3/5 : ADD . /app
 ---> a7087887091f
Removing intermediate container 99bc0526ebb0
Step 4/5 : EXPOSE 80
 ---> Running in 7882a1e84596
 ---> 80f5220880d9
Removing intermediate container 7882a1e84596
Step 5/5 : CMD node app.js
 ---> Running in f2646b475210
 ---> 5c3edbac6421
Removing intermediate container f2646b475210
Successfully built 5c3edbac6421
Successfully tagged node-app:0.2
```

- 출력 메세지를 확인해보면, `Step1 ~ 2`까지는 변경되지 않고, 그 이후의 변경된 `app.js`로 인해서 layer가 수정된 것을 확인할 수 있다.



이제 변경된 Docker Image를 container로 실행시켜보자.

( 이전에 background로 `4000:80` 포트 맵핑이된 node.js 어플리케이션이 구동중이니 해당 포트 매핑은 피한다.)

```bash
docker run -p 8080:80 --name my-app-2 -d node-app:0.2
docker ps
>>
CONTAINER ID     IMAGE             COMMAND            CREATED             
xxxxxxxxxxxx     node-app:0.2      "node app.js"      53 seconds ago      ...
xxxxxxxxxxxx     node-app:0.1      "node app.js"      About an hour ago   ...
```



해당 container의 동작을 확인해보면 다음과 같다.

```bash
$ curl http://localhost:8080
$ curl http://localhost:4000
>>
Welcome to Cloud
Hello World
```

​    

## Debug

이제 Docker를 이용해 Docker Image를 build하는 것과 Docker Container를 run하는 것은 익숙해졌을 것이다.

이제 Debug에 대해서 알아보자.



### Log

위에서도 언급했지만, `docker logs`를 통해서 해당 container의 log를 확인할 수 있다.

만약에 현재 작동중인 container의 log를 확인하고 싶다면, 다음 명령어를 이용한다.

```bash
$ docker logs -f [container_id]
>>
Server running at http://0.0.0.0:80/
```

​     

### Access Bash of Container

만약에 해당 Container bash에 붙고싶다면 다음과 같은 명령어를 이용한다.

```bash
$ docker exec -it [container_id] bash
>>
root@xxxxxxxxxxxx:/app#
```

- `-it`옵션은 pseudo-tty를 할당하고 stdin을 열린 상태로 유지함으로써 컨테이너와 상호 작용할 수 있다.



만약에 Bash session을 종료하고싶다면, `exit`를 통해서 나올 수 있다.

​    

### Check metadata of container

Docker Container의 metadata를 확인해보고 싶다면, `inspect`를 통해서 확인할 수 있다.

```bash
$ docker inspect [container_id]
>>
[
    {
        "Id": "xxxxxxxxxxxx....",
        "Created": "2017-08-07T22:57:49.261726726Z",
        "Path": "node",
        "Args": [
            "app.js"
        ],
...
```

- `--format`옵션을 사용해서 반환된 JSON의 특정 필드만 검사할 수 있다.

  ```bash
  $ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress} {{end}}' [container_id]
  >>
  192.168.9.3
  ```



## Publish

만든 Docker Image를 Google Container Registry(gcr)에 push 해보자.

push 전에 project ID를 번저 확인한다.

```bash
$ gcloud config list project
>>
[core]
project = [project-id]

Your active configuration is: [default]
```



기존에 만들었던 `node-app:0.2`를 gcr에 올릴 수 있도록 tag를 재설정한다.

```bash
$ docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2
$ docker images
>>
REPOSITORY                      TAG         IMAGE ID          CREATED
node-app                        0.2         76b3beef845e      22 hours ago
gcr.io/[project-id]/node-app    0.2         76b3beef845e      22 hours ago
node-app                        0.1         f166cd2a9f10      26 hours ago
node                            6           5a767079e3df      7 days ago
hello-world                     latest      1815c82652c0      7 weeks ago
```



확인이 완료되었다면, docker image를 gcr에 push한다.

```bash
$ gcloud docker -- push gcr.io/[project-id]/node-app:0.2
>>
The push refers to a repository [gcr.io/[project-id]/node-app]
057029400a4a: Pushed
342f14cb7e2b: Pushed
903087566d45: Pushed
99dac0782a63: Pushed
e6695624484e: Pushed
da59b99bbd3b: Pushed
5616a6292c16: Pushed
f3ed6cb59ab0: Pushed
654f45ecb7e3: Pushed
2c40c66f7667: Pushed
0.2: digest: sha256:25b8ebd7820515609517ec38dbca9086e1abef3750c0d2aff7f341407c743c46 size: 2419
```



무사히 push를 완료했다면 GCP에서 이를 확인할 수 있다.

​    

## Docker Container Stop & Remove All 

모든 Docker Container를 정지하고 삭제하고 싶다면, 다음과 같은 명령어를 이용한다.

#### 정지

```bash 
$ docker stop $(docker ps -q)
```



#### 삭제

```bash
docker rm $(docker ps -aq)
```

​    

## Docker Images Remove All 

Docker Image를 삭제하는 명령어는 `docker rmi`이다.

만약에 특정 Docker Image만 삭제하고싶다면 다음과 같은 명령어를 이용한다.

```bash
$ docker rmi node-app:0.2 gcr.io/[project-id]/node-app node-app:0.1
```



하지만, 모든 Docker Image를 삭제하고싶다면 다음과 같은 명령어를 통해서 모두 삭제한다.

```bash
$ docker rmi $(docker images -aq)
$ docker images
>>
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

​    

## Pull Docker Image 

나에게 없는 Docker image를 pull하고싶다면 `docker pull`명령어를 이용하여 image를 pull한다.

```bash
$ gcloud docker -- pull gcr.io/[project-id]/node-app:0.2
```

​    

## Reference

[[1] 도커 무작정 따라하기 : 도커가 처음인 사람도 60분이면 웹 서버를 올릴 수 있습니다.](https://www.slideshare.net/pyrasis/docker-fordummies-44424016)

[[2] Playing Catch-up with Docker and Containers](https://rancher.com/playing-catch-docker-containers/)

[[3] 초보를 위한 도커 안내서 - 도커란 무엇인가?](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)