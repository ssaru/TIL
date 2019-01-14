# Internal MicroService API in Seldon-Core

사용자가 runtime prediction graph*(Tensorflow의 symbolic graph와 같은 graph를 이야기하는 듯 하다)*에 microservice 컴포넌트를 추가하려면 사용자는 Seldon-Core에서 사용하고있는 MicroSerivce에 적합한 API를 사용해서 Machine Learning 서비스를 만들어야한다.

Seldon-Core는 시스템 내부에서 다음과 같은 기본적인 서비스를 제공한다.

- Model
- Router
- Combiner
- Transformer
- Output Transformer



![example_runtime_model_graph](https://user-images.githubusercontent.com/13328380/51100848-44dc9080-181b-11e9-8c36-b2328fa4db57.png)

​    

## Model

예측값을 반환해주는 서비스. `RESTful` 방식과 `gRPC`방식을 지원한다.

​    

### REST API

| Endpoint | POST /prediction                                             |
| -------- | ------------------------------------------------------------ |
| Request  | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |
| Response | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |



request의 payload의 예시는 다음과 같다

```bash
{"data":{"names":["a","b"],"tensor":{"shape":[2,2],"values":[0,0,1,1]}}}
```

​    

### gRPC

```python
service Model {
  rpc Predict(SeldonMessage) returns (SeldonMessage) {};
}
```

[proto 정의](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition)를 참고

​    

## Router

request를 어떠한 children에서 routing해주고, childeren으로부터 feedback reward*(???)*를 받는 서비스.

​    

### REST API

| Endpoint | POST /route                                                  |
| -------- | ------------------------------------------------------------ |
| Request  | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |
| Response | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |



request의 payload 예시

```bash
{"data":{"names":["a","b"],"tensor":{"shape":[2,2],"values":[0,0,1,1]}}}
```

​    

### Send Feedback

| Endpoint | POST /send-feedback                                          |
| -------- | ------------------------------------------------------------ |
| Request  | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |
| Response | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |



request의 payload 예시

```bash
    "request": {
        "data": {
            "names": ["a", "b"],
            "tensor": {
                "shape": [1, 2],
                "values": [0, 1]
            }
        }
    },
    "response": {
        "data": {
            "names": ["a", "b"],
            "tensor": {
                "shape": [1, 1],
                "values": [0.9]
            }
        }
    },
    "reward": 1.0
}
```

​    

### gRPC

```python
service Router {
  rpc Route(SeldonMessage) returns (SeldonMessage) {};
  rpc SendFeedback(Feedback) returns (SeldonMessage) {};
 }
```

[proto 정의](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition)를 참고

​    

## Combiner

Children들의 응답을 하나의 응답으로 결합하는 서비스.

​    

### REST API

#### Combine

| Endpoint | POST /combine                                                |
| -------- | ------------------------------------------------------------ |
| Request  | JSON representation of [`SeldonMessageList`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |
| Response | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |

​    

### gRPC

```python
service Combiner {
  rpc Aggregate(SeldonMessageList) returns (SeldonMessage) {};
}
```

[proto 정의](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition)를 참고

​    

## Transformer

Input값을 변환해주는 서비스

​    

### REST API

#### Transform

| Endpoint | POST /transform-input                                        |
| -------- | ------------------------------------------------------------ |
| Request  | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |
| Response | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |



request의 payload 예시

```bash
{"data":{"names":["a","b"],"tensor":{"shape":[2,2],"values":[0,0,1,1]}}}
```

​    

### gRPC

```python
service Transformer {
  rpc TransformInput(SeldonMessage) returns (SeldonMessage) {};
}
```

[proto 정의](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition)를 참고

​    

## Output Transformer

child의 response를 변환하는 서비스 

​    

### REST API

| Endpoint | POST /transform-output                                       |
| -------- | ------------------------------------------------------------ |
| Request  | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |
| Response | JSON representation of [`SeldonMessage`](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition) |



request payload의 예시

```bash
{"data":{"names":["a","b"],"tensor":{"shape":[2,2],"values":[0,0,1,1]}}}
```

​    

### gRPC

```python
service OutputTransformer {
  rpc TransformOutput(SeldonMessage) returns (SeldonMessage) {};
}
```

[proto 정의](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/prediction.md/#proto-buffer-and-grpc-definition)를 참고

