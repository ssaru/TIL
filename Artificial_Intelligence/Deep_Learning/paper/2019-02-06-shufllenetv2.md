# ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design    

​    

# ShuffleNet V2: an Efficient Architecture

​    

## Review of ShuffleNet v1

- light-weight network의 주요한 도전과제는 제한된 $FLOPs$안에서 제한된 feature channel의 갯수만 사용할 수 있다는 것이다.
- $FLOPs$의 증가 없이 channel의 개수를 증가시키는 방법에는 2가지 테크닉이 있다.
  - *pointwise group convolution*
  - *bottleneck-like structure*
- *channel shuffle* operation은 다른 channel group간 정보 교환을 가능하게 만들어 accuracy를 향상시켰다.
- *pointwise group convolution*과 *bottleneck structure*는 앞서 이야기했듯이 $MAC$ 을 증가시킨다.(**G1** and **G2**). 이러한 비용은 light-weight model에서는 꽤 중요한 요소가 된다. 또한 많은 그룹을 사용하는 것은 **G3**를 위반한다. *shortcut connection*의 element-wise *Add* operation 또한 **G4**를 위반한다. 따라서 핵심 이슈는 **어떻게 많은 수, 동등한 넓이의 channel들을 dense convolution이나 많은 그룹없이 유지할 수 있냐**가 된다.

​    

## Channel Split and ShuffleNet V2

- 앞서 이야기한 목적에 맞게 우리는 *channel split*이라고 불리는 operation을 사용한다. 이는 Figure 3(c)와 같다.

![fig3](https://user-images.githubusercontent.com/13328380/52345797-6d922780-2a61-11e9-96a6-63d85e668b20.png)



- 각 unit이 시작할 때, input feature $c$의 channel을 2개의 브랜치($c$ 와 $c^{\prime}$)로 나눈다. 
- **G3**를 따라 하나의 브랜치는 identity로 남겨두고, 다른 브랜치는 3개의 convolution으로 구성되게 만들고, input channel과 output channel이 같게끔 만들어 **G1**을 만족시키도록 한다.
- 2개의 1x1 convolution은 group-convolution을 사용하지 않는다. 이는 split operation이 2개의 group을 만들기 때문에 부분적으로 **G2**를 만족한다.
- convolution을 통과한 다음에는 2개의 브랜치는 concatenation된다. 결국, channel은 같게 유지되며 **G1**을 만족한다. 
- *channel shuffle* operation은 shuffleNet V1과 같게, 2개의 브랜치간 정보를 교환한다.
- *channnel shuffle*후에 다음 unit이 시작된다. shuffleNet V1 처럼 *Add* operation은 존재하지 않는다. *ReLU* 그리고 *depthwise convolution*은 하나의 브랜치에만 존재한다. 
- 3가지 연속적인 *element-wise* operation인 *Concat*, *Channel Shuffle*, *Channel Split* operation은 single element-wise opeartion으로 합쳐진다.
- 이러한 변화는 **G4**를 만족하게되어 이득을 주게 된다.

- *spartial down sampling*의 경우 unit이 조금 변경되는데, 이는 Figure 3(d)에 나타나있다. *channel split operation*이 빠지기 때문에 output channel이 2배가 된다.
- 제안한 unit block (c), (d)이 *ShuffleNet V2*의 구조이다. 이를 기반으로 실험한 결과, 우리는 해당 네트워크 구조가 효율적인 구조라고 결론지었다. 네트워크의 구조는 Table 5에 요약되어 나타나있다. shuffleNet V1과의 한가지 차이점은 Global averaged pooling 앞에 1x1 convolution layer 있어 feature을 혼합한다.

![table5](https://user-images.githubusercontent.com/13328380/52347655-daa7bc00-2a65-11e9-81cf-adaf7419838d.png)

​    

## Analysis of Network Accuracy

- *ShuffleNet V2*는 효율적일뿐만 아니라 정확하다. 이유는 아래와 같이 2가지로 분석된다.
  - 각 unit block 높은 효율이 더 많은 feature channel과 큰 네트워크와 버금가는 용량을 갖게 만든다.
  - 각각의 unit block이 절반의feature channel($c^{\prime}$)을 가지고있는데, 이는 직접적으로 다음 block에 가서 합쳐진다. 이는 *feature reuse*로 간주할 수 있는데, *DenseNet*과 *CondenseNet*과 유사하다고 볼 수 있다.
  - *DenseNet*에서 feature reuse pattern을 분석하기 위해 layer간 weights의 *l1-norm*을 plotting한다. 그 결과는 Figure 4(a)와 같다.
  - 이는 명백하게 다른 layer와는 다르게 인접한 layer 사이에서는 강력하게 연결되어있음을 확인할 수 있다.
  - 모든 layer간 dense connection은 어떤 capacity의 낭비를 의미한다고 볼 수 있다. *CondenseNet*은 이러한 관점을 지지한다.

![fig4](https://user-images.githubusercontent.com/13328380/52347823-3d995300-2a66-11e9-81f6-e211cd09190c.png)

​    

- ShuffleNet V2에서는 $i$-th와 ($i+j$)-th building block 사이의 *directly-connected* channel의 수가 $r^{j}c$임을 쉽게 증명할 수 있다. 

  > $r = (1-c^{\prime})/c$

- 다른 말로, 2개의 block 사이의 거리에 따라 feature reuse의 양이 exponentially하게 decays된다. 즉, 멀리 떨어진 block사이에서는 feature reuse가 약해진다. Figure 4(b)는 $r = 0.5$로 설정한 결과이며, (a)와 유사한 결과를 보여준다.

- ShuffleNet V2의 구조는 이러한 feature reuse 패턴 기반의 설계 구조를 따른다. 이 구조에 대한 실험 결과는 Table 8에서 검증되었다.

![table8](https://user-images.githubusercontent.com/13328380/52348863-abdf1500-2a68-11e9-9b94-56a98574318d.png)

