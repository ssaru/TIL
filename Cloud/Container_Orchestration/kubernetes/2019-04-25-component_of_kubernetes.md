

Kubernetes 컴포넌트에 대해서 알아본다.



# 마스터 컴포넌트

- 클러스터의 컨트롤 플레인(control plane)을 제공한다.

  - 스켸쥴링과 같은 전반적인 결정 수행

  - 클러스터 이벤트 감지 및 반응

    - 레플리카 필드가 요구조건에 충족되지 못했을 때, 새로운 파드(Pods) 구동

- 클러스터 내 어떠한 머신에서든지 동작 될 수 있음

  - 하지만 간결성을 위해서 구성 스크립트는 동일 머신 상에서 모든 마스터 컴포넌트를 구동

    - `kube-apiserver`, `etcd`, `kube-scheduler`, `kube-controller-manager`, `cloud-controller-manager` 관련 모든 컴포넌트를 하나의 머신 상에서 구성한다.

  - 사용자 컨테이너는 해당 머신 상에서 동작시키지 않는다.



![master component](https://docs.microsoft.com/bs-cyrl-ba/azure/aks/media/concepts-clusters-workloads/cluster-master-and-nodes.png)





## kube-apiserver

- 쿠버네티스 API를 노출하는 마스터 상의 컴포넌트

  - 쿠버네티스 컨트롤 플레인에 대한 프론트엔드

  - 수평적 스케일(더 많은 인스턴스를 디플로이하는 것)을 위해 설계됨

- kubectl을 이용해서 pod를 생성할 경우 kubernetes에서 일어나는 작업 패턴은 아래 그림과 같다.

![api server](https://cdn-images-1.medium.com/max/800/0*c2N7STjiWZjCy8we.png))



## etcd

- 모든 클러스터 데이터를 담는 쿠버네티스 뒷단의 저장소

  - 구성관리, 서비스 디스커버리, 분산된 작업을 조정하기 위한 키-값 저장소

  - 쿠버네티스 클러스터, 클러스터 상태를 저장하고 있기 때문에 백업 계획은 필수

- Go로 작성되어있으며, 고 가용성 복제 로그를 관리하기 위해 [Raft consensus algorithm](https://raft.github.io/raft.pdf)을 사용

- etcd의 역활은 아래와 같다

  - key들을 생성하고 제거한다.

  - 클러스터의 헬스확인을 한다.

  - etcd node를 추가하고 제거한다.

  - database snapshot을 생성한다.



## kube-scheduler

- 노트가 배정되지 않은 새로 생성된 파드 감지 후, 구동될 노드를 선택

- 스케쥴링을 위해 고려되는 요소는 아래와 같음

  - 리소스에 대한 개별 및 총체적 요구 사항

  - 하드웨어

  - 소프트웨어

  - 정책적 제약

  - 어피니티(affinity) 및 안티-어피니티(anti-affinity) 명세

  - 데이터 지역성

  - 워크로드 간 간섭

  - 데드라인



## kube-controller-manager

- 컨트롤러를 구동하는 마스터 상의 컨트롤러

- 논리적으로 각 컨트롤러는 개별 프로세스

  - 실제로는 복잡성을 낮추기 위해 모두 단일 바이너리로 컴파일 및 단일 프로레스 내에서 실행

- 컨트롤러는 아래를 포함한다.

  - 노드 컨트롤러

    - 노드가 다운 되었을 때, 통지와 대응에 대해 책임을 가짐

  - 레플리케이션 컨트롤러

    - 시스템의 모든 레플리케이션 컨트롤러 오브젝트에 대해 알맞는 수의 파드들을 유지시키는 책임을 갖는다.

  - 앤드포인트 컨트롤러

    - 엔드포인트 오브젝트를 채운다. (서비스와 파드를 연결시킨다.)

  - 서비스 어카운트 & 토큰 컨트롤러

    - 새로운 네임스페이스에 대한 기본 계정과 API 접근 토큰을 생성



## cloud-controller-manager

- 바탕을 이루는 클라우드 제공 사업자와 상호작용하는 컨트롤러 작동(릴리즈 1.6에서 도입된 알파기능)

- 클라우드 제공사업자 고유의 컨트롤러 루프만을 동작시킴

- kube-controller-manager에서는 해당 컨트롤러 루프를 비활성 해야한다.

  - kube-controller-manager구동 시에 `--cloud-provider`플래그를 `external`로 설정한다.

- 아래 컨트롤러들은 클라우드 제공사업자에 의존성을 갖는다.

  - 노드 컨트롤러

    - 노드가 응답을 멈추고 나서 클라우드 상에서 삭제되었는지 확정하기 위한 확인절차

  - 라우트 컨트롤러

    - 바탕을 이루는 클라우드 인프라에 경로를 구성

  - 서비스 컨트롤러

    - 클라우드 제공사업자 로드밸런서를 생성, 업데이트, 삭제

  - 볼륨 컨트롤러

    - 볼륨 생성, 부착, 마운트 및 볼륨 조정을 위해 클라우드 제공사업자와 상호작용



### REFERENCE

[1. Kubernetes cluster architecture](https://docs.microsoft.com/bs-cyrl-ba/azure/aks/concepts-clusters-workloads)

[2. Kubernetes Master Components: Etcd, API Server, Controller Manager, and Scheduler](https://medium.com/jorgeacetozi/kubernetes-master-components-etcd-api-server-controller-manager-and-scheduler-3a0179fc8186)
