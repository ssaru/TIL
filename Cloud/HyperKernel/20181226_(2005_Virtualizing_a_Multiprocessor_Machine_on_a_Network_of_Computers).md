# Virtualizing a Multiprocessor Machine on a Network of Computers

​    

## INTRODUCTION

Grid 컴퓨팅의 기본적인 목적은 여러 컴퓨터에 분산되어있는 이기종 자원을 완벽하게 다중화하는 것이다. 예를들어 Globus toolkit[8]은 cpu 아키텍쳐나 os시스템같이 다양한 하드웨어/소프트웨어 설정을 활용할 수 있는 middleware를 구축하는데 있다.  또 다른 예로는 사용자가 기여한 유후 cpu자원을 활용하여 외계지능을 찾아내는 SETI@home [1]이 될 수 있다.



Grid 컴퓨터환경(Computational Grid)을 개발하는 것의 주요 이슈는 사용자화(customizable)가 가능하고, 안전한 실행환경을 제공하는 것이다. 다양한 하드웨어/소프트웨어 설정은 특정한 os나 라이브러리에 의존성이 있는 인기있는 프로그램을 실행하기 어렵기 때문에 사용자화(customizable) 플랫폼은 꼭 필요하다. 또한 os에 보완적인 격리 및 보안 메커니즘을 제공하는 것은 사용자에 의해 실행되는 응용프로그램은 기본적으로 신뢰할 수 없고 시스템을 파괴할 수 있기 때문에 중요하다.



Grid 컴퓨터 환경(Computational Grid)을 개발하는 방법 중에 기존의 os컨셉을 따르며 자원 공유를 기본으로 하는 virtual machine monitor(VMM)[13]이 있다. VMM은 실제 컴퓨터에 있는 하드웨어 레이어(프로세스, 메모리, I/O device)를 가상화하고, 실제 단일의 컴퓨터처럼 보이게끔 만드는 virtual machine으로 추출한다. VMM은 다음과 같은 이유로 Grid 컴퓨터환경 개발을 촉진하는 요소가 된다.

- VMM은 사용자가 물리서버와 디-커플링된 사용자화된(customized) 응용프로그램 환경에 접근할 수 있도록 허용한다.
- VMM은 신뢰할수 없는 사용자 혹은 어플리케이션이 가상환경내에서 OS만 손상시킬 수 있으며 물리적인 시스템을 손상시킬수 없음을 보장한다.



최근 몇몇 연구집단은 Grid Computing 가상머신 구현방법에 대해서 탐구하기 시작했다 [4, 5, 9, 12]. 예를들어 VM-Plant[9]는 자동으로 유연한 VMs를 생성하고 설정하는 것에 의해 이기종 분산 Grid 컴퓨팅 실행 환경을 지원하는 Grid 서비스다. VM-Plant와 같은 현존하는 시스템들은 Grid 환경에서 병렬 어플리케이션을 위한 개별적인 실행환경을 지원하는 Grid 환경에서 feasiblity 확인이 되지 않았다. 좀 더 자세하게 이야기하자면, 이러한 시스템은 스스로 job submission이나 job scheduling mechanism과 같은 병렬컴퓨팅을 위한 특별한 프레임워크로서는 부족하다는 의미다.  따라서 이러한 middleware들을 사용하게되면 VMM의 추상화가 깨지고, VMM이 제공하던 리소스 관리의 간단성이 깨지게 되므로 다양한 호스트들은 병렬프로그램을 실행하는데 어려움에 직면하게 된다.



이러한 문제를 해결하기 위해서 우리는 VMM이 제공하는 추상화 수준은 지키면서 병렬컴퓨팅이 가능한 환경을 제안한다. 본 논문에서는 네트워크 컴퓨터 상의 멀티 프로세서 머신을 에뮬레이트하는 소프트웨어 레이어를 제안한다. 현존하는 VMM과 같이 이 소프트웨어 레이어는 머신의 하드웨어 컨트롤을 완벽하게 취하고, 독립적으로 단일의 물리 머신에서 돌아가는 OS와 같이 보이도록하는 가상머신을 생성한다. 현존하는 VMM들과는 다르게 우리 시스템은 단일 프로세서 머신들의 콜렉션 위에서 가상 멀티 프로세서 머신을 생성한다. 예를들어 실제로 네트워크에 의해 N개의 단일 프로세서로 묶여있는 시스템이지만 사용자에게는 N개의 CPU를 가지고있는 멀티 프로세서 머신이라고 생각할 수 있게 해준다.  기능적인 우리의 시스템은 굉장히 사용하기 간단한 분산 리소스가 된다. 예를들어 단순하게 대용량 멀티 프로세서 시스템에서 사용되는 응용 어플리케이션을 가져와서 그것보다 더 작고 저렴한 시스템에서 실행할 수 있다. 



