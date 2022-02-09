# Architecture

Triton Architecture 소개를 통해 각각의 주요기능들이 내부적으로 어떻게 동작하는지 자세히 소개합니다. 아래의 사진은 Triton Inference Server의 High-Level Architecture를 보여줍니다. 각 주요 컴포넌트를 살펴보면 다음과 같습니다.

Model Repository triton이 추론을 위한 모델들을 저장/관리하는 저장소입니다. 보통 쿠버네티스 클러스터 환경에서 데이터를 영구적으로 저장하기 위한 PV(Persistent Volume) 저장소를 이용하여 모델을 저장/관리 합니다. 사용자의 Inference requests 는 HTTP/REST, gRPC, C API 와 같은 다양한 형의 API를 통해 TritonServer에 요청하고, 이후 각 모델의 스케쥴러로 전송되어 Batch Job Queuing 처리하고, 최종적으로 모델에 맞는 백엔드로 요청이 전달되어 Inferencing 하고, 그 결과물을 리턴하여 사용자에게 응답 메시지로 전송합니다.

Triton은 전처리, 후처리 과정 심지어 새로운 딥러닝 프레임워크와 같이 새로운 기능을 추가 확장할 수 있는 backend C API를 지원합니다. 또한 Triton으로 서빙되는 모델은 모델 관리 API (HTTP/REST or gRPC or C API)를 통해 쿼리되고 제어될 수 있습니다.

특히 Triton 은 쿠버네티스 멀티 GPU 노드 환경에서 Deployment 프레임워크와 쉽게 통합되어 Readness, Liveness 기능을 통해 Inference Service 에 대한 Health 체크가 가능하고, 그 밖에 네트워크 성능을 위한 주요 지표 Throughput, Latency 관련 정보를 수집하여 모니터링 할 수 있습니다.

