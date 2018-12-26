# Virtualizing a Multiprocessor Machine on a Network of Computers

​    

## INTRODUCTION

Grid 컴퓨팅의 기본적인 목적은 여러 컴퓨터에 분산되어있는 이기종 자원을 완벽하게 다중화하는 것이다. 예를들어 Globus toolkit[8]은 cpu 아키텍쳐나 os시스템같이 다양한 하드웨어/소프트웨어 설정을 활용할 수 있는 middleware를 구축하는데 있다.  또 다른 예로는 사용자가 기여한 유후 cpu자원을 활용하여 외계지능을 찾아내는 SETI@home [1]이 될 수 있다.



Grid 컴퓨터환경(Computational Grid)을 개발하는 것의 주요 이슈는 사용자화(customizable)가 가능하고, 안전한 실행환경을 제공하는 것이다. 다양한 하드웨어/소프트웨어 설정은 특정한 os나 라이브러리에 의존성이 있는 인기있는 프로그램을 실행하기 어렵기 때문에 사용자화(customizable) 플랫폼은 꼭 필요하다. 또한 os에 보완적인 격리 및 보안 메커니즘을 제공하는 것은 사용자에 의해 실행되는 응용프로그램은 기본적으로 신뢰할 수 없고 시스템을 파괴할 수 있기 때문에 중요하다.



Grid 컴퓨터 환경(Computational Grid)을 개발하는 방법 중에 기존의 os컨셉을 따르며 자원 공유를 기본으로 하는 virtual machine monitor(VMM)[13]이 있다. VMM은 실제 컴퓨터에 있는 하드웨어 레이어(프로세스, 메모리, I/O device)를 가상화하고, 실제 단일의 컴퓨터처럼 보이게끔 만드는 virtual machine으로 추출한다. VMM은 Grid 컴퓨터환경 개발을 촉진하는 요소가 된다.