사용자가 2개의 단일 프로세서 머신을 가지고있다고 가정하자.  사용자는 쉽게 가상의 듀얼 프로세서를 가진 머신을 VMM을 통해서 만들 수 있다. 만약 가상머신 안에서 돌고있는 게스트 OS에 사용자가 프로세스를 포크하는 경우, 이러한 프로세스는 게스트 OS의 스켸쥴러에 의해서 가상의 멀티 프로세서에 할당되며, 최종적으로 VMM에 의해서 다수의 물리머신 위에 할당된다. 본 논문에서의 프로토타입의 설명 및 평가는 인텔 펜티엄 아키텍쳐 위의 Linux gues operating system에서 작동한다.



본 논문의 구성은 다음과 같다.

- Section 2에서는 시스템의 기본적인 디자인을 설명하고, 
- Section 3는 VMM의 구현체에 대해서 설명한다.
- 마지막 Section은 논문 요약이다.

​    

## Basic Design

해당 섹션에서는 본 논문에서 제안한 시스템의 디자인에 대해 설명한다.  섹션은 다음과 같이 2가지 파트로 나뉘게된다.



- 제안한 VMM이 제공하는 가상화의 특징에 대해서 언급
- 물리 리소스가 가상 리소스에 어떻게 맵핑되는지와 시스템의 아키텍쳐 소개

​    

### Virtual Machine Interface

앞서 이야기했지만, VMM은 하나의 OS에서 작동하는가상의 멀티 프로세서 머신을 만든다. 가상화 특징을 요약하자면 다음과 같다.



- 가상머신은 Intel Pentium architecture의 instruction set architecture(ISA)를 가상화한다.

- VMM은 모든 물리머신의 전체 가상화를 지원하지 않는다. 그러나 반 가상화를 지원한다. 다른말로 표현하자면 guest OS 추상화에 의해 사용되는 instruction set은 물리적인 하드웨어와 유사하지만 같지는 않다. 이러한 방식은 성능 개선을 보장하나 guest os의 수정을 요구한다. 이러한 gest os의 상세 수정내용은 확인이 필요하며, 이는 섹션 3에 소개되어있다.
- 가상머신은 가상머신이 실행되고있는 물리적 머신의 콜렉션과 같은 같은 수의 프로세서를 갖는다.



Intel Pentium Architecture들의 다양한 프로세서 사이에서 우리는 Pentium 4, Intel Xeon과 Intel 486 프로세서 같은 다른 Intel Petium Architecture보다 조금 더 relaxed memory ordering model을 허용하는 P6 family 프로세서를 목표로 했다 [2]. 현재 구현체는 오직 Linux/x86 호스트환경에서만 구동된다. 하지만 이러한 기술을 이용하여 다른 OS 시스템에서도 적용할 수 있을 것이다.

​    

### System Architecture