![](https://github.com/triton-inference-server/server/raw/main/docs/images/arch.jpg)

### **Concurrent Model Execution**

triton은 여러 모델 그리고 하나의 모델에 대한 여러 인스턴스를 동일한 시스템에서 병렬로 실행할 수 있습니다. 시스템은 GPU가 하나 혹은 여러 개일 수도 있고, 없을 수도 있습니다. 아래의 사진에서는 2개의 모델(model0, model1) 이 있고, 각 모델에 대해 총 2개의 요청이 들어온 모습입니다. 요청이 들어오면 triton은 즉시 Inferencing을 위한 GPU 자원을 예약하고, GPU 하드웨어 스케줄러는 병렬로 계산을 시작합니다.



![](https://github.com/triton-inference-server/server/raw/main/docs/images/multi\_model\_exec.png)

기본적으로 같은 모델에 대해 여러 요청이 동시에 들어오면, triton은 gpu에서 하나만 스케줄링하여 요청을 직렬로 처리합니다.

![](https://github.com/triton-inference-server/server/raw/main/docs/images/multi\_model\_serial\_exec.png)

triton은 instance-group이라는 [model configuration](https://benlee73.tistory.com/46) 을 통해, 각 모델이 병렬 실행될 수 있는 수를 지정해줄 수 있다. 기본적으로 각 모델은 하나의 인스턴스만 가지고, instance-group을 통해 인스턴스를 증가하면 아래의 그림과 같이 진행된다. 아래 그림에 경우, model1 에 대해서 3개의 인스턴스를 지정했고, 동시에 3개의 요청을 실행할 수 있다.

![](https://github.com/triton-inference-server/server/raw/main/docs/images/multi\_model\_parallel\_exec.png)

### **Models And Schedulers**

triton은 각 모델에 대해 독립적으로 multiple scheduling, bathcing algorithms 을 지원 합니다. stateless, stateful, ensemble model 의 타입에 따라 triton이 지원하는 스케줄러가 다릅니다.

**Stateless Models**

stateless 모델은 inference 요청들 사이의 state를 저장하지 않고, 각 inference 들은 서로 독립적이다. 예를 들어, CNN의 image classification, object detection이 그렇다. default scheduler, dynamic batcher 가 이 stateless model에 사용될 수 있다. 즉, CNN과 같은 모델은 이전 요청들과는 관계가 중요치 않기 때문에, 다른 모델 인스턴스로 inference 해도 상관 없다.

**Statueful Models**

반면, stateful 모델은 inference 요청들 사이의 관계를 유지한다. 각 모델은 여러개의 인스턴스 실행을 통해 동접 요에 대한 병렬처리를 실행 수 있는데 이 경우 각 Inference 요청에 대한 상태 관리가 필요한 경우 서로 다른 인스턴스로 전달되면 상태 관리가 어렵습니다. 따라서 연속되는 요청들 사이의 상태가 유지되어야 하는 경우 새로운 요청을 새로운 모델 인스턴스에 보내지 않고 동일한 모델로 보내서 inference를 해야합니다. 이를 위해 모델은 triton에게 요청 시퀀스의 시작과 끝을 알리는 control 신호를 요구 합니다.

stateful model은 sequence batcher를 사용한다. 이는 한 시퀀스의 모든 inference 요청들이 같은 모델 인스턴로 들어가게 하여, 모델이 올바르게 state를 유지하도록 한다. 그리고 batcher는 모델과 통신하여, 시퀀스가 언제 시작하고 끝나는지, 시퀀스의 correlation ID 등을 알려준다.

client가 stateful model로 inference 요청을 보낼 때, 같은 시퀀스 안의 요청들은 모두 같은 correlation ID를 가져야하고, 시퀀스의 시작과 끝을 표시해야한다.

**Ensemble Models**

앙상블 모델은 하나 또는 여러 모델을 pipeline 연결하고, 각 모델들 간의 입출력 Tensor의 연결을 나타낸다. 파이프라인은 "data preprocessing -> inference -> data postprocessing" 와 같이 여러 모델을 포함하는 절차를 구하기 위해 앙상블 모델을 사용합니다. 이를 통해, Tensor 전송의 오버헤드를 피할 수 있고, triton으로 보내는 요청의 개수도 줄일 수 있다.

앙상블에 속해 있는 각 모델에 스케줄러가 있음에도 불구하고, 앙상블 모델에는 앙상블 스케줄러가 사용됩니다. model configuration의 _ModelEnsembling::Step_ 에서 모델 사이의 dataflow를 지정할 수 있다. 스케줄러는 위에서 작성한 각 step의 output tensors를 모아서 지정된 step으로 전달한다. 앙상블 모델은 실제 모델이 아님에도 불구하고, 이러한 특성 때문에 밖에서는 하나의 모델처럼 보여진다.&#x20;

Image Classification 과 Segmenation 의 앙상블 모델은 아래와 같이 작성된다.

```
name: "ensemble_model"
platform: "ensemble"
max_batch_size: 1
input [
  {
    name: "IMAGE"
    data_type: TYPE_STRING
    dims: [ 1 ]
  }
]
output [
  {
    name: "CLASSIFICATION"
    data_type: TYPE_FP32
    dims: [ 1000 ]
  },
  {
    name: "SEGMENTATION"
    data_type: TYPE_FP32
    dims: [ 3, 224, 224 ]
  }
]
ensemble_scheduling {
  step [
    {
      model_name: "image_preprocess_model"
      model_version: -1
      input_map {
        key: "RAW_IMAGE"
        value: "IMAGE"
      }
      output_map {
        key: "PREPROCESSED_OUTPUT"
        value: "preprocessed_image"
      }
    },
    {
      model_name: "classification_model"
      model_version: -1
      input_map {
        key: "FORMATTED_IMAGE"
        value: "preprocessed_image"
      }
      output_map {
        key: "CLASSIFICATION_OUTPUT"
        value: "CLASSIFICATION"
      }
    },
    {
      model_name: "segmentation_model"
      model_version: -1
      input_map {
        key: "FORMATTED_IMAGE"
        value: "preprocessed_image"
      }
      output_map {
        key: "SEGMENTATION_OUTPUT"
        value: "SEGMENTATION"
      }
    }
  ]
}
```

스케줄러는 Ensemble 모델의 input(IMAGE), output(CLASSIFICATION, SEGMENTATION), 각 input\_map 과 output\_map의 모든 값을 인식한다. 그래서 앙상블 스케줄러가 보는 앙상블 모델은 아래의 그림과 같다.

![](https://github.com/triton-inference-server/server/raw/main/docs/images/ensemble\_example0.png)

앙상블 모델로 요청이 들어왔을 때, 앙상블 스케줄러의 동작은 아래와 같다.&#x20;

1. 요청의 "IMAGE" 텐서가 전처리 모델의 "RAW\_IMAGE" 로 매핑된 것을 인식한다.
2. 앙상블 내의 모델을 확인하고, 필요한 input tensor가 모두 준비되면 전처리 모델(image_preprocessor_model)로 요청을 보낸다.
3. 출력 텐서를 가져와서 "preprocessed\_image" 에 매핑한다.
4. 새로 수집된 텐서를 앙상블 내 모델의 input 으로 전송합니다. 그러면 두 모델 classficiation_model, segmentation\_model_ 은 준비상태가 됩니다.
5. 위의 tensor가 필요한 모델을 확인하고, input tensor가 모두 준비된 모델에 내부 요청을 보낸다. 응답은 모델 로드와 계산 시간에 따라 다르게 나타난다.
6. 3\~5번 step을 내부 요청이 전송될 때마다 반복하고, 응답을 앙상블 output 이름으로 매핑한다.

