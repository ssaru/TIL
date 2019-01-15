# Seldon Deployment

SeldonDeployment는 kubernetes의 custom resource로 정의된다.

- [Design](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/seldon-deployment.md#design)
- [Proto Buffer Definiton](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/seldon-deployment.md#definition)
- [Examples](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/seldon-deployment.md#examples)

​        

## Design

![seldon-deployment-sketch](https://user-images.githubusercontent.com/13328380/51164311-e0cfd000-18df-11e9-8fa5-17332d0c6ac4.png)

​    

## Proto Buffer Definition

Seldon Deployment Custom Resource는 Proto Buffers에 의해서 정의된다.

```protobuf
syntax = "proto2";
package seldon.protos;

import "k8s.io/apimachinery/pkg/apis/meta/v1/generated.proto";
import "v1.proto";

option java_package = "io.seldon.protos";
option java_outer_classname = "DeploymentProtos";

message SeldonDeployment {
  required string apiVersion = 1;
  required string kind = 2;
  optional k8s.io.apimachinery.pkg.apis.meta.v1.ObjectMeta metadata = 3;
  required DeploymentSpec spec = 4;
  optional DeploymentStatus status = 5;
}

/**
 * Status for seldon deployment
 */
message DeploymentStatus {
  optional string state = 1; // A short status value for the deployment.
  optional string description = 2; // A longer description describing the current state.
  repeated PredictorStatus predictorStatus = 3; // A list of individual statuses for each running predictor.
}

message PredictorStatus {
  required string name = 1; // The name of the predictor.
  optional string status = 2;  // A short status value.
  optional string description = 3; // A longer description of the current status.
  optional int32 replicas = 4; // The number of replicas requested.
  optional int32 replicasAvailable = 5; // The number of replicas available.
}


message DeploymentSpec {
  optional string name = 1; // A unique name within the namespace.
  repeated PredictorSpec predictors = 2; // A list of 1 or more predictors describing runtime machine learning deployment graphs.
  optional string oauth_key = 3; // The oauth key for external users to use this deployment via an API.
  optional string oauth_secret = 4; // The oauth secret for external users to use this deployment via an API.
  map<string,string> annotations = 5; // Arbitrary annotations.
}

message PredictorSpec {
  required string name = 1; // A unique name not used by any other predictor in the deployment.
  required PredictiveUnit graph = 2; // A graph describing how the predictive units are connected together.
  repeated k8s.io.api.core.v1.PodTemplateSpec componentSpecs = 3; // A description of the set of containers used by the graph. One for each microservice defined in the graph. Can be split over 1 or more PodTemplateSpecs.
  optional int32 replicas = 4; // The number of replicas of the predictor to create.
  map<string,string> annotations = 5; // Arbitrary annotations.
  optional k8s.io.api.core.v1.ResourceRequirements engineResources = 6; // Optional set of resources for the Seldon engine which is added to each Predictor graph to manage the request/response flow
  map<string,string> labels = 7; // labels to be attached to entry deployment for this predictor
}


/**
 * Represents a unit in a runtime prediction graph that performs a piece of functionality within the prediction request/response calls.
 */
message PredictiveUnit {

  /**
   * The main type of the predictive unit. Routers decide where requests are sent, e.g. AB Tests and Multi-Armed Bandits. Combiners ensemble responses from their children. Models are leaf nodes in the predictive tree and provide request/response functionality encapsulating a machine learning model. Transformers alter the request features.
   */
  enum PredictiveUnitType {
    // Each one of these defines a default combination of Predictive Unit Methods
    UNKNOWN_TYPE = 0;
    ROUTER = 1; // Route + send feedback
    COMBINER = 2; // Aggregate
    MODEL = 3; // Transform input
    TRANSFORMER = 4; // Transform input (alias)
    OUTPUT_TRANSFORMER = 5; // Transform output
  }

  enum PredictiveUnitImplementation {
    // Each one of these are hard-coded in the engine, no microservice is used
    UNKNOWN_IMPLEMENTATION = 0; // No implementation (microservice used)
    SIMPLE_MODEL = 1; // An internal model stub for testing.
    SIMPLE_ROUTER = 2; // An internal router for testing.
    RANDOM_ABTEST = 3; // A A-B test that sends traffic 50% to each child randomly.
    AVERAGE_COMBINER = 4; // A default combiner that returns the average of its children responses.
  }

  enum PredictiveUnitMethod {
    TRANSFORM_INPUT = 0;
    TRANSFORM_OUTPUT = 1;
    ROUTE = 2;
    AGGREGATE = 3;
    SEND_FEEDBACK = 4;
  }

  required string name = 1; //must match container name of component if no implementation
  repeated PredictiveUnit children = 2; // The child predictive units.
  optional PredictiveUnitType type = 3;
  optional PredictiveUnitImplementation implementation = 4;
  repeated PredictiveUnitMethod methods = 5;
  optional Endpoint endpoint = 6; // The exposed endpoint for this unit.
  repeated Parameter parameters = 7; // Customer parameter to pass to the unit.
}

message Endpoint {

  enum EndpointType {
    REST = 0; // REST endpoints with JSON payloads
    GRPC = 1; // gRPC endpoints
  }

  optional string service_host = 1; // Hostname for endpoint.
  optional int32 service_port = 2; // The port to connect to the service.
  optional EndpointType type = 3; // The protocol handled by the endpoint.
}

message Parameter {

  enum ParameterType {
    INT = 0;
    FLOAT = 1;
    DOUBLE = 2;
    STRING = 3;
    BOOL = 4;
  }

  required string name = 1;
  required string value = 2;
  required ParameterType type = 3;

}
```

​    

## Single Model

- 해당 모델은 `seldonio/mock_classifier:1.0`에  포함되어있다.
- 모델은 1 MB 메모리를 필요로한다.
- 모델은 seldon-core에 구현된 API gateway를 사용할 oauth키와 암호를 정의한다.
- 모델은 `REST API`를 지원한다.

```json
{
    "apiVersion": "machinelearning.seldon.io/v1alpha2",
    "kind": "SeldonDeployment",
    "metadata": {
        "labels": {
            "app": "seldon"
        },
        "name": "seldon-deployment-example"
    },
    "spec": {
        "annotations": {
            "project_name": "FX Market Prediction",
            "deployment_version": "v1"
        },
        "name": "test-deployment",
        "oauth_key": "oauth-key",
        "oauth_secret": "oauth-secret",
        "predictors": [
            {
                "componentSpecs": [{
                    "spec": {
                        "containers": [
                            {
                                "image": "seldonio/mock_classifier:1.0",
                                "imagePullPolicy": "IfNotPresent",
                                "name": "classifier",
                                "resources": {
                                    "requests": {
                                        "memory": "1Mi"
                                    }
                                }
                            }
                        ],
                        "terminationGracePeriodSeconds": 20
                    }
                }],
                "graph": {
                    "children": [],
                    "name": "classifier",
                    "endpoint": {
			"type" : "REST"
		    },
                    "type": "MODEL"
                },
                "name": "fx-market-predictor",
                "replicas": 1,
		"annotations": {
		    "predictor_version" : "v1"
		}
            }
        ]
    }
}
```

​    

## Guide to constructing this custom resource service graph is provided

### Inference Graph

Seldon Core는 SeldonDeployment라는 custom resource를 이용해 kubernetes를 확장한다. SeldonDeployment에서는 Seldon이 관리할 다른 컴포넌트와 모델 구성요소로 이루어진runtime inference graph를 정의할 수 있다.



SeldonDeployment는 kubernetes의 PodTemplateSpce을 이용하여 실행시킬 컴포넌트 이미지와 서로 각자 다른 리소스의 이미지의 graph에 대한 정의파일이며, 파일포맷은 `JSON`이나 `YAML`로 구성되어있다.

아래는 SeldonDeployment의 일부 사진이다.

![inf-graph](https://user-images.githubusercontent.com/13328380/51164566-afa3cf80-18e0-11e9-97b9-71d7b17693d7.png)



해당 사진은 위에 있는 `YAML`파일과 같다.

```yaml
{
    "apiVersion": "machinelearning.seldon.io/v1alpha2",
    "kind": "SeldonDeployment",
    "metadata": {
        "labels": {
            "app": "seldon"
        },
        "name": "seldon-deployment-example"
    },
    "spec": {
        "annotations": {
            "project_name": "FX Market Prediction",
            "deployment_version": "v1"
        },
        "name": "test-deployment",
        "oauth_key": "oauth-key",
        "oauth_secret": "oauth-secret",
        "predictors": [
            {
                "componentSpecs": [{
                    "spec": {
                        "containers": [
                            {
                                "image": "seldonio/mock_classifier:1.0",
                                "imagePullPolicy": "IfNotPresent",
                                "name": "classifier",
                                "resources": {
                                    "requests": {
                                        "memory": "1Mi"
                                    }
                                }
                            }
                        ],
                        "terminationGracePeriodSeconds": 20
                    }
                }],
                "graph": {
                    "children": [],
                    "name": "classifier",
                    "endpoint": {
			"type" : "REST"
		    },
                    "type": "MODEL"
                },
                "name": "fx-market-predictor",
                "replicas": 1,
		"annotations": {
		    "predictor_version" : "v1"
		}
            }
        ]
    }
}
```



Single model에 대한 아주 간단한 예제는 다음과 같다.

```YAML
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: seldon-model
spec:
  name: test-deployment
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - image: seldonio/mock_classifier:1.0
    graph:
      children: []
      endpoint:
        type: REST
      name: classifier
      type: MODEL
    name: example
    replicas: 1
```



여기서 핵심 컴포넌트는 다음과 같다

- Predictors 리스트; replicas의 갯수를 지정한다.
  - 각각 graph를 정의하고 deployment를 결정한다. main graph와 canary 혹은 다른 제품의 rollout 시나리오사이에서 트래픽을 분기하고싶을 때, Multiple predictor들은 유용하다.
- 각각의 predictor는 componentSpec들의 리스트다. 각각의 componentSpec은 kubernetes PodTemplateSpec이며, Seldon은 이를 이용하여 kubernetes Deployment로 빌드한다. 여기에 graph와 그 graph의 요구사항들로 만들어진 image들이 위치하게 된다. e.g Volumes, ImagePullSecrets, Respirce Reqiest etc.
- Graph는 어떻게 component들이 함께 엮이는지 설명하는 명세이다.



inference graph정의에 대해서 자세히 확인하고싶다면 [here](https://github.com/SeldonIO/seldon-core/blob/master/docs/crd/readme.md)를 참조하자