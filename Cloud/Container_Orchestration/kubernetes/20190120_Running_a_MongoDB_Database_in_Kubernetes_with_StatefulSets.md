# Running a MongoDB Database in Kubernetes with StatefulSets - (KUBERNETES IN THE GOOGLE CLOUD 2주차)



이번 세션에서는 6강을 리뷰한다.

​    

## Overview

StatefulSet Object를 시용해서 Kubernetes Engine에서 MongoDB replicaSet를 실행하는 방법에 대해서 알아본다.

​    

## Setting up

이번시간에는 MongoDB와 Kubernetes 클러스터를 통합하는 방법에 대해서 알아볼 것이다. 데이터가 높은 가용성과 중복성을 갖도록 Replica Set을 이용할 것이다. 이러한 성질은 제품 레벨의 어플리케이션에서는 필수적이다. MongoDB의 Replica Set를 설정하기 위해서는 다음과 같은 절차가 필요하다.

- MongoDB [replica set/sidecar](https://github.com/thesandlord/mongo-k8s-sidecar.git)를 다운로드한다.

- [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/)를 인스턴스화 한다.
- [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) 인스턴스화 한다.
- [StatefulSet](http://kubernetes.io/docs/concepts/abstractions/controllers/statefulsets/) 인스턴스화 한다.



1. 다음 Github repository에서 MongoDB/Kuberntes replicaset을 clone한다.

   ```bash
   $ git clone https://github.com/thesandlord/mongo-k8s-sidecar.git
   $ cd ./mongo-k8s-sidecar/example/StatefulSet/
   ```

​    

### Create the StorageClass

`StorageClass`는 kubernetes에게 database node로 어떤 종류의 storage를 사용하기 원하는지 이야기해준다. GCP에서는 [다음](https://cloud.google.com/compute/docs/disks/)과 같이 Storage를 선택할 수 있다.



`StagefulSet` 디렉토리를 확인해보면 Azure와 GCP를 모두를 위한 SSD / HDD 설정을 확인할 수 있다. 

```bash
$ cat googlecloud_ssd.yaml
>
#       Copyright 2017, Google, Inc.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http:#www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

이러한 설정은 SSD volume에 의해 지원되는 "fast"로 불리는 새로운 StorageClass를 만든다.



```bash
$ kubectl apply -f googlecloud_ssd.yaml
```



이제 StorageClass가 설정되었으니 우리는 StatefulSet에서 자동으로 만들어지는 volume을 요청할 수 있다.

​    

## Deploying the Headleess Service and StatefulSet

### Find and inspect the files

headless service와 StatefulSets을 본격적으로 살펴보기 전에 `mongo-statefulset.yaml` 설정파일을 확인해보자

```bash
$ cat mongo-statefulset.yaml
>
apiVersion: v1   <-----------   Headless Service configuration
kind: Service
metadata:
  name: mongo
  labels:
    name: mongo
spec:
  ports:
  - port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    role: mongo
---
apiVersion: apps/v1beta1    <------- StatefulSet configuration
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"
  replicas: 3
  template:
    metadata:
      labels:
        role: mongo
        environment: test
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: mongo
          image: mongo
          command:
            - mongod
            - "--replSet"
            - rs0
            - "--smallfiles"
            - "--noprealloc"
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-persistent-storage
              mountPath: /data/db
        - name: mongo-sidecar
          image: cvallance/mongo-k8s-sidecar
          env:
            - name: MONGO_SIDECAR_POD_LABELS
              value: "role=mongo,environment=test"
  volumeClaimTemplates:
  - metadata:
      name: mongo-persistent-storage
      annotations:
        volume.beta.kubernetes.io/storage-class: "fast"
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 100Gi
```

​    

#### Headless service

`mongo-statefulset.yaml`의 첫번째 섹션은 `headless service`를 참고한다. Kubernetes 용어로 **service**는 특정한 pods에 접근하기 위한 정책이나 역활로 설명된다. 간단하게 이야기해서 `headless service`란 load balancing에 대해서 정책이 없는 것 중 하나다. **StatefulSet**과 함께 섞이게되면 `headless service`는 pod들이 접근할 수 있는 개별적인 DNS들을 우리에게 준다. 그리고 차례대로 모든 MongoDB 노드에 개별적으로 연결할 수 있게 해준다. `yaml`파일에 clusterIP 필드가 `None`으로 설정되어있는 것을 확인하여 Service가 headlss인지 검증할 수 있다.

​     

#### StatefulSet

StatefulSet 설정은 `mongo-statefulset.yaml`파일의 두번째 섹션에 있다. StatefulSet은 마치 어플리케이션에 있어서 빵과 버터같은 사이이다. StatefuleSet은 MongoDB를 실행하는 workload이고, Kubernetes resource들을 조율한다. `yaml`파일을 확인하면 우리는 첫번째 섹션에서 StatefuleSet Object의 설명을 확인할 수가 있고, metadata 섹션으로 내려가다 보면 label과 특정한 replicaset의 개수가 지정되어있는 것을 확인할 수 있다.



그 다음으로는 pod의 spec에 대한 내용이 오는데 `terminationGracePeriodSeconds`는  replica의 수가 scale down될 때, pod를 정상적으로 shutdown하는데 사용된다. 



그 다음으로 곧바로 2개의 컨테이너를 위한 설정을 볼 수 있다.

첫째로는 MongoDB를 실행하는데 필요한 replica set 이름을 설정하는 command line flag이며, 추가적으로 persistent storage volume을 `/data/db`로 마운트하는 command line flag도 존재한다. `/data/db`에는 MongoDB의 data가 위치하게 된다.

두번째로는 sidecar를 실행하는 컨테이너에 대해서 명시가 되어있다. 이 [sidecar container](https://github.com/cvallance/mongo-k8s-sidecar)는 MongoDB replica set을 자동적으로 설정할 것이다. 앞서 설명했듯이 "sidecar"는 메인 컨테이너가 여러 task를 수행하는데 도움을 주는 컨테이너다



마지막으로 `VolumeClaimTemplates`가 있는데, `volumeClaimTemplates`는 volume을 provision하기 위해서 생성한 StorageClass와 관련이 있다. 해당 명세는 100 GB disk를 각 MongoDB replica에 provision한다.

​    

#### Deploy Headless service and the StatefulSet

Headless service와 StatefulSet에 대해서 대략적으로 이해했으니, 이제 Headless service와 StatefulSet을 배포하보자. 



다음 명령어를 이용해서 배포작업을 진행할 수 있다.

```bash
$ kubectl apply -f mongo-statefulset.yaml
>
service "mongo" created
statefulset "mongo" created
```

​    

## Connect to the MongoDB ReplicaSet

이제 우리는 클러스터를 작동시키고 우리의  replica set을 배포했다. 이제 이를 연결해보자

​    

### Wait for the MongoDB replica set to be fully deployed

Kubernetes StatefulSets은 각각의 pod를 순차적으로 배포한다. 따라서 StatefulSet 배포는 MongoDB replica set member가 모두 부팅되고 다음 MongoDB replica Set meember가 시작되기전에 backing disk를 만드는 모든 과정을 기다린다.



다음 명령을 통해서 member가 모두 정상적으로 올라왔는지 확인할 수 있다.

```bash
$ kubectl get statefulset
>
NAME      DESIRED   CURRENT   AGE
mongo     3         3         3m
```

​    

#### Initiating and Viewing the MongoDB replica set

현재 시점에 우리는 cluster안에 3개의 pod를 가지고있을 것이다. 이 3개의 노드는 MongoDB replica set과 일치한다. 



다음 명령어를 통해서 이를 확인해보자

```bash
$ kubeectl get pods
>
NAME        READY     STATUS    RESTARTS   AGE
mongo-0     2/2       Running   0          3m
mongo-1     2/2       Running   0          3m
mongo-2     2/2       Running   0          3m
```

모든 멤버들이 Running STATUS가 될 때까지 기다렸다가 다음으로 넘어가자



첫번째 replica set 멤버에 접속해보자

``` bash
$ kubectl exec -ti mongo-0 mongo
```

MongoDB에 연결된 것을 통해서 이제 우리는 `REPL`환경에 진입했다.



`rs.initiate()`명령을 실행해서 얻을 수 있는 기본 설정으로 replica set을 인스턴스화 하자

```bash
$ rs.initiate()
>
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "localhost:27017",
        "ok" : 1,
        "operationTime" : Timestamp(1547982309, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1547982309, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
```



replica set의 설정을 `rs.conf()`명령어로 확인할 수 있다.

```bash
$ rs.conf()
>
rs0:PRIMARY> rs.conf()
{
        "_id" : "rs0",
        "version" : 1,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "localhost:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5c4455e515e5ab7d52a53f86")
        }
}
```



출력값은 replica set  `rs0`의 현재 member에 대한 상세한 내용을 보여준다. 이번 강의에서는 오직 하나의 멤버에 대해서 정보를 확인할 수 있지만, 모든 멤버들의 정보를 확인하고싶다면 [nodeport ](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) 나 [load balancer](https://cloud.google.com/compute/docs/load-balancing/http/)와 같은 추가적인 service를 통해서 replica set를 노출시켜야한다.



`exit`명령어로 `REPL`환경에서 빠져나오자

​    

## Scaling the MongoDB replica set

kubernetes와 StatefulSet의 큰 이점은 MongoDB replica들의 수를 하나의 명령어로 scale up하거나 down할 수 있다는 것읻다.



replica set의 수를 3에서 5개로 scale up해보자.

```bash
$ kubectl scale --replicas=5 statefulset mongo
```



몇분 후면, 5개의 MongoDB pod를 확인할 수 있다.

```bash
$ kubectl get pods
>
NAME      READY     STATUS    RESTARTS   AGE
mongo-0   2/2       Running   0          41m
mongo-1   2/2       Running   0          39m
mongo-2   2/2       Running   0          37m
mongo-3   2/2       Running   0          4m
mongo-4   2/2       Running   0          2m
```



이제 5개에서 3개로 scale down작업을 진행해보자

```bash
$ kubectl scale --replicas=3 statefulset mongo
$ kubectl get pods
>
NAME      READY     STATUS    RESTARTS   AGE
mongo-0   2/2       Running   0          41m
mongo-1   2/2       Running   0          39m
mongo-2   2/2       Running   0          37m
```

​    

## Using the MongoDB replica set

Headless Service에 의해 지원되는 StatefulSet안의 각각의 pod는 안정적인 DNS 이름을 가지고있다. 템플릿은 `<pod-name>.<service-name>`형태를 따른다.



이는 MongoDB replica set을 위한 DNS name이 다음과 같다는 것을 의미한다.

```bash
mongo-0.mongo
mongo-1.mongo
mongo-2.mongo
```



우리는 이런 이름을 우리의 어플리케이션에서 [connection string URI](http://docs.mongodb.com/manual/reference/connection-string)으로 바로 사용할 수 있다.



데이터베이스 사용에 대한 실습은 이번 강의의 범위를 넘어가지만 connection string URI는 다음과 같다.

```bash
"mongodb://mongo-0.mongo,mongo-1.mongo,mongo-2.mongo:27017/dbname_?"
```

​    

## Clean up

배포된 resource들은 다음과 같이 StatefulSet, Headless Service, provisioned volume들을 삭제해주는 명령을 통해서 지울 수 있다

**StatefulSet 삭제**:

```bash
$ kubectl delete statefulset mongo
```



**service 삭제**:

```bash
$ kubectl delete svc mongo
```



**Volume들 삭제**:

```bash
$ kubectl delete pvc -l role=mongo
```



**Cluster 삭제**:

```bash
$ gcloud container clusters delete "hello-world"
```

