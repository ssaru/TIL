# ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design    

​    

# Abstract

- 현재의 neural network architecture design에서 computation complexity를 측정할 때,  대부분 **FLOPs**와 같은 *indirect* metric을 사용한다.
- speed와 같은 *direct* metric은 memory access cost나 platform characterics과 같은 다른 요소들에 의존한다.
- 따라서 본 논문은 *direct* metric을 이용해 이를 평가하는 방법을 제안하고 새로운 네트워크 아키텍쳐인 ShuffleNet V2를 제안한다.

​    

# Introduction

- MobileNet v2는 NASNET-A보다 빠르지만, FLOPs는 비슷하다.

- 따라서 FLOPs는 computation complexity를 표현하는 metric으로 부족하다.

- 이러한 indirect metric과 direct metric의 부조화는 2가지 이유에서 나타난다.

  - 몇몇 중요한 factor들(memory access cost; MAC)이 speed에 영향을 끼친다.

  - *degree of parallelism*. 즉, 병렬화가 쉬운 디자인이냐 아니냐에 따라서 속도 차이가 난다.

  - 같은 FPLOs를 갖는 operation도 platform에 따라서 실행시간이 다르다.

    - matrix multiplication을 가속화하는 tensor decomposition같은 경우 GPU에서의 tensor decomposition가 FLOP를 75%가까히 감소시킴에도 불구하고, 속도가 줄어든다.

    - 이에 대해서 확인해보니, cuDNN 라이브러리가 3x3 conv연산에 특별하게 최적화 되어있다는 것을 확인했다.

      > 일반적으로 3x3 conv가 1x1 conv보다 9배 느리다고 생각하기 어렵다. 

- 논문에서는 effective network architecture design시 중요한 원칙 2가지를 제안한다.

  - direct metric을 사용해야한다.
  - direct metric은 target platform에서 평가되어야한다.

​        

![fig1](https://user-images.githubusercontent.com/13328380/52105710-14765e00-2633-11e9-8af9-7f858157b8a1.PNG)



​    

# Practical Cuidelines for Efficient Network Design

- 2개의 산업레벨의 hardware를 기반으로 연구를 진행했다.

  - GPU. A single NVIDIA GeForce GTX 1080 Ti, cuDNN 7.0

  - ARM. A Qualcomm Snapdragon 810



    ![table1](https://user-images.githubusercontent.com/13328380/52106111-d4b07600-2634-11e9-9172-59b0774a9772.PNG)



- ShuffleNet v1과 MobileNet v2의 네트워크 실행시간을 분석해보면 아래 그럼과 같다.

- Other는 data I/O, data shuffle, element-wise operation이다.

  ![fig2](https://user-images.githubusercontent.com/13328380/52106083-b2b6f380-2634-11e9-8e4a-b813aa40591a.PNG)