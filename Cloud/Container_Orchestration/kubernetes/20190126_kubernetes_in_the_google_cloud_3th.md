# KUBERNETES IN THE GOOGLE CLOUD 3주차



이번 세션에서는 7, 8, 9, 10강을 한번에 리뷰한다.

개인적으로 불필요한 내용들이 몇개 있는거 같아서 실제로 내가 궁금한 것들만 정리했다.

각 세션에 대한 Overview는 아래에 정리해두니 참고하자

​      

**Chapter 7 - Overview**

[Botkit toolkit](https://howdy.ai/botkit/)을 이용해서 Slack Bot을 만들고 GCP에서 실행한다. 

   

해당 세션에서는 아래와 같은 내용을 학습할 수 있다.

- slack에서 custom slackbot을 만들 수 있다
- Docker에서 Node.js 이미지를 빌드할 수 있다.
- Docker image를 private Google Container Registry에 업로드할 수 있다.
- Slackbot을 Kubernetes Engine에서 실행하고 관리할 수 있다.

​         

**Chapter 8 - Overview**

Kubernetes Engine에서 private cluster는 master가 public internet을 통해서 접근하지 못하게한다.

해당 세션에서는 GKE에서 Private Kubernetes Cluster를 만드는 방법에 대해서 학습한다.

​    

**Chapter 9 -Overview** 

Helm는 `Charts`라고 불리는 Kubernetes package를 관리하는 툴셋이며, 여기에는 pre-configured kubernetes reousrce들을 포함하고있다.



해당 세션에서는 아래와 같은 내용을 학습할 수 있다.

- GKE에서 Kuberntes Cluster 생성
- Helm 설치
- MySQL chart 설치
- MySQL client로 설치된 MySQL chart 테스트

​     

**Chapter 10 - Overview**

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) 는 외부의 사용자 및 client application들이 HTTP service에 접근할 수 있게 해준다. [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) 는 2가지 컴포넌트로 구성되어있다.

- **Ingress Resource**

  Ingress Resource는 inbound traffic이 Service에 도착할 수 있게 해주는 규칙들의 집합이다. 

- **Ingress Controller**

  HTTP, L7 load balancer를 통해서 Ingress Resource로 설정된 rule들 위에서 작동한다. 외부 클라이언트에서 Kubernetes 서비스로 트래픽을 라우팅 할 수 있게 두 부분을 적절하게 구성하는게 핵심이 된다.



아래 그림은 해당 세션에서 실습할 기본 flow에 대해서 묘사한다. 

