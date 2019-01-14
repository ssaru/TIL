# Kubeflow



## BIOS CPU Hypervisor Enable

Kubeflow 공식 홈페이지의 [Getting Start](https://www.kubeflow.org/docs/started/getting-started-minikube/)를 통해서 Kubeflow를 설치 도중에 다음과 같은 에러를 만남



>Starting local Kubernetes v1.8.0 cluster...
>
>Starting VM...
>
>E0110 15:27:36.156882 21701 start.go:150] Error starting host: Error getting state for host: getting connection: looking up domain: virError(Code=42, Domain=10, Message='Domain not found: no domain with matching name 'minikube'').



에러를 찾아 떠돌다보니까 minikube의 다음 [이슈](https://github.com/kubernetes/minikube/issues/2991)를 발견했는데, 여기서 3가지 명령어를 확인해달라고 이야기함

```bash
virt-host-validate 
dmesg | grep kvm
lscpu | grep -q Virtualization || echo "No Virtualization"
```



`virt-host-validate`를 통해서 아래와 같은 결과를 확인함

> QEMU: Checking if device /dev/kvm exists                                   : FAIL



BIOS 설정에서 CPU 가상화 Enable 설정

​    

## Install ksonnet

다음 [가이드](https://www.kubeflow.org/docs/guides/components/ksonnet/)를 통해서 ksonnet를 설치해준다.