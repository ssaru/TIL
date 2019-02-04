# ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design    

​    

### G1) Equal channel width minimizes memory access cost (MAC)

- 현대의 네트워크는 *depthwise separable convolution*을 사용하는데, *depthwise separable convolution*의 computation complexity의 대부분은 pointwise convolution이 차지하고 있다.

- 본 논문에서는 1x1 convolution에 대해서 연구했다.

- 1x1 convolution의 $FLOPs$와 $MAC$은 아래와 같다.

  $FLOPs = B = hwc_{1}c_{2}$

  $MAC = hw(c_{1}+c_{2}) + c_{1}c_{2}$

  $c_{1} : the\ number\ of\  input\ channels$

  $c_{2} : the\ number\ of\  output\ channels$



  > NOTE
  >
  > the two terms correspond to the memory access for input/output feature maps and kernel weights, respectively

- 위 결과는 아래와 같은 식으로 전개될 수 있다.

  $MAC \geq 2\sqrt{hwB} + \frac{B}{hw}$

- $MAC$은 $FLOPs$에 의해 lower bound를 갖는다.

- 하지만, 많은 device의 cache는 충분히 큰 용량을 갖지 않으므로, 현대의 컴퓨테이션 라이브러리는 최대한의 cache mechanism을 이용하기 위해서 복잡한 blocking 전략을 적용한다. 

- 따라서 $MAC$의 실질적인 계산은 위의 계산식을 따르지만 완벽하게 반영하지는 않을 것이다.

- Table 1은 고정된 $FLOPs$에서 다양한 $c_{1} : c_{2}$ 비율에 따라서 running speed 결과를 나타낸다.

- Table 1의 결론은 **$c_{1} : c_{2}$ 의 비율이 $1 : 1$일 때, MAC이 최소가 되며, 속도가 제일 빨라진다** 

![table1](https://user-images.githubusercontent.com/13328380/52106111-d4b07600-2634-11e9-9172-59b0774a9772.PNG)

​     

### G2) Excessive group convolution increases MAC

- *Group convolution* 또한 현대 네트워크에서 핵심이 된다.

- *Group convolution*은 dense convolution에서 모든 channel들을 sparse하게 변경하는 것을 통해서 computational complexity( $FLOPs$ )를 줄인다.

- 이는 주어진 $FLOPs$ 안에서 더 많은 channels들을 사용하므로 network capacity가 좋아지나, 더 많은 $MAC$ 을 요구한다.

- 이를 **G1**과 이의 *Eq. 1*을 사용하면 아래와 같이 표현될 수 있다.

  $MAC = hw(c_{1} + c_{2}) + \frac{c_{1}c_{2}}{g} = hwc_{1} + \frac{Bg}{c_{1}} + \frac{B}{hw}$

  $g : the\ number\ of\ groups$

  $FLOPs = B : hwc_{1}c_{2} / g $

- Table 2는 *Group convolution*에 대한 결과를 보여준다. 이는 large group number는 running speed를 크게 감소시킨다는 결과를 보여준다.

- 따라서 *group number*는 target platform과 task에 따라서 신중하게 결정되어야한다



![table_2](https://user-images.githubusercontent.com/13328380/52193600-5271bd00-2893-11e9-9255-b677a0235866.png)

​    

### G3) Network fragmentation reduces degree of parallelism

- GoogLeNet 시리즈나, autoML로 생성된 모델들은 *multi-path*구조를 가지고있는 경우가 많다.
- 다수의 작은 operator들(*fragmented operators*라고 불리는 operator)은 작은 수에 큰 operator들 대신에 사용된다.
- *fragmented operator*들을 사용하는 것을 통해서 accuracy측면에서 이득을 볼 수 있지만, parallel computing power측면에 적합하지 않기때문에 속도가 감소된다.
- 아래 Table 3이 그 결과를 보여준다. 단, ARM에서 속도의 감소량은 GPU 기반의 환경에서보다는 상대적으로 덜 감소했다.

![table3](https://user-images.githubusercontent.com/13328380/52193772-1be87200-2894-11e9-8d78-0eae3fc73245.png)

​    

### G4) Element-wise operations are non-negligible

- Figure 2에서 보여지는 것과 같이, element-wise operation이 processing time의 많은 영역을 차지하는 것을 확인할 수 있다.
- element-wise operation은 낮은 $FLOPs$를 갖지만, 상대적으로 큰 $MAC$을 갖는다. 
- 이번 파트에서는 *depthwise convolution*의 element-wise operator의 $MAC/FLOPs$의 비율을 집중적으로 살펴본다.
- Table 4는 그 결과를 보여준다. ReLU와 shortcut가 제거된 이후, GPU / ARM 모두에서 속도가 20% 증가된 것을 확인할 수 있다.



![fig2](https://user-images.githubusercontent.com/13328380/52106083-b2b6f380-2634-11e9-8e4a-b813aa40591a.PNG)



![table4](https://user-images.githubusercontent.com/13328380/52193906-c791c200-2894-11e9-8e3c-0728a5e83207.png)

​    

### Conclusion and Discussions

- 실험적인 연구 후에, 우리는 효율적인 네트워크 구조를 만들기 위해서는 아래와 같은 내용을 고려해야한다는 결론을 얻었다.
  - *balanced convolution*(channel width가 같은)을 사용해야한다.
  - *group convolution*의 비용을 고려해야한다.
  - *degree of fragmentation*를 줄여야한다.
  - element-wise operation들을 줄여야한다.