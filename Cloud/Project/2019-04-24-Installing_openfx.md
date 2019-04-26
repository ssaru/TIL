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

- 현재 minikube의 `--vm-driver=None`옵션에 문제가 있는 듯 해서, virtualbox를 설치한다.

  - VirtualBox 버전은 5.2로 맞춘다.
    - minikube 가 VirtualBox 5.2버전을 지원한다.

- grpc-gateway

  ```bash
  $ go get -u github.com/kardianos/govendor
  $ cd $GOPATH/src
  $ govendor fetch github.com/googleapis/googleapis/google/api
  $ cd $GOPATH/src/github.com/golang/protobuf
  $ git checkout master
  $ go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway
  $ go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
  $ go get -u github.com/golang/protobuf/protoc-gen-go
  ```

# MINIKUBE

[install minikube](https://kubernetes.io/ko/docs/tasks/tools/install-minikube/) 참조

- 환경변수 설정을 아래와 같이 진행한다.

  ```bash
  export MINIKUBE_WANTUPDATENOTIFICATION=false
  export MINIKUBE_WANTREPORTERRORPROMPT=false
  export MINIKUBE_HOME=$HOME
  export CHANGE_MINIKUBE_NONE_USER=true
  export KUBECONFIG=$HOME/.kube/config
  ```

- kube 관련된 파일 및 폴더를 생성해준다.

  ```bash
  mkdir $HOME/.kube || true
  touch $HOME/.kube/config
  ```

- minikube를 실행한다.

  ```bash
  $ minikube start --vm-driver=none
  ```

# KIND(Kubernetes In Docker) -> 보류(critical bug)

[KIND github](https://github.com/kubernetes-sigs/kind) 참조

> KIND를 실행하면 `export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"`를 입력해야하는데
> 
> 이를 ~/.bashrc 같이 터미널 환경변수하는 곳에 미리 넣어놓자.
> 
> 추후에 make할 때, 해당 KUBECONFIG이 없어서 KIND로 작동이 안되는 경우가 있다.



# Minikube에 private registry 서버 구축 및 port-forwarding

> host os에 local private registry를 구축하게 되면, minikube에서 host os의 local registry 서버에 접근할 수 없음.
> 
> 따라서 [여기](https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615/)를 참조하여 minikube안에서 docker private registry 서버를 구축하고, 이를 host os에 port-forwarding하는 방법으로 사용함



- 아래의 `yaml` 파일을 사용하여, `kube-registry.yaml`파일을 생성함.

```yaml
# kube-registry.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kube-registry-v0
  namespace: kube-system
  labels:
    k8s-app: kube-registry
    version: v0
spec:
  replicas: 1
  selector:
    k8s-app: kube-registry
    version: v0
  template:
    metadata:
      labels:
        k8s-app: kube-registry
        version: v0
    spec:
      containers:
      - name: registry
        image: registry:2.5.1
        resources:
          # keep request = limit to keep this container in guaranteed class
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: REGISTRY_HTTP_ADDR
          value: :5000
        - name: REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY
          value: /var/lib/registry
        volumeMounts:
        - name: image-store
          mountPath: /var/lib/registry
        ports:
        - containerPort: 5000
          name: registry
          protocol: TCP
      volumes:
      - name: image-store
        hostPath:
          path: /data/registry/

---

apiVersion: v1
kind: Service
metadata:
  name: kube-registry
  namespace: kube-system
  labels:
    k8s-app: kube-registry
spec:
  selector:
    k8s-app: kube-registry
  ports:
  - name: registry
    port: 5000
    protocol: TCP

---

apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: kube-registry-proxy
  namespace: kube-system
  labels:
    k8s-app: kube-registry
    kubernetes.io/cluster-service: "true"
    version: v0.4
spec:
  template:
    metadata:
      labels:
        k8s-app: kube-registry
        version: v0.4
    spec:
      containers:
      - name: kube-registry-proxy
        image: gcr.io/google_containers/kube-registry-proxy:0.4
        resources:
          limits:
            cpu: 100m
            memory: 50Mi
        env:
        - name: REGISTRY_HOST
          value: kube-registry.kube-system.svc.cluster.local
        - name: REGISTRY_PORT
          value: "5000"
        ports:
        - name: registry
          containerPort: 80
          hostPort: 5000
```

- 아래 명령어를 이용하여 `kube-registry.yaml`파일을 구동함

  ```bash
  $ kubectl create -f kube-registry.yaml
  
  replicationcontroller/kube-registry-v0 created
  service/kube-registry created
  daemonset.extensions/kube-registry-proxy created
  ```

- minikube에 ssh로 접속하여, registry가 잘 작동하는지 확인

  ```bash
  $ sudo minikube ssh
                           _             _            
              _         _ ( )           ( )           
    ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
  /' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
  | ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
  (_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)
  
  $ curl http://localhost:5000/v2/_catalog
  {"repositories":[]}
  ```

- minikube의 registry를 host os에 port-forwarding

  ```bash
  $ kubectl port-forward --namespace kube-system $(kubectl get po -n kube-system | grep kube-registry-v0 | \awk '{print $1;}') 5000:5000 --request-timeout="30s" > /dev/null &
  [1] 17443
  ```

  - 이때, 프로세스는 백그라운드에서 실행되며, 표준출력은 출력되지 않는다.

    > 도커 이미지를 푸쉬하는 경우, 백그라운드 실행 상태이면 에러가 발생하는 경우가 있음
    > 
    > **백그라운드 실행을 원치 않는 경우 아래와 같은 명령어를 이용한다.**

    ```bash
    $ kubectl port-forward --namespace kube-system $(kubectl get po -n kube-system | grep kube-registry-v0 | \awk '{print $1;}') 5000:5000 --request-timeout="30s"
    ```

  - 백그라운드에서 실행 중인 프로세스를 취소하기 위해 포그라운드로 다시 돌리려면 아래 명령어를 사용한다.

    ```bash
    $ fg %1
    [1]  + 18237 running    kubectl port-forward --namespace kube-system  5000:5000 --v=0 > /dev/null
    ```



# Docker private registry 서버 구축

> host os에 local private registry를 구축하게 되면, minikube에서 host os의 local registry 서버에 접근할 수 없음.
> 
> 따라서 [여기](https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615/)를 참조하여, kubernetes안에 registry 서버를 구성하여 local 개발환경을 구축하는 방향을 생각 중



> TODO
> 
> `ifconfig`을 통해 host OS의 IP를 확인 후, 해당 IP SSH 접속을 확인함.
> 
> host OS의 registry server를 minikube가 사용할 수 있는 방법에 대해서 알아봐야함



- 디렉토리 생성

  - `mkdir -p /opt/registry/{data,compose,config,security}`

- registry 설정파일 생성

  `/opt/registry/config/cred_config.yml`

  > `/opt/registry/`에 대한 디렉토리를 변경해야하는 경우, 해당 파일은 건들이지 않는다.
  > 
  > 이유는, 해당 파일은 docker container안에서 접근하는 경로이기 때문이다.
  > 
  > 따라서 `/opt/registry/compose/cred_config.yml` 수정시 `--volume`옵션을 이용해서 로컬 디바이스의 registry 디렉토리 경로를 변경해준다.

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

  `/opt/registry/compose/cred_config.yml`

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

  - `echo "<PASSWD>" | sudo htpasswd -iB -c /opt/registry/security/htpasswd <ACCOUNT>`

- docker compose up 해서 registry 서버 가동

  - `docker-compose -f /opt/registry/compose/cred_config.yml up -d`

- docker login을 통해 작동 확인

  - `docker login "IP:PORT"`

- 5000번 포트를 개방해야함

  - Linux: 

    ```bash
    $ sudo iptables -A INPUT -p tcp --dport 0:65535 -j DROP
    $ sudo iptables -A INPUT -p udp --dport 0:65535 -j DROP
    ```

  - Mac:

    아래의 명령어를 이용하여 nmap 설치 후, `pf.conf` 에 아래 구문을 추가

    ```bash
    $ brew install nmap
    
    # 현재 port scan
    $ nmap -p 5000 localhost
    
    # port 개방을 위한 파일 수정
    $ sudo vim /etc/pf.conf
    
    # 구문 추가
    pass in proto tcp from any to any port 1234
    
    # 추가 구문 적용
    $ sudo pfctl -f /etc/pf.conf
    # port open 확인
    $ nmap -p 5000 localhost
    ```

- insecure registries 등록

  - private 저장소에 로그인하기 위해서 다음과 같이 설정이 필요하다.

    ```bash
    $ echo '{"insecure-registries": ["<YOUR PRIVATE REGGISTRY SERVIER IP:PORT>"]}' >> /etc/docker/daemon.json
    $ service docker restart
    ```

- Registry에 등록된 이미지는 아래와 같은 request로 확인한다.

  ```bash
  $ curl -X GET http://<ID>:<PASSWORD>@<REGISTRY>:<PORT>/v2/_catalog
  ```

  - 레퍼런스는 [여기](https://stackoverflow.com/questions/31251356/how-to-get-a-list-of-images-on-docker-registry-v2)를 참조



# OpenFx 컴파일

## gateway

Makefile을 이용해서 docker image를 생성하고, 생성한 private repository에 저장한다.

- Makefile에서 `REGISTRY=<PRIVATE REGISTRY SERVER IP> : <PORT>`를 적합하게 변경해준다.

- gateway 폴더의 `deploy/yaml/gateway-dep.yml` 파일에서 containers의 images address를 private registry server에 맞춰 변경해줘야한다.

- `make`명령어를 통해서 gateway를 make한다.

  - 여기서 make 명령어는 아래와 같이 순차적으로 실행해야한다.

    > 절대 make 단독으로 사용하지 말 것.
    > 
    > make만 사용하게 되면 build, push, deploy 순차적으로 실행되는데 아직 deploy할 단계가 아니다.

    ```bash
    $ make build
    $ make push 
    ```



## watcher

- watcher를 git clone한다.

- openfx를 git clone한다.

  - clone하게 되면 폴더 이름이 `OpenFx`로 되어있을 텐데, 이를 `openfx`로 변경한다.

- make한다.

  > 현재 go Dockerfile build시, 아래와 같은 에러가 출력됨. protobuf 버전 업그레이드로 인해 컴파일이 안되는 듯 함
  > 
  > Step 7/9 : RUN go build -o fxwatcher .
  >  ---> Running in 8b68b098754d
  > 
  > github.com/keti-openfx/openfx-watcher/go/pb
  > 
  > pb/fxwatcher.pb.go:25:11: undefined: "github.com/keti-openfx/openfx-watcher/go/vendor/github.com/golang/protobuf/proto".ProtoPackageIsVersion3

  - protobuf와 protoc을 최신으로 upgrade한다.

    - https://github.com/protocolbuffers/protobuf/releases

    - protoc

      ```bash
      # Make sure you grab the latest version
      curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v3.7.1/protoc-3.7.1-linux-x86_64.zip
      
      # Unzip
      unzip protoc-3.7.1-linux-x86_64.zip -d protoc3
      
      # Move protoc to /usr/local/bin/
      sudo mv protoc3/bin/* /usr/local/bin/
      
      # Move protoc3/include to /usr/local/include/
      sudo mv protoc3/include/* /usr/local/include/
      
      # Optional: change owner
      sudo chown [user] /usr/local/bin/protoc
      sudo chown -R [user] /usr/local/include/google
      ```

    - dep를 설치하고, dependency를 설치한다.

      > `go ensure`를 사용하게 될 경우 private repository를 가져올 때 dep가 어떠한 에러를 출력하지도 않은 상태로 계속 돌고있는 경우가 있다. 이때는 아래의 경우를 진행한다.

      ```bash
      $ go get -u github.com/golang/dep/cmd/dep
      $ go ensure
      ```

    - protoc-gen-go를 직접 설치한다.

      ```bash
      $ git clone https://github.com/golang/protobuf
      $ cd protobuf/protoc-gen-go
      $ git checkout tags/v1.2.0 -b v1.2.0
      $ go install
      ```

    - installing grpc_tools

      ```bash
      $ pip install grpcio-tools
      $ pip3 install grpcio-tools
      ```



## CLI

- openfx-cli clone 후, 아래와 같은 절차를 따른다.

  ```bash
  $ cd openfx-cli
  $ dep init
  $ dep ensure
  $ make
  ```

- $GOPATH/bin 에보면 openfx-cli 실행파일을 확인할 수 있다.



## Runtime

- Runtime은 단순히 github에 push하는 역활만 하므로 생략



## openfx yaml(kubernetes specfile)

- 이 또한 Kubernetes 배포를 위한 yaml 파일을 github에 push 하므로 생략한다.



### REFERENCE

[1. Sharing a local registry with minikube](https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615/)

[2. Port-forwarding times out](https://github.com/kubernetes/kubernetes/issues/19231)

[3. Port Forwarding to Access Application in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

[4. Setting up a local Kubernetes cluster with insecure registries](https://medium.com/@alombarte/setting-up-a-local-kubernetes-cluster-with-insecure-registries-f5aaa34851ae)

[5. minikube configuration with virtualbox driver](https://gist.github.com/kunalg/015aacb58d18bd110844922da7329c22)

[6. background process stdout not display](https://jybaek.tistory.com/115)

[7. kubectl port-frowarding options](https://www.mankier.com/1/kubectl-port-forward)

[8. kubernetes logging level --v](https://stackoverflow.com/questions/43991231/kubernetes-logging-level-v)

[9. kubernetes + private docker registry](https://blog.uniqbuild.co.kr/?p=724)

[10. most common reasons kubernetes deployments fail](https://blog.uniqbuild.co.kr/?p=724)

[11. Enabling Docker Insecure Registry](https://github.com/kubernetes/minikube/blob/master/docs/insecure_registry.md)

[12. Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

[13. kubectl cheetsheet](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/#%EB%A6%AC%EC%86%8C%EC%8A%A4-%EC%82%AD%EC%A0%9C)






