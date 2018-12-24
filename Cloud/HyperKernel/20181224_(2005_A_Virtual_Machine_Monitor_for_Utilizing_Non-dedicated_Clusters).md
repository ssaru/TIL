# A Virtual Machine Monitor for Utilizing Non-dedicated Clusters

​    

## ABSTRACT

non-dedicated cluster를 위한 virtual machine monitor(VMM)을 설계하고 구현함. VMM은 commodity cluster위에서 shared-memory multi-processor machine을 가상화한다. 본 논문에서 설계하고 구현한 VMM은 물리 하드웨어의 설정의 동적변화를 숨긴다. 

(*commodity cluster는 여러 머신들의 집합을 이야기하는 듯 합니다.*)

​    

## INTRODUCTION

non-dedicated commodity cluster들을 효율적으로 사용하는 것은 어렵다. non-dedicated cluster의 resource들은 다수의 사용자와 공유되므로, 어플리케이션에 사용할 수 있는 가용한 양적 / 질적 리소스는 지속적으로 변화한다. 병렬 프로그램이 이러한 동적 변경에 적응하는 것이 중요해지더라도 병렬 프로그램을 작성하는 것은 여전히 많은 노력이 필요하다. 그 결과로 리소스 가용성에 대한 동적변화는 cluster들의 wide deployment 부분에서 굉장히 큰 문제(burdens)가 되었다.



이러한 문제로 인해서 우리는 VMM을 개발했다. 기존의 존재하던 VMM와 같이 우리가 개발한 VMM은 하드웨어의 완벽한 제어와 완벽하게 물리서버와 같은 virtual machine을 만든다.



기존의 존재하던 VMM과 차이점을 이야기하면, 크게 2가지 특징이 있다.



1. cluster들의 shared-memory multi-processor machine을 가상화한다.

   : N개의 Single processor을 N-way의 단일 multi-processor처럼 보이게 한다.

2. 물리 서버 설정의 변화를 숨긴다.

   : 동적으로 물리서버가 추가 및 제거되어도 외부적으로는 고정된 multi processor의 갯수를 제공한다.



![1545636797967](C:\Users\KETI-Windows-Martin\AppData\Roaming\Typora\typora-user-images\1545636797967.png)



이러한 특징으로 인해서 본 논문의 시스템(VMM)은 non-dedicated cluster에서 사용이 간단해지고, 소스코드의 변화없이 병렬 프로그램에서 좋은 성능을 달성할 수 있다.

​    

## IMPLEMENTATION OF THE VMM

본 논문의 VMM은 x86 아키텍쳐에서 설계되었다. processor들, shared memory, I/O device의 가상화는 다음을 따른다.



- processor의 가상화를 위해 instruction set 아키텍쳐의 반 가상화를 지원한다. [1, 4]

- Guest OS는 가상머신에서 최적화되어 실행되기 위해서 정적으로 수정되었다.

- 가상 processor의 갯수를 고정하여 제공하기 위해서 VMM은 하나 이상의 가상 processor와 물리 processor를 mapping하고 이를 동적으로 변경한다.

  *(이러한 동적 mapping은 가상 processor들의 비대칭적인 속도의 원인이 될 것이므로, load balancing을 위해 time ballooning technique[8]을 사용한다.)*

- shared memory의 가상화를 위해서 VMM은 distributed shared memory software와 유사한 방법을 사용한다.

  *(VMM은 공유 메모리의 consistency protocol을  물리 서버의 virtual memory page protection 기법과 함께 구현했다. 현재 consistency 알고리즘은 Ivy[6]를 기반으로 하고 있다.)*

- I/O device의 가상화를 위해 VMM은 모든 device의 상태를 지속적으로 추적하는 중앙 서버 준비한다. 

  *(VMM은 virtual processor가 I/O operation을 실행할 때마다 server와 통신한다.)*

​    

  ## CURRENT STATUS

본 논문의 VMM의 prototype을 구현하였고, 실험을 진행하였다. 현재 구현체는 Linux kernel 2.4 for SMP를 host로 하는 8개의 물리 머신을 이용하여 가상 8-way multi-processor machine 구현하였고, 최적화가 안되어있는 병렬 task를 구현한 가상 8-way multi-processor에서 실행하여 실행시간을 측정하는 것을 통해서 우리의 접근에 대한 가능성을 확인했다.



가상 8-way multi-processor에서 fibonacci 수를 물리서버 1~8까지 증설하면서 개발적으로 실험했다. Figure 2는 실험 결과를 보여준다. `fib(n)`은 n번째 fibonacci 수를 구하는 것을 의미한다. 



실험결과를 확인하면, `fib(46)`에서 8-way multi-processor machine은 1-way processor machine보다 빠르다는 것을 확인했다.



![figure2](https://user-images.githubusercontent.com/13328380/50393870-7faf5380-079c-11e9-9c5b-ba3245d5733d.PNG)



​    

## RELATED WORK

`vNUMA`[2]는 Itanium 아키텍쳐로 구성된 물리서버위에 `cc-NUMA`machine을 가상화한 것이다. `vNUMA`와 본 논문의 VMM과의 주 차이점은 `물리 서버의 동적 추가 및 제거를 지원하지 않는다는 것`이다. 



`Virtual Iron`[9]은 cluster위에 multi-processor machine을 가상화한다. `Virtual Iron`은 물리서버의 동적 변경을 허용한다. `Iron`의 기본적인 방법은 본 논문의 VMM과 유사하나, 아직 구현체가 없다.(2005/10/14일 기준)



​    

## FUTURE WORK

- 시스템의 확장(물리서버 개수의 확장) 및 개선

  *(checkpointing/recovery[3]과 replication for VMMs[7]등의 방법을 이용한 fault 허용 기법 등등)*

- memory consistency 알고리즘의 개선

  *(IA-32 memory model[5]과 같은 processor의 동기명령이나 instruction set 실행 명령이 오기전까지 memory 쓰기를 지연하는 방법)*

- SPLASH-2나 Apache같은 실제 세계에서의 테스트 진행



More information can be found at [here](http://web.yl.is.s.u-tokyo.ac.jp/~kaneda/dvm/)



​    

## REFERENCE

[[1] Xen and Art of Virtualization](http://www.cs.yale.edu/homes/yu-minlan/teach/csci599-fall12/papers/xen.pdf)

[[2] Implementing Transparent Shared Memory on Clusters Using Virtual Machines](http://unsworks.unsw.edu.au/fapi/datastream/unsworks:4551/SOURCE1)

[[3] A Survey of Rollback-recovery Protocols in Message-passing Systems](https://www.cs.utexas.edu/~lorenzo/papers/SurveyFinal.pdf)

[[4] Running BSD Kernels as User Processes by Partial Emulation and Rewriting of Machine Instructions](https://www.researchgate.net/publication/41035932_Running_BSD_kernels_as_user_processes_by_partial_emulation_and_rewriting_of_machine_instructions/download)

[[5] IA-32 Intel Architecture Software Developer’s Manual Volume 3: System Programming Guide](http://flint.cs.yale.edu/cs422/doc/24547212.pdf)

[[6] Memory Coherence in Shared Virtual Memory Systems](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.80.5091&rep=rep1&type=pdf)

[[7] Hypervisor-based fault tolerance](http://courses.mpi-sws.org/ds-ws18/papers/bressoud-hypervisor.pdf)

[[8] Towards Scalable Multiprocessor Virtual Machines](https://www.usenix.org/legacy/event/vm04/tech/full_papers/uhlig/uhlig.pdf)

[[9] Virtual Iron Software](http://www.virtualiron.com/)