![fig1](https://user-images.githubusercontent.com/13328380/50464002-f3867180-09d1-11e9-91b0-b5becedc12f0.PNG)



본 논문의 시스템은 host os의 위에 VMM이 위치하는 *hosted* architecture[3] 이다. VMM이 사용자 host os의 user process로 실행되는 현존하는 hosted architecture기반의 VMM과 같다.(LilyVM[6], FAU machine [7])

이러한 hosted architecture는 VMM이 배어메탈에 직접적으로 위치하는[10]것에 비교했을 때, 큰 오버헤드가 발생하나 몇몇 기술적/구현적인 난관을 극복하였다.



- hosted architecture는 [11]에서 보여준 자연스럽지 못한 가상화인 Intel Pentium Architecture의 가상화에 대해서 유용하다.
- 해당 architecture는 가상머신이 굉장히 다양한 주변 장치를 최소한의 프로그래밍적 노력을 host operating system을 통해서 지원하게 해준다.
- 해당 architecture는 guest os가 이전의 host operating system과 공존할 수 있게 해준다.



본 논문의 시스템의 이러한 구별되는 특징은 네트워크로 연결된 머신 위에서 다수의 VMM 프로세스가 실행되는 것에 의해 멀티 프로세스 머신의 가상화를 가능하게해준다. 현존하는 시스템인 단일 VMM은 물리머신과 동시에 실행된다. 반면 본 시스템은 다수의 머신에서 실행되는 VMM이 서로 협력하여 가상 컴퓨터를 만든다.

가상화된 리소스(프로세스, 메모리, I/O 장치)들은 다음과 같은 방법을 통해서 물리적 리소스와 맵핑된다.



- 가상 프로세스는 1:1 방식으로 물리적 프로세서와 맵핑된다.

  예를들어, Figure 1처럼 4개의 프로세서가 물리머신에 있다면 가상머신에서도 4개의 프로세서로 나타나게된다.

- 메모리 및 I/O device는 하나의 가상머신에 의해 공유된다.

​    

## Implementation of the VMM

이번 섹션에서는 VMM이 어떻게 프로세서, 공유 메모리, I/O device를 가상화 했는지 설명한다. 프로세서의 가상화는 단일 프로세서 머신과 같으며 주요 주제와는 무관하다. 여기서는 mechanism의 개요만 볼 예정이다. 그 이후에는 다른 하드웨어 자원의 가상화가 단일 프로세서 시스템의 가상화와 어떻게 다른지 자세히 설명한다.

​    

### Virtualizing Processors

프로세서의 가상화 방법은 LilyVM[6]와 같다. 여기서는 많은 비율로 가상 프로세서의 instruction들이 VMM의 간섭없이 실제 머신의 프로세서에 의해 실행된다. 오직 몇몇의 instruction들만 VMM이나 host OS의 상태에 따라서 실제 머신에 의해 직접적으로 실행되지 않고, VMM에 의해 간섭받게 된다.

VMM의 중재를 요구하는 instruction들은 sensitive instruction이라고 부른다. 어떻게 sensitive instruction들을 VMM이 trap과 emulate 설명하기 전에 sensitive instruction들을 2개의 카테고리로 분류해보면, `privileged instruction`과 `non-privileged instruction`으로 분류된다.  

- `privileged instruction`

  대부분 권한이 부여된 하드웨어 도메인과 관련된 몇몇의 sensitive instruction들은 실행하게 되면 일반적인 보호 예외가 발생하게 된다.

- `non-privileged instruction`

  `privileged instruction`처럼 예외를 발생시키지 않는 instruction을 `non-privileged instruction`이라고 한다.



예를들어 global descriptor table register(GDTR) 로 source의 값을 로드하는 연산인 `lgdt` instruction은 `privileged instruction`이다.

다른 한편으로 GDTR의 contents를 목적지에 저장하는 연산인 `sgdt` instruction은 `non-privileged instruction`이다.



VMM이 `sensitive instruction`을 캐치(traps)하는 방법은 instruction이 `privileged`냐 아니냐에 따라서 다른 방법이 사용된다. `privileged` instruction은 직접적으로 캐치(traps)한다. 가상머신은 사용자모드에서 실행되기 때문에 VMM은 실행으로 인해서 예외가 발생하는 부분만 캐치하면 된다. 다른 한편 `non-privileged` instruction을 캐치하는 것은 특별한 mechanism이 필요하다. 우리는 정적으로 `non-privileged` instruction의 실행이 원인이 되는 예외를 만들기 위해 guest OS kernel의 일부분을 재작성하였다. 조금 더 상세하게 이야기해서 각각의 kernel compile time에 `sensitiv` instruction 앞에 illegal instruction을 삽입한다. VMM은 `sensitive` instruction을 실행할 때 illegal instruction을 뒤따르는 `sensitive` instruction에 의해 원인이 되는 신호를 가로챈다.



이러한 가상 머신은 몇가지 장점과 단점이 존재한다. 장점은 작은 구현비용으로 OS를 구현할 수 있다는 점이고, 단점은 소스코드로부터 나온 시스템 레벨 바이너리를 실행할 수 없다는 것입니다

​    

### Virtualizing a Shared Memory

가상머신 내부에서 실행되는 guest OS는  실제 하드웨어로부터 제공되는 것과 같은 zero-based 물리 주소 공간을 갖기를 기대한다. 우리는 사용자 주소 공간에서 메모리의 가상화를 위해 4가지 기술을 발명했다.  3가지 기술은 paging mechanism이고, LiLyVM과 같은 기술이다. 4번째 기술은 가상 프로세서가 전역적으로 shared memory에 접근할 수 있게 허용하는 기술이다.



- VMM은 page를 가상머신의 page directory에 따라 물리적 메모리에 맵핑한다. 이러한 맵핑은 기본적인 page directory 물리 주소를 포함고있는 컨트롤 레지스터 3(page directory base register)가 변경될 때, 업데이트 된다.
- kernel 코드를 host OS와 geust OS의 kernel space의 중복을 피하기 위해서 정적으로 재작성했다.(kernel address space의 base address 부분)
- VMM은 memory에 없는 page를 찾을 때 실제 머신의 하드웨어에 의해 발생되는 예외를 가로채는 것을 통해서 page fault exception을 에뮬레이트 했다.
- VMM은 가상 머신 page 보호 mechanism을 이용한 공유 메모리 consistency protocol 을 구현했다. 자세하게 이야기해서 VMM은 shared page에 접근하기 위해서  `mprotect` 시스템 콜을 사용한다.  shared page에서 제한된 접근을 수행하는 어떤 시도들은 `SIGSEGV` signal을 발생시킨다. 이러한 signal을 캐치(trapping)했을 때, VMM은 원격 머신과 통신하는 것을 통해서 page의 content를 업데이트하고 page의 protection level을 업그레이드 한다.



​    

### Virtualizing I/O Devices

VMM은 I/O Device의 에뮬레이션을 위해서 하나의 중앙 서버를 준비한다.  서버는 지속적으로 모든 디바이스의 상태를 유지하고 geust OS에 의해 실행되는 I/O 작업에 대해서 원격 머신들과 통신한다. 예를 들어 geust OS가 `in` instruction과 함께 I/O port를 통해서 특정한 값을 읽으면 VMM은 다음과 같은 instruction을 에뮬레이트한다.

- VMM은 instruction의 실행을 가로챈 후, 요청을 중앙 서버로 보낸다. 중앙 서버에서 해당 요청을 받으면 서버는 가상 머신의 특별한 I/O포트로 부터 값을 읽어들인다. 그리고 난 후 해당 값을 VMM에게 보낸다. 마지막으로 VMM은 목적지에 instruction 연산을 통해 값을 복사한다. 



## Conclusion

우리는 본 논문에서 네트워크 상의 컴퓨터 위에서 가상화된 멀티 프로세서 머신인 VMM을 소개했다. VMM은 다양한 병렬/분산 컴퓨팅을 배포하는데 훌륭한 플랫폼을 제공한다.



## References

[[1] SETI@home](http://setiathome.ssl.berkeley.edu/)

[[2] IA-32 Intel Architecture Software Developer's Manual Volume 3: System Programming Guide](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.pdf)

[[3] Virtual Machines: Architectures, Implementations and Applications, chapter 0. An Overview of Virtual Machine Architectures](https://pdfs.semanticscholar.org/bb6c/4460b37a54530269dbdec19892c3e0836fc0.pdf)

[[4] The vMatrix: A Network of Virtual Machine Monitors for Dynamic Content Distribution](https://pdfs.semanticscholar.org/cc13/b6bdd5e3cdc74ad2ec55b6baa5ce320c98be.pdf)

[[5] Ananth I. Sundararaj and Peter A. Dinda. Towards Virtual Networks for Virtual Machine Grid Computing](https://www.researchgate.net/publication/2884273_Towards_Virtual_Networks_for_Virtual_Machine_Grid_Computing)

[[6] Running BSD kernels as user processes by partial emulation and rewriting of machine instructions](https://www.researchgate.net/publication/41035932_Running_BSD_kernels_as_user_processes_by_partial_emulation_and_rewriting_of_machine_instructions)

[[7] Implementing a User Mode Linux with Minimal Changes from Original Kernel](https://www.researchgate.net/publication/2554214_Implementing_a_User_Mode_Linux_with_Minimal_Changes_from_Original_Kernel)

[[8] Globus : A Metacomputing Infrastructure Toolkit](https://pdfs.semanticscholar.org/97d3/01f44b504e8c21c99658372343fba796c654.pdf)

[[9] VMPlants: Providing and Managing Virtual Machine Execution Environments for Grid Computing](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.603.2765&rep=rep1&type=pdf)

[[10] Virtualizing I/O Devices on VMware Workstation's Hosted Virtual Machine Monitor.](https://os.inf.tu-dresden.de/Studium/ReadingGroupArchive/slides/2007/20070404-doebel--vmware-io-virtualization.pdf)

[[11] Analysis of the Intel Pentium’s Ability to Support a Secure Virtual Machine Monitor](https://pdfs.semanticscholar.org/f6bf/459d260e89762ddff68f65e0c14e149bf4da.pdf)

[[12] A Case for Grid Computing on Virtual Machines](https://www.researchgate.net/publication/221459809_A_Case_for_Grid_Computing_on_Virtual_Machines)

[[13] A Survey of Virtual Machine System: Current Technology and Future Trends](https://www.researchgate.net/publication/232625032_A_Survey_of_Virtual_Machine_System_Current_Technology_and_Future_Trends)