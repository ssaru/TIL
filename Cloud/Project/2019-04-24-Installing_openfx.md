회사 프로젝트를 팔로우업 해야하는데, 일단 개발환경셋팅을 할 줄 알아야해서 이를 해보고 정리해본다.

개발환경셋팅 순서는 아래와 같다.



- 가상 kubernetes환경을 만들기 위한 kind 셋팅

- docker private registry server 구축

- compile openfx

- 배포



이를 위해 기본적으로 준비되어야하는 것은 다음과 같다.

- kubectl

  - [Install and Set Up kubectl ](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

- docker

- docker-compose

- htpasswd

  ```bash
  $ sudo apt-get install apache2-utils
  ```

- gRPC

  ```bash
  $ go get -u google.golang.org/grpc
  $ go get -u golang.org/x/net
  $ go get -u golang.org/x/sys/unix
  ```

- protocol buffers

  ```bash
  $ wget https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protoc-3.6.1-linux-x86_64.zip
  $ unzip protoc-3.6.1-linux-x86_64.zip -d protoc3
  $ sudo mv protoc3/bin/* /usr/local/bin/
  $ export PATH=$PATH:/usr/local/bin
  $ sudo mv protoc3/include/* /usr/local/include/
  ```

- protoc plugin

  ```bash
  $ go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
  $ export PATH=$PATH:$GOPATH/bin
  ```



# KIND(Kubernetes In Docker)

[KIND github](https://github.com/kubernetes-sigs/kind) 참조



# Docker private registry 서버 구축



- 디렉토리 생성

  - `mkdir -p /opt/registry/{data,compose,config,security}`

- registry 설정파일 생성

  ```yaml
  version: 0.1
  log:
    fields:
      service: registry
  storage:
    delete:
      enabled: true
    cache:
      blobdescriptor: inmemory
    filesystem:
      rootdirectory: /opt/registry/data
  http:
    addr: :5000
    headers:
      X-Content-Type-Options: [nosniff]
      Access-Control-Allow-Origin: ['<IP ADDRESS>']
      Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE']
      Access-Control-Allow-Headers: ['Authorization']
      Access-Control-Max-Age: [1728000]
      Access-Control-Allow-Credentials: [true]
      Access-Control-Expose-Headers: ['Docker-Content-Digest']
  auth:
    htpasswd:
      realm: basic-realm
      path: /opt/registry/security/htpasswd
  ```



- docker-compose 파일 생성

  ```yaml
  version: '2.0'
  services:
    registry:
      image: registry:2
      ports:
        - 5000:5000
      volumes:
        - /opt/registry:/opt/registry
        - /opt/registry/config/cred_config.yml:/etc/docker/registry/config.yml
  ```



- docker registry에 인증할 수 있게 authentication 설정

  - `echo "<PASSWD>" | sudo htpasswd -iB /opt/registry/security/htpasswd <ACCOUNT>`

- docker compose up 해서 registry 서버 가동

  - `docker-compose -f /opt/registry/compose/cred_config.yml up -d`

- docker login을 통해 작동 확인

  - `docker login "IP:PORT"`



# OpenFx 컴파일















































































ㅇ











































































ㅇ