![499ada984a455cbbb0dee7cc376f19219712f624228fcd2330aadbc61efaba48](https://user-images.githubusercontent.com/13328380/51796356-fc1dd200-2233-11e9-8ef1-d852bdf91057.png)

​    

해당 세션에서는 아래와 같은 내용을 학습할 수 있다.

- 간단한 kubernetes web application을 배포한다.
- Helm Chart를 이용해서 NGINX Ingress Controller를 배포한다.
- NGINX ingress를 controller로 사용하는 application을 위해서 Ingress Resource를 배포한다.
- Google Cloud L4(TCP/UDP) Load Balancer frontend IP로 접근해보는 것을 통해서 NGINX Ingress의 기능을 테스트한다. 그리고 이를 이용해서 web application에 접속할 수 있는지 확인한다.



## What is Helm?

Helm은 `Cloud Native Computing Foundation (CNCF)`가 유지보수하고 있는 Kubernetes package manager이다. Helm은 update roll out그리고 option들을 application에 수정하는 거 같은 kubernetes application 관리를 간단하게 해주는 이점으로 인해서 클라우드 개발자들에게 유명해졌다.



Helm은 2가지 부분으로 구성되어있다.

- **Client**(Helm)
- **Server**(Tiller)

​     

## Install Helm

아래 script파일을 받아서, 이를 실행한다.

```bash
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```



Helm과 Tiller를 초기화하기 전에 Tiller 서비스를 만들어야한다.

아래 명령어를 이용해서 Tiller 서비스를 만들자

```bash
$ kubectl -n kube-system create sa tiller
```



Cluster role을 서비스 계정에 바인딩한다.

```bash
$ kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```

​    

## Initialize Helm and Install Tiller

Helm의 local environment를 설정하고, Tiller를 아래와 같은 명령어로 설치하자

```bash
$ helm init --service-account tiller
>
Creating /home/gcpstaging3629_student/.helm
Creating /home/gcpstaging3629_student/.helm/repository
Creating /home/gcpstaging3629_student/.helm/repository/cache
Creating /home/gcpstaging3629_student/.helm/repository/local
Creating /home/gcpstaging3629_student/.helm/plugins
Creating /home/gcpstaging3629_student/.helm/starters
Creating /home/gcpstaging3629_student/.helm/cache/archive
Creating /home/gcpstaging3629_student/.helm/repository/repositories.yaml
$HELM_HOME has been configured at /home/gcpstaging3629_student/.helm.
```



이를 잘 실행했는지 확인하기 위해서 sanity check를 하자.

```bash
$ kubectl get po --namespace kube-system
>
NAME                          READY       STATUS    RESTARTS      AGE
fluentd-gcp-v2.0-jvqc2    2/2       Running    0          34m
fluentd-gcp-v2.0-tv0kt    2/2       Running    0          34m
fluentd-gcp-v2.0-tzkkq    2/2       Running    0          34m
Heapster-v1.3.0-128....     2/2       Running    0          33m
kube-dns-3664836949-78znt 3/3       Running    0          34m
kube-dns-3664836949-8c27t 3/3       Running    0          34m
kube-dns-autoscaler-2667913178-jbqqp                   1/1       Running   0          34m
kube-proxy-gke-my-cluster-default-pool-8c8019d4-9xz9   1/1       Running   0          35m
kube-proxy-gke-my-cluster-default-pool-8c8019d4-ghj6   1/1       Running   0          35m
kube-proxy-gke-my-cluster-default-pool-8c8019d4-r3xk   1/1       Running   0          33m
kubernetes-dashboard-2917854236-sh7xx                  1/1       Running   0          34m
l7-default-backend-1044750973-hv34r                    1/1       Running   0          34m
tiller-deploy-4026033803-gdrrg                         1/1       Running   0          3m
```

​    

```bash
$ helm version
>
Client: &version.Version{SemVer:"v2.6.1", GitCommit:"bbc1f71dc03afc5f00c6ac84b9308f8ecb4f39ac", GitTreeState:"clean"}

Server: &version.Version{SemVer:"v2.6.1", GitCommit:"bbc1f71dc03afc5f00c6ac84b9308f8ecb4f39ac", GitTreeState:"clean"}
```

​    

## Install a Chart

이제 Chart를 설치하자. Chart를 설치하는 방법은 2가지가 있으나, 여기서는 공통적으로 많이 쓰는 명령어 하나를 실행할 것이다. 아래와 같은 명령어로 설치할 수 있는 chart들의 list를 업데이트하자

```bash
$ helm repo update
>
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```



이제 MySQL chart를 설치하자

```bash
$ helm install stable/mysql
>
NAME:   giddy-rodent
LAST DEPLOYED: Tue Sep 19 17:04:27 2017
NAMESPACE: default
STATUS: DEPLOYED
RESOURCES:
==> v1/Secret
NAME                TYPE    DATA  AGE
giddy-rodent-mysql  Opaque  2     1s
==> v1/PersistentVolumeClaim
NAME                STATUS   VOLUME    CAPACITY  ACCESSMODES  STORAGECLASS  AGE
giddy-rodent-mysql  Pending  standard  1s
==> v1/Service
NAME                CLUSTER-IP    EXTERNAL-IP  PORT(S)   AGE
giddy-rodent-mysql  10.31.247.29  <none>       3306/TCP  1s
==> v1beta1/Deployment
NAME                DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
giddy-rodent-mysql  1        1        1           0          1s
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
giddy-rodent-mysql.default.svc.cluster.local

To get your root password run:
    kubectl get secret --namespace default giddy-rodent-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo

To connect to your database:
1. Run an Ubuntu pod that you can use as a client:
    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:
    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h giddy-rodent-mysql -p
```

​    

## Get your Root Password

Helm으로 설치하는 MySQL의 root password는 아래와 같은 명령어로 알 수 있다

```bash
$ kubectl get secret --namespace default giddy-rodent-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo
```

​    

### Install and Ubuntu Pod

Client로 Ubuntu pod를 설치하자. 

```bash
$ kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il
```

​    

## Connect to your MySQL Database

MySQL client를 설치하자

```bash
$ apt-get update && apt-get install mysql-client -y
```



client를 설치했으니, MySQL에 접속해보자

```bash
$ mysql -h giddy-rodent-mysql -p
```

​    

## Deploying the NGINX Ingress Controller via Helm

kubernetes platform은 administrator에게 Ingress Controller의 유연성을 제공한다. - 서비스 제공자(클라우드 서비스 제공자)가 제공하는 기본 제공 서비스를 이용하지 않고, 스스로 자신의 제품을 통합할 수 있다. NGINX controller는 반드시 외부에서 접속이 가능하게 노출되어야하고, 이는 NGINX controller 서비스 위의 Service `type: LoadBalancer` 를 사용해서 해결할 수 있다. GKE에서는 NGINX Controller Service를 포함해서 Google Cloud Network(TCP/IP) Load Balancer를 만들고 이는 백엔드 역활을 하게 된다. Google Cloud는 적합한 방화벽 규칙을 Service의 VPC안에 만든다. 이는 loadbalancer 프론트엔드 IP 주소에 web HTTP(s) 트래픽을 허용한다.



### NGINX Ingress Controller on Kubernetes Enginie

GKE에서 실행되는 NGINX controller가 어떻게 시각적으로 표현되는지에 대한 묘사는 아래와 같다.

![1356d6cd1e7d299f188e5c338fd266e4cd77f0bd49c27ff2758f5c65d0f60385](https://user-images.githubusercontent.com/13328380/51796548-1efeb500-2239-11e9-8b8f-b0e1a6ea8baf.png)

### Deploy NGINX Ingress Controller

NGINX Ingress Controller를 배포한다.

```bash
$ helm install --name nginx-ingress stable/nginx-ingress --set rbac.create=true
>
==> v1/Service
NAME                            TYPE          CLUSTER-IP    EXTERNAL-IP  
nginx-ingress-controller        LoadBalancer  10.7.248.226  pending      
nginx-ingress-default-backend   ClusterIP     10.7.245.75   none         
```



두번째 서비스인 `nginx-ingress-default-backend`를 확인하자.

기본 backend는 모든 URL path들을 다루고, NGINX controller를 호스트하는 서비스다.

기본 backend는 2가지 URL들을 노출한다.

- 200을 return 하는 `/healthz`
- 404를 return하는 `/`



`nignx-ingress-controller` Service가 배포되었는지 확인하자.

```bash
$ kubectl get service nginx-ingress-controller
>
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP      
nginx-ingress-controller   LoadBalancer   10.7.248.226   35.226.162.176   
```

​    

## Configure Ingress Resource to use NGINX Ingress Controller

Ingress Resource Object는 kubernetes SErvice에게 inbound traffic을 라우딩하기 위한 L7 rule들의 집합이다. 다양한 rule은 하나의 Ingress Reousrce에서 정의되거나, 다양한 Ingress Resource manifest파일로 나뉘어질 수 있다. Ingress Resource는 traffic에 사용될 controller를 결정할 수 있다. 이는 Ingress Resource의 metadata section 섹션에 `kubernetes.io/ingress.class`로써 명시될 수 있다. 

NGINX controller를 위해 우리는 nginx value를 아래와 같이 사용한다.

```bash
annotations: kubernetes.io/ingress.class: nginx
```



만약 metadata section에 annotations가 정의되어있지 않다면, GKE에서는 Ingress Resource를 GCP GCLB L7 load balancer를 traffic에 사용하도록 결정한다. 이러한 방법은 아래와 같이 gce라는 annotation value로 강제로 설정된다.

```bash
annotations: kubernetes.io/ingress.class: gce
```



NGINX Ingress Controller에 사용하기 위해서 간단한 Ingress Resource YAML file을 만들자.

```bash
touch ingress-resource.yaml
or
nano ingress-resource.yaml
```



내용은 아래와 같이 작성한다.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-resource
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /hello
        backend:
          serviceName: hello-app
          servicePort: 8080
```



`kind: Ingress`는 해당 yaml파일이 Ingress Resource Object라는 것을 의미한다. 이 Ingress Resource는  `hello-app`을 port`8080`으로 제공하기 위해 Inbound L7 rule을 `/hello`라는 경로에 정의해놨다고 적어놨다.



아래와 같은 명령어를 이용해서 해당 rule을 kubernetes application에 적용한다.

```bash
$ kubectl apply -f ingress-resource.yaml
```



아래의 명령어를 이용해서 Ingress Resource가 생성되었는지 확인한다.

```bash
$ kubectl get ingress ingress-resource
>
NAME               HOSTS     ADDRESS   PORTS     AGE
ingress-resource   *                   80        
```



​     

### Test Ingress and default backend

NGINX ingress controller의 `EXTERNAL-IP/hello` URL을 통해서 web application에 접근할 수 있다.