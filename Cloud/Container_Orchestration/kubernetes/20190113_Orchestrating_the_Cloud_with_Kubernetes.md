# Orchestrating the Cloud with Kubernetes - (KUBERNETES IN THE GOOGLE CLOUD 1주차)



이번 세션에서는 3장을 리뷰한다.

​    

## 빠른 Kubernetes 데모

`kubectl run`명령을 이용해서 kubernetes를 빠르게 사용할 수 있다.

이를 이용하여 nginx 컨테이너의 단일 인스턴스를 실행한다.

```bash
$ kubectl run nginx --image=nginx:1.10.0
```



이를 이용하여 kubernetes에서 deployment 객체를 만들었다. 

kubernetes에서 모든 컨테이너는 한 pod에서 실행된다. 실행 중인 nginx 컨테이너를 확인하려면 `kubectl get pods` 명령어를 이용한다.

```bash
$ kubectl get pods
```



`kubectl expose` 명령을 이용하여 nginx 컨테이너를 kubernetes 외부로 노출할 수 있다.

```bash
$ kubectl expose deployment nginx --port 80 --type LoadBalancer
```

- 해당 명령을 이용하면 백그라운드에서 Kubernetes가 Public IP 주소가 연결된 LoadBalancer를 만든다.

- Public IP에 도달한 Client는 서비스 뒤에 있는 Pod로 Route 된다.

  *(해당 경우는 nginx 포드로 Routing 된다.)*



`kubectl get services`명령을 사용하여 서비스 리스트를 확인해보자

```bash
$ kubectl get servicees
```



curl http 명령을 이용하여 external IP를 원격으로 nginx 반응을 출력해보자

```bash
$ curl http://<External IP>:80
```

​    

## Pods

- Pod는 kubernetes의 핵심 요소이다.

- Pod는 하나 이상의 컨테이너가 포함된 집합을 의미하고, 의존관계가 높은 컨테이너끼리 단일 pod로 패키징 한다.

- Pod에는 볼륨이 존재하며, 해당 볼륨은 포드가 활성화되어있어야지만 사용가능한 데이터 디스크로서 활용될 수 있다. pod는 컨텐츠에 대한 공유 네임스페이스를 제공한다. 즉, pods에 속한 두 컨테이너가 서로 통신할 수 있으며, 연결된 볼륨을 공유한다.

- pods는 네임워크 스페이스도 공유한다. 즉, pods마다 IP 주소가 하나씩 있다.



![56124565adb6c28b](https://user-images.githubusercontent.com/13328380/51085557-9aaf2b00-177e-11e9-8914-ce89f7c06d81.png)



### Create Pod

pod 구성 파일을 사용하여 포드를 만들 수 있다.

먼저 monolith pod파일의 구성을 보자

```bash
$ cat pods/monolith.yaml
>>
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```



- pod는 하나의 컨테이너로 구성된다.
- 시작시, 파라미터를 컨테이너에 전달한다.
- http 트래픽을 위하 포트 80을 연다.



이제 `kubectl`을 이용해서 monolith pod를 생성해보자

```bash
$ kubectl create -f pods/monolith.yaml
```



이제 `get pods`와 `describe pods`명령어를 이용해서 실행중인 포드와 포드 정보를 살펴보자

```bash
$ kubectl get pods
$ kubectl describe pods monolith
```

​    

### pod의 상호작용

#### 포트포워딩

pod는 기본적으로 private IP주소가 할당되며, 클러스터 외부에 도달할 수 없다. 따라서 포트포워딩을 통해서 local 포트를 monolith pod의 포트로 맵핑을 해주어야한다.

다음과 같은 명령어를 이용하여 포트포워딩을 해줄 수 있다.

```bash
$ kubectl port-forward monolith 10080:80
```



포트포워딩이 완료되었다면, 새로운 터미널을 이용해서 통신을 시도하면 통신이 잘 되는 것을 확인할 수 있다.

```bash
$ curl http://127.0.0.1:10080
```



#### Logs

`kubectl logs` 명령어를 이용하여 해당 pod의 로그를 확인할 수 있다.

```bash
$ kubectl logs -f monolith
```



만약 해당 로그를 실시간으로 받아보고 싶다면(로그 스트림), 다음과 같은 명령어를 실행한다.

```bash
$ kubectl logs -f monolith
```



#### Access bash at pod

`kubectl exec` 명령을 이용하여 monolith pod의 bash에 접속할 수 있다

```bash
$ kubectl exec monolith --stdin --tty -c monolith /bin/sh
```



사용을 완료했다면, `exit` 명령어를 이용하여 로그아웃 한다.

```bash
$ exit
```



​    

## 서비스

Pod는 영구적인것이 아니다. 어떤 상황에 따라서 Pod가 생성되거나 추가되기도 하고, 사라지기도 한다. 매번 이렇게 생성되고 추가되서 사라지기 때문에, pod의 IP를 이용해서 통신을 하려고하면 문제가 생길 수 있다. 이를위해서 사용하는것이 서비스이다. 서비스는 label을 이용해서 pod간 안정적인 엔드포인트를 제공한다.



![b8b7a35f34a77aec](https://user-images.githubusercontent.com/13328380/51085671-fb8b3300-177f-11e9-8905-2e0381fb71d9.png)



서비스가 pod 집합에 제공하는 접근 수준은 서비스 유형에 따라서 달라진다.

현재는 3가지 유형이 지원된다.

- `ClusterIP(내부)`
  - 기본 유형으로서 이 서비스는 클러스터 내부에서만 볼 수 있다.
- `NodePort`
  - 클러스터 각 노드에 외부 액세스가 가능한 IP를 제공한다.
- `LoadBalancer`
  - 서비스에서 포함된 노드로 트래픽을 전송하는 클라우드 제공업체의 부하 분산기를 추가한다.







