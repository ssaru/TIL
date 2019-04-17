# Comparing kubernetes deployment tools

현재 kubenetes 관련 작업을 하고 있는데, 환경설정을 위해서는 **kubespray**와 **Ansible**에 대한 기본 이해가 있어야함.

관련 지식이 전혀 없는 관계로 차근차근 정리해보고자 함.



원래 목적은 Kubespray를 알기위해서 작성하는 글이나 숲을 보고자 시작은 Kubernetes Deployment tool들을 정리하고 넘어가도록 하겠다.

    

## What Kubernetes deployment tools?

Kubernetes가 오픈소스 진영에서 container application의 scaling, management, deployment를 위한 표즌으로 빠르게 성장했으며, 높은 유연성과 기능들을 제공했다. 하지만 Kubenetes 설치를 위한 방대한 문서는 kubenetes의 가파른 learning curve를 초래했다. 그 외에도 cluster 구축을 위한 설치에도 몇가지 번거로운 작업들이 있었다.   

    

이러한 이유로 이러한 작업들을 쉽게해주는 Kubenetes deployment tool들이 출현하게 된다. Kubenetes deployment tool들은 아래와 같이 여러가지 종류가 있다.

- Kubespray (Kubo)

- Kubeadm

- kops



![1_7IxE5YGoJ0thLLT6QG1d6w](https://user-images.githubusercontent.com/13328380/56263642-b24fd400-611e-11e9-9038-4555f8023b63.png)

    

### Kops

Kops는 *Kubernetes operations*를 의미한다. 표어는 "The easiest way to get a production-grade Kubernetes cluster up and running"이다.  

    

Kops를 통해 Kubernetes Cluster를 "create", "destroy", "upgrade"가 가능하다. AWS를 지원하며 GKE는 beta, VMware vSphere에서는 alpha버전을 지원한다.



Kops의 특징은 아래와 같다.

- Highly Available(HA) Kubernetes Masters

  - HA는 "고 가용성" 즉, *잘 고장나지 않음*을 의미한다.

- A state-sync model for dry-runs and automatic idempotency

  - dry-run이란 시운전/연습을 의미한다.

  - [**Dry run** feature는 Kubernetes v1.13](https://kubernetes.io/docs/reference/using-api/api-concepts/#dry-run)에서 적용되었는데 **POST / PUT / PATCH / DELETE**와 같은 요청을 Dry run mode에서 받아들일 수 있으면서 시스템에는 영향을 주지않게 만들어져있어 시스템에 부작용 없이 테스트를 위한 기능으로 보여진다. *(아직은 정확하게 이해되지 않아서 어떠한 역활을 하는지 모르겠다. )*

  - **Idempotent(멱등성)**는 여러 번 수행을 해도 결과가 같은 경우를 이야기한다. 

  - **Idempotent(멱등성)**연산은 부작동을 일으키지 않으며 *안정성 / 동시성 / 데이터 손실 방지 / 실패로부터 재시도, 복구 기능*을 제공한다.

  - REST API 기준으로 보면 *GET / PUT / DELETE*은* Idempotent 성질을 갖고 *POST*는 Non-idempotent 성질을 갖는데, 앞에 3가지 메소드들은 Status를 갖지 않기 때문에 요청이 실패한 경우, 단순히 재요청을 하면 되지만 *POST* 메소드는 어떠한 값을 변경하기 때문에 Status가 변경되게 된다. 따라서 별도의 트렌젝션 처리가 별도로 필요하게 된다.

  - **Ansible**과 같은 개념인 것 같은데, 결국 Kubernetes의 deploy를 지원하는 툴이니 실행하고 난 결과는 매번 동일한 결과값을 보장한다는 것 같다.

  - 결론적으로 **automatic Idempotent / dry-runs**와 을 위한 state-sync model이라는 feature를 가지고있다는 이야기다.

- Can generate Terraform

  - 테라폼은 인프라스트럭처 관리 도구로써 **Ansible**과 같은 설정관리 도구와 더불어서 프로비저닝 도구로 분류된다. Kops에서 Terraform을 생성하는 기능이 있는 듯 하다.

- Support for custom kubernetes add-ons

- Command line auto-completion

- YAML Manifest Based API Configuration

- Templating and dry-run modes for creating Manifests

- Out-of-the-box support from eight different CNI Networking providers, including Weave Net

  - Linux Container의 Network 설정 Spec을 이야기하는데, Kubernets의 기본 CNI Networking provider인 kubenet은 기본적인 CNI Networking provider이나, 기능이 많지 않다. 이러한 CNI Networking provider들은 여러가지가 있는데 총 8개의 CNI를 지원한다는 것을 의미한다.

- Support for Kube-up upgrades

  - `kube-up.sh`라는 script가 있는 듯 하고, 사용자 플랫폼에서 간편하게 Kubernetes를 사용할 수 있는 설치방법을 제공해주는 듯 하다.

- Ability to add containers, as hooks, and files to nodes via a cluster manifest

     

### Kubeadm

Kubeadm은 이미 존재하는 기존 인프라스트럭처에 Kubernetes cluster를 셋팅하는데 있어 최고의 bootstrapping이지만, Kops와의 주요 차이점은 인프라를 프로비저닝 할 수 없다. 



즉, Kubeadm은 Kubernetes 클러스터를 만들기 위한 최선의 경로를 제공하는게 목적이다. 설계 목적이 그런만큼 프로비저닝 머신이 아닌 부트 스트랩 기능에 초점이 맞춰져있고, Kubernetes 대시보드, 모니터링 솔루션 및 클라우드 특정 애드온과 같이 다양한 애드온을 설치하는 것은 범위에 포함되지 않는다.



Kubeadm의 특징은 다음과 같다.

- Kubeadm is a tool for bootstrapping a best-practice Kubernetes cluster easily

- The user experience should be slick, and the cluster secure

- Kubeadm's scope is limited; intended to be a building block as well

- kubeadm assumes that machines already are provisioned

- Swappable Architecture with everything divided into phases

- Setting up or favoring a specific CNI network is out of scope

- intended audience: build-your-first-own-cluster-on-bare-metal-users & higher-level tools like kops

    

### Kubespray

Kubespray는 Kubernetes 클러스터를 클라우드 혹은 온프레미스환경에 배포하도록 설계되었다. Kubespray는 Ansible playbook 기반으로 만들어진 툴이며 아래와 같은 특징을 갖는다.



- support for deployments on AWS, Google Compute Engine, Microsoft Azure, OpenStack, and bare metal

- deployment of Kubernetes highly available clusters

- a composable architecture with a choice from six network plugins

- support for a variety of Linux distributions

- support for continuous integration tests

- kubeadm under the hood



### Reference

[1. Kubespray – 10 Simple Steps for Installing a Production-Ready, Multi-Master HA Kubernetes Cluster](https://dzone.com/articles/kubespray-10-simple-steps-for-installing-a-product)

[2. How to Install Kubernetes Cluster with Ansible based tool Kubespray](https://linoxide.com/containers/install-kubernetesk8s-cluster-ansible-based-tool-kubespray/)

[3. Install Kubernetes on bare-metal CentOS7](https://itnext.io/install-kubernetes-on-bare-metal-centos7-fba40e9bb3de)

[4. A Multitude of Kubernetes Deployment Tools: Kubespray, kops, and kubeadm](https://www.altoros.com/blog/a-multitude-of-kubernetes-deployment-tools-kubespray-kops-and-kubeadm/)

[5. kubespray official document - Comparison](https://kubespray.io/#/docs/comparisons)

[6. Deploy a Kubernetes Cluster on OpenStack using Kubespray](https://itnext.io/deploy-a-kubernetes-cluster-on-openstack-using-kubespray-39b230b13d62)

[7. 서버 이중화(HA) 솔루션에 대해서(RoseHA)](https://www.sharedit.co.kr/posts/53)

[8. REST API의 이해와 설계 #1 - 개념 소개](https://bcho.tistory.com/953)

[9. REST - IDEMPOTENCY AND SAFETY](http://pradeeploganathan.com/rest/rest-idempotency-safety/)

[10. 멱등성(Idempotent) 이해하기](https://knight76.tistory.com/entry/ansible-%EB%A9%B1%EB%93%B1%EC%84%B1idempotent-%EC%9A%A9%EC%96%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

[11. DB기초 - 트랜잭션이란 무엇인가?](https://coding-factory.tistory.com/226)

[12. 테라폼(Terraform) 기초 튜토리얼](https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code#%EB%93%A4%EC%96%B4%EA%B0%80%EB%A9%B0-infrstructure-as-code-%EB%8F%84%EA%B5%AC-%ED%85%8C%EB%9D%BC%ED%8F%BCterraform)

[13. Choosing a CNI Network Provider for Kubernetes](https://chrislovecnm.com/kubernetes/cni/choosing-a-cni-provider/)

[14. CNI (Container Network Interface)](https://ssup2.github.io/theory_analysis/CNI/)

[15. Alternatives to kebe-up.sh](https://www.oreilly.com/library/view/getting-started-with/9781787283367/f46795f6-3f97-40af-8e1b-35fee9651d23.xhtml)

[16. Intro to Kops: Kubernetes Operations and How to Get Started Managing Kubernetes in Production](https://medium.com/devopslinks/intro-to-kops-kubernetes-operations-how-to-get-started-managing-kubernetes-in-production-4bdb23f019b6)

[17. 프로비저닝](https://asfirstalways.tistory.com/279)
