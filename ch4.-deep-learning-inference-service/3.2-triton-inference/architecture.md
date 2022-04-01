# Architecture

Triton Architecture 소개를 통해 각각의 주요기능들이 내부적으로 어떻게 동작하는지 자세히 소개합니다. 아래 그림은 Triton Inference Server의 High-Level Architecture 에 대해 자세히 설명해 주고 있습니다. 각 주요 컴포넌트를 살펴보면 다음과 같습니다.

* Model Repository\
  triton이 추론을 위한 모델들을 저장/관리하는 저장소입니다. 보통 쿠버네티스 클러스터 환경에서 데이터를 영구적으로 저장하기 위한 PV(Persistent Volume) 저장소를 이용하여 모델을 저장/관리 합니다.
* API\
  사용자의 Inference requests 는 HTTP/REST, gRPC, C API 와 같은 다양한 형의 API를 통해 TritonServer에 요청합니다.
* Backend\
  이후 각 모델의 스케쥴러로 전송되어 Batch Job Queuing 처리하고, 최종적으로 모델에 맞는 백엔드로 요청이 전달되어 Inferencing 하고, 그 결과물을 리턴하여 사용자에게 응답 메시지로 전송합니다.

Triton은 전처리, 후처리 과정 뿐만 아니라 새로운 딥러닝 프레임워크에 대해 새로운 기능을 추가 확장할 수 있는다양한 [Backend](https://github.com/triton-inference-server/backend) 기능을 지원합니다. 또한 Triton으로 서빙되는 모델은 모델 관리 API (HTTP/REST or gRPC or C API)를 통해 쿼리하고 제어할 수 있습니다.

특히 Triton 은 쿠버네티스 멀티 GPU 노드 환경에서 Deployment 프레임워크와 쉽게 통합되어 Readness, Liveness 헬스 체크기능을 통해 Inference Service 에 대한 가용성을 제공합니다. 또한 Triton 이 기본적으로 제공하는 Metric 서버를 이용하여 Prometheus DB 에 수집/저장하고, Grafna 를 통해 Inferencing 서비스 주요 성능 지표 중 하나인 Throughput, Latency 관련 정보를 모니터링 할 수 있습니다.

![](https://github.com/triton-inference-server/server/raw/main/docs/images/arch.jpg)

### **Concurrent Model Execution**

triton은 여러 모델 그리고 하나의 모델에 대한 여러 인스턴스를 동일한 시스템에서 병렬로 실행할 수 있습니다. 시스템은 GPU가 하나 혹은 여러 개일 수도 있고, 없을 수도 있습니다. 아래의 이미지는 2개의 모델(model0, model1) 이 있고, 각 모델에 대해 총 2개의 요청이 들어온 모습입니다. 요청이 들어오면 triton은 즉시 Inferencing을 위한 GPU 자원을 예약하고, GPU 하드웨어 스케줄러는 병렬로 계산을 시작합니다.



![](https://github.com/triton-inference-server/server/raw/main/docs/images/multi\_model\_exec.png)

기본적으로 같은 모델에 대해 여러 요청이 동시에 들어오면, triton은 gpu에서 하나만 스케줄링하여 요청을 직렬로 처리합니다.

![](https://github.com/triton-inference-server/server/raw/main/docs/images/multi\_model\_serial\_exec.png)

triton은 instance-group이라는 [model configuration](https://benlee73.tistory.com/46) 을 통해, 각 모델이 병렬 실행될 수 있는 수를 지정해줄 수 있습니다. 기본적으로 각 모델은 하나의 인스턴스만 가지고, instance-group을 통해 인스턴스를 증가하면 아래의 그림과 같이 병렬 처리 됩니다. 아래 그림에 경우, model1 에 대해서 3개의 인스턴스를 지정했고, 동시에 3개의 요청을 실행할 수 있습니다.

![](https://github.com/triton-inference-server/server/raw/main/docs/images/multi\_model\_parallel\_exec.png)

### **Models & Schedulers**

triton은 각 모델에 대해 독립적으로 multiple scheduling, bathcing algorithms 을 지원 합니다. stateless, stateful, ensemble model 의 타입에 따라 triton이 지원하는 스케줄러가 다릅니다.

**Stateless Models**

stateless 모델은 inference 요청들 사이의 state를 저장하지 않고, 각 inference 들은 서로 독립적입니다. 예를 들어, CNN 계열의 Image Classification, Object Detection 모델이 이러한 예 입니다. default scheduler, dynamic batcher 가 Stateless Model 에 적용하여 사용할 수 있습니다, 즉, CNN과 같은 모델은 이전 요청들과의 관계가 중요하지 않기 때문에 다른 모델 인스턴스로 inference 해도 문제가 없습니다.

**Statueful Models**

반면, stateful 모델은 inference 요청들 사이의 관계를 유지합니. 각 모델은 여러개의 인스턴스 실행을 통해 동접 요청에 대한 병렬처리를 할 수 있는데 이 경우 각 Inference 요청에 대한 상태 관리가 필요한 경우 서로 다른 인스턴스로 전달되면 상태 관리가 어렵습니다. 따라서 연속되는 요청들 사이의 상태가 유지되어야 하는 경우 새로운 요청을 새로운 모델 인스턴스에 보내지 않고 동일한 모델 인스턴스로 전송하여 inference를 해야합니다. 이를 위해 모델은 triton에게 요청 시퀀스의 시작과 끝을 알리는 control 신호를 요구합니다.

stateful model은 sequence batcher를 사용합니다. 이는 한 시퀀스의 모든 inference 요청들이 같은 모델 인스턴로 들어가게 하고, 모델이 올바르게 state를 유지하도록 합니다. 그리고 batcher는 모델과 통신하여 시퀀스가 언제 시작하고 끝나는지 시퀀스의 correlation ID 정보를 알려줍니다.

client가 stateful model로 inference 요청을 보낼 때, 같은 시퀀스 안의 요청들은 모두 같은 correlation ID를 가져야하며, 시퀀스의 시작과 끝을 표시해야합니다.

**Ensemble Models**

앙상블 모델은 하나 또는 여러 개의 모델을 pipeline 으로 연결하고, 각 모델들 간의 입/출력 Tensor의 연결을 구성합니다. 파이프라인은 "data preprocessing -> inference -> data postprocessing" 와 같이 여러 모델을 포함하는 전/후처리 과정을 구성하기 위해 앙상블 모델을 사용합니다. 이를 통해, Tensor 전송의 오버헤드를 피할 수 있고, triton으로 보내는 요청의 개수도 줄일 수 있습니다.

Image Classification & Segmenation 을 위한 앙상블 모델 작성 방법은 다음과 같습니다.&#x20;

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

스케줄러는 Ensemble 모델의 input(IMAGE), output(CLASSIFICATION, SEGMENTATION) 그리고 각 input\_map 과 output\_map의 모든 값을 인식합니다. 아래 이미지는 앙상블 스케쥴러가 앙상블 모델을 처리하는 과정을 보여주고 있습니다.

![](https://github.com/triton-inference-server/server/raw/main/docs/images/ensemble\_example0.png)

앙상블 모델로 요청이 들어올 때, 앙상블 스케줄러는 다음 순서대로 동작 합니다.

1. 요청의 "IMAGE" 텐서가 전처리 모델의 "RAW\_IMAGE" 로 매핑된 것을 인식합니다.
2. 앙상블 내의 모델을 확인하고, 필요한 input tensor가 모두 준비되면 전처리 모델(image\__preprocessor\__model)로 요청을 전송합니다.
3. 출력 텐서를 가져온 후,  "preprocessed\_image" 에 매핑합니다.
4. 새로 수집된 텐서를 앙상블 내 모델의 input 으로 전송합니다. 그러면 앙상블 내 2개의 모델 classficiation\__model, segmentation\_model_ 은 준비상태가 됩니다.
5. 위의 tensor가 필요한 모델을 확인하고, input tensor가 모두 준비된 모델에 내부 요청을 전송합니다.\
   응답 값은 모델 로드와 계산 시간에 따라 다르게 나타날 수 있습니다.
6. 3\~5번 step을 내부 요청이 전송될 때마다 반복하고, 응답을 앙상블 output 이름 CLASSIFICATION\__OUTPUT, SEGMENTAION\_OUTPUT_ 으로 매핑합니다.

