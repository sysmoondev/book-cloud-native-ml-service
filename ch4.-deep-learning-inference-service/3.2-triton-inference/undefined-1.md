# 모델 구성

* [Model Configuration](https://benlee73.tistory.com/46?category=1030892)
* [Model Management](https://benlee73.tistory.com/58?category=1030892)
* [Model Repositor](https://benlee73.tistory.com/56?category=1030892)
* [Triton Backend](https://benlee73.tistory.com/51?category=1030892)

### **Model Configuration**

모델 리포지토리의 각 모델에는 모델에 대한 필수 및 선택적 정보를 제공하는 모델 구성이 포함되어야 합니다. 일반적으로 이 구성은 ModelConfig protobuf로 지정된 config.pbtxt 파일에 제공되며, 기본 형식은 다음과 같습니다.

```
  name: "Your Model Configuration Name'
  platform: "tensorrt_plan"
  max_batch_size: 8
  input [
    {
      name: "input0"
      data_type: TYPE_FP32
      dims: [ 16 ]
    },
    {
      name: "input1"
      data_type: TYPE_FP32
      dims: [ 16 ]
    }
  ]
  output [
    {
      name: "output0"
      data_type: TYPE_FP32
      dims: [ 16 ]
    }
  ]
```

위 설정은 float32 tensor 타입의 16 dimesion을 가지는 2개의 입력 값(input0, input1)과 1개의 출력값(output0)을 가지는  TensorRT 모델 입니다.

* name: 모델 구성 이름 (옵션)
* platform: TensorRT
* max\__batch\__size: 8
* input: 2개의 입력값 (input0, input1)
* output: 1개의 출력값 (output0)

&#x20;

**Name, Platform & Backned**

name property 가 지정되어 있지 않으면, 모델 repository 디렉토리와 같은 이름을 가집니다. name 항목이 지정되더라도 모델 repository 디렉토리와 반드시 동일한 이름을 가져야 합니다.

**Maximum Batch Size**

max\_batch\_size 속성은 Triton에서 활용할 수 있는 일괄 처리 유형에 대해 모델이 지원하는 최대 일괄 처리 크기를 나타냅니다. 모델의 일괄 처리 차원이 첫 번째 차원이고 모델에 대한 모든 입력 및 출력에 이 일괄 처리 차원이 있는 경우 Triton은 동적 일괄 처리 또는 시퀀스 일괄 처리를 사용하여 자동으로 모델에 일괄 처리를 사용할 수 있습니다.

이 경우 max\_batch\_size는 Triton이 모델과 함께 사용해야 하는 최대 배치 크기를 나타내는 1보다 크거나 같은 값으로 설정해야 합니다. 이 경우 max\_batch\_size는 Triton이 모델과 함께 사용해야 하는 최대 배치 크기를 나타내는 1보다 크거나 같은 값으로 설정해야 합니다.

**Inputs and Outputs**

각 모델의 inputs 및 outputs은  name, datatype 및 shape를 반드시 지정해야 합니다. input 또는 output tensor에 대해 지정된 이름은 모델에서 정의된 파라미터 이름과 일치해야 합니다. name은 보통 입력값의 경우  input0, input1 그리고 출력값의 경우 output0, output1 형식으로 지정합니다. datatype 과 shape 는 모델에 따라 달리 지정됩니다.

input/output shape는 dims property 와 max\_batch\_size 의 조합으로 결정됩니다. max\_batch\_size가 1 이상이라면 \[-1] + dims 가 되고, 0이라면 dims 이 됩니다. 따라서 위의 경우 max\_batch\_size 가 8 이므로, input output shape 는 모두 \[-1, 16] 가 됩니다.&#x20;

### **Auto-Generated Model Configuration**

특정 경우 --strict-model-config=false 옵션을 주고 triton을 시작시키면, triton이 자동으로 configuration file을 생성합니다. 생성되는 모델 구성 파일은 위의 필수적인 부분만 채워서 작성됩니다. 특히 TensorRT, TensorFlow saved-model, ONNX model 은 triton이 자동으로 세팅을 해주므로 configuration file이 필요하지 않습니다. 하지만 그 이외의 모델은 configuration file이 필수입니다.

### **Datatypes**

다음 표는 Triton에서 지원하는 텐서 데이터 유형을 보여줍니다. 첫 번째 열(Model Config)은 모델 구성 파일에 나타나는 데이터 유형의 이름을 보여줍니다. 다음 4개의 열(TensorRT, TensorFlow, ONNX Runtime, Pytorch)은 지원되는 모델 프레임워크에 대한 해당 데이터 유형을 보여줍니다. 모델 프레임워크에 주어진 데이터 유형에 대한 항목이 없으면 Triton은 해당 모델에 대해 해당 데이터 유형을 지원하지 않습니다.

TRITONSERVER C API, TRITONBACKEND C API, HTTP/REST 프로토콜 및 GRPC 프로토콜에 대한 해당 데이터 유형을 보여줍니다. 마지막 열은 Python numpy 라이브러리에 해당하는 데이터 유형을 보여줍니다.

| Model Config | TensorRT | TensorFlow | ONNX Runtime | PyTorch | API    | NumPy         |
| ------------ | -------- | ---------- | ------------ | ------- | ------ | ------------- |
| TYPE\_BOOL   | kBOOL    | DT\_BOOL   | BOOL         | kBool   | BOOL   | bool          |
| TYPE\_UINT8  |          | DT\_UINT8  | UINT8        | kByte   | UINT8  | uint8         |
| TYPE\_UINT16 |          | DT\_UINT16 | UINT16       |         | UINT16 | uint16        |
| TYPE\_UINT32 |          | DT\_UINT32 | UINT32       |         | UINT32 | uint32        |
| TYPE\_UINT64 |          | DT\_UINT64 | UINT64       |         | UINT64 | uint64        |
| TYPE\_INT8   | kINT8    | DT\_INT8   | INT8         | kChar   | INT8   | int8          |
| TYPE\_INT16  |          | DT\_INT16  | INT16        | kShort  | INT16  | int16         |
| TYPE\_INT32  | kINT32   | DT\_INT32  | INT32        | kInt    | INT32  | int32         |
| TYPE\_INT64  |          | DT\_INT64  | INT64        | kLong   | INT64  | int64         |
| TYPE\_FP16   | kHALF    | DT\_HALF   | FLOAT16      |         | FP16   | float16       |
| TYPE\_FP32   | kFLOAT   | DT\_FLOAT  | FLOAT        | kFloat  | FP32   | float32       |
| TYPE\_FP64   |          | DT\_DOUBLE | DOUBLE       | kDouble | FP64   | float64       |
| TYPE\_STRING |          | DT\_STRING | STRING       |         | BYTES  | dtype(object) |

&#x20;

### **Reshape**

Reshape는 Inference API 에서 받은 input output shape 가 예상한 입출력 모양과 다를 때 사용됩니다.

입력의 경우 reshape를 사용하여 입력 텐서를 프레임워크 또는 백엔드에서 예상하는 다른 모양으로 변경할 수 있습니다. 일반적인 사용 사례는 일괄 처리를 지원하는 모델이 일괄 처리된 입력의 모양이 \[ batch-size ]일 것으로 예상하는 경우입니다. 이는 일괄 처리 차원이 모양을 완전히 설명한다는 것을 의미합니다. 추론 API의 경우 각 입력이 비어 있지 않은 dims를 지정해야 하므로 동등한 모양 \[ batch-size, 1 ]을 지정해야 합니다. 이 경우 입력은 다음과 같이 지정해야 합니다.

```
  input [
    {
      name: "in"
      dims: [ 1 ]
      reshape: { shape: [ ] }
    }
```

출력의 경우 reshape를 사용하여 프레임워크 또는 백엔드에서 생성된 출력 텐서를 추론 API에서 반환되는 다른 모양으로 변형할 수 있습니다. 일반적인 사용 사례는 일괄 처리를 지원하는 모델이 일괄 처리된 출력이 \[ batch-size ] 모양을 가질 것으로 예상하는 경우입니다. 이는 일괄 처리 차원이 모양을 완전히 설명한다는 것을 의미합니다. 추론 API의 경우 각 출력이 비어 있지 않은 dims를 지정해야 하므로 동등한 모양 \[ batch-size, 1 ]을 지정해야 합니다. 이 경우 출력은 다음과 같이 지정해야 합니다.

```
  output [
    {
      name: "in"
      dims: [ 1 ]
      reshape: { shape: [ ] }
    }
```

~~~~

### Shape Tensors

blabla

&#x20;

### ~~Version Policy~~

&#x20;

### **Instance Groups**

triton은 한 모델에게 여러 instance를 제공하여, 여러 inference 요청을 동시에 처리할 수 있다.

_ModelInstanceGroup_ property로 실행 인스턴스 수와, 그 인스턴스를 띄울 자원을 지정할 수 있다.

기본적으로 시스템에서 사용 가능한 각 gpu 마다, 모델의 단일 실행 인스턴스가 생성된다.

instance group을 통해 모든 gpu 혹은 특정 gpu에 여러 실행 인스턴스를 띄울 수 있다.

코드는 아래와 같이 작성하여, 각 gpu에 2개의 인스턴스를 띄울 수 있다.

gpu 뿐만 아니라 cpu도 지정이 가능하다.

```
  instance_group [
    {
      count: 2
      kind: KIND_GPU
    }
  ]
```

&#x20;

### **Scheduling And Batching**

triton은 여러 inference requests 를 배치로 묶고 한번에 처리하여, inferencing throuphput을 크게 증가시킬 수 있다.

여러 경우 개별적인 inference requests 를 배치로 묶지 않아서, batching에 의한 throughput 이득을 챙기지 못한다.

&#x20;

**Default Scheduler (뿌리는 역할)**

configuration에서 _scheduling\_choice_ 가 지정되지 않을 때 사용된다.

모든 모델 인스턴스에 inference requests 를 분배한다.

**Dynamic Batcher (묶는 역할)**

dynamic batching은 requests 를 서버가 묶는 triton의 특징이다.

이는 throughput 향상을 가져오고, stateless 모델을 사용해야만 한다.

이렇게 만든 배치가 모든 모델 인스턴스에 분배된다.

&#x20;

dynamic batching은 model configuration에서 각 모델별로 독립적으로 설정이 가능하다.

동적으로 주로 생성할 배치 사이즈, 다른 요청과 결합되도록 스케줄러에서 기다릴 수 있는 최대 시간, 큐 properties 등을 설정할 수 있다.

&#x20;

**Preferred Batch Sizes**

dynamic batcher가 만들려고 시도할 배치 사이즈를 말한다.

아래 코드의 경우 4, 8로 설정하였다.

batcher는 inference할 수 있는 모델 인스턴스가 생성되면, scheduler에 있는 요청들로 배치를 묶는다.

요청이 온 순서대로 배치를 채우며, _preferred\_batch\_size_ 중 가능한 큰 배치로 묶는다.

만약 다 불가능하다면, 가장 큰 batch size보다는 작지만, 그래도 가장 큰 사이즈로 묶어서 보낸다.

&#x20;

&#x20;

**Delayed Batching**

dynamic batcher는 request를 배치로 묶기 위해, 바로 보내지 않고 조금 delay를 줄 수 있다.

아래 코드의 경우 100ms 시간을 지연시킬 수 있는 최대 값으로 지정하였다.

batcher 는 배치를 구성하는 요청 중 _max\_queue\_delay\_microseconds_ property 를 넘는 요청이 없다면, 계속 딜레이를 줄 수 있다.

만약 딜레이 중 _preferred\_batch\_size_를 만족하는 배치가 만들어지면, inference를 위해 즉시 전송된다.

최대 딜레이 시간을 채우면, 배치 사이즈가 만족스럽지 않더라도 보내게 된다.

```
  dynamic_batching {
    preferred_batch_size: [ 4, 8 ]
    max_queue_delay_microseconds: 100
  }
```

&#x20;

**Preserve Ordering**

_preserve\_ordering_ property는 모든 응답(responses)들이 요청(requests)와 같은 순서로 반환되도록 한다.

&#x20;

**Priority Levels**

기본적으로 dynamic batcher는 한 모델에 대해 요청을 쌓아두는 하나의 queue만 가진다.

그래서 요청은 순서대로, 한번에 처리된다.

_priority\_levels_ property는 우선순위를 부여하여, 높은 우선순위를 가진 요청이 먼저 배치에 쌓이도록 한다.

같은 수준의 우선순위끼리는 요청된 순서대로 처리한다.

우선순위를 지정하지 않은 요청은 default\_priority\_level 로 스케줄된다.

&#x20;

**Queue Policy**

_priority\_levels_ 이 설정되지 않았다면, default\_queue\_policy에 따라 하나의 queue만 사용된다.

_priority\_levels_ 가 설정되었다면, default\_queue\_policy and priority\_queue\_policy 에 따라 ModelQueuePolicy가 결정된다.

&#x20;

ModelQueuePolicy는 max\_queue\_size, timeout\_action, default\_timeout\_microseconds and allow\_timeout\_override 를 지정할 수 있도록 한다.

&#x20;

**Sequence Batcher**

요청을 배치로 묶는 다는 점이 dynamic batcher와 같다.

하지만 inference 요청들의 sequence가 같은 모델 인스턴스로 들어가야하는 stateful 모델을 사용한다는 점이 다르다.

_ModelSequenceBatching_ property를 통해 sequence batching을 설정한다.

이 설정은 sequence의 시작, 끝, 준비, 관계 ID를 정할 수 있다.

&#x20;

**Ensemble Scheduler**

ensemble model 과 사용되고, _ModelEnsembleScheduling_ property를 통해 설정된다.

모델간 tensor의 흐름을 지정할 수 있다.

