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

### Shape Tensors

모양 텐서를 지원하는 모델의 경우 모양 텐서 역할을 하는 입력 및 출력에 대해 is\_shape\_tensor 속성을 적절하게 설정해야 합니다. 다음은 모양 텐서를 지정하는 구성과 예시입니다.

```
  name: "myshapetensormodel"
  platform: "tensorrt_plan"
  max_batch_size: 8
  input [
    {
      name: "input0"
      data_type: TYPE_FP32
      dims: [ -1 ]
    },
    {
      name: "input1"
      data_type: TYPE_INT32
      dims: [ 1 ]
      is_shape_tensor: true
    }
  ]
  output [
    {
      name: "output0"
      data_type: TYPE_FP32
      dims: [ -1 ]
    }
  ]
```

위에서 논의한 바와 같이 Triton은 입력 또는 출력 텐서 딤에 나열되지 않은 첫 번째 차원을 따라 일괄 처리가 발생한다고 가정합니다. 그러나 모양 텐서의 경우 첫 번째 모양 값에서 일괄 처리가 발생합니다. 위의 예에서 추론 요청은 다음과 같은 형태의 입력을 제공해야 합니다.

```
  "input0": [ x, -1]
  "input1": [ 1 ]
  "output0": [ x, -1]
```

여기서 x는 요청의 배치 크기입니다. Triton은 일괄 처리를 사용할 때 모델에서 모양 텐서를 모양 텐서로 표시해야 합니다. ~~~~ "input1"의 모양은 \[ 2 ]가 아니라 \[ 1 ]입니다. Triton은 모델에 요청을 보내기 전에 "input1"에 모양 값 x를 추가합니다.

### **Instance Groups**

Triton은 해당 모델에 대한 동시 추론 요청을 위해 모델의 여러 인스턴스를 제공할 수 있습니다. 모델 구성 항목 중에 ModelInstanceGroup 속성은 사용 가능한 실행 인스턴스의 수와 해당 인스턴스에 사용해야 하는 컴퓨팅 리소스 (CPU or GPU)를 지정하는 데 사용됩니다. 기본적으로 시스템에서 사용 가능한 각 GPU 마다 모델의 단일 실행 인스턴스가 생성됩니다. Instance Group을 통해 모든 GPU 혹은 특정 GPU 에 여러 실행 인스턴스를 띄울 수 있다. 아래 예제와 같이 각 GPU 에 2개의 인스턴스를 띄울 수 있다. GPU 뿐만 아니라 CPU도 지정도 가능합니다.

```
  instance_group [
    {
      count: 2
      kind: KIND_GPU
    }
  ]
```

그리고 다음 구성은 GPU 0에 하나의 실행 인스턴스를 배치하고 GPU 1과 2에 두 개의 실행 인스턴스를 배치합니다.

```
  instance_group [
    {
      count: 1
      kind: KIND_GPU
      gpus: [ 0 ]
    },
    {
      count: 2
      kind: KIND_GPU
      gpus: [ 1, 2 ]
    }
  ]
```

#### CPU Model Instance

인스턴스 그룹 설정은 CPU에서 모델을 실행하는 데에도 사용됩니다. 시스템에 사용 가능한 GPU가 있더라도 CPU에서 모델을 실행할 수 있습니다. 다음은 CPU에 두 개의 실행 인스턴스를 배치합니다.

```
  instance_group [
    {
      count: 2
      kind: KIND_CPU
    }
  ]
```

#### Host Policy

인스턴스 그룹 설정은 호스트 정책과 연결됩니다. 다음 구성은 인스턴스 그룹 설정에 의해 생성된 모든 인스턴스를 호스트 정책 "policy\_0"과 연결합니다. 기본적으로 호스트 정책은 인스턴스의 장치 종류에 따라 설정됩니다. 예를 들어 KIND\_CPU는 "cpu", KIND\_MODEL은 "모델", KIND\_GPU는 "gpu\_\<gpu\_id>"입니다.

```
  instance_group [
    {
      count: 2
      kind: KIND_CPU
      host_policy: "policy_0"
    }
  ]
```

### **Scheduling And Batching**

triton은 여러 inference requests 를 배치로 묶고 한번에 처리하여, inferencing throuphput을 크게 증가시킬 수 있습니다.

Triton은 개별 추론 요청이 입력 배치를 지정할 수 있도록 하여 배치 추론을 지원합니다. 입력 배치에 대한 추론은 동시에 실행되며, 추론 처리량을 크게 증가시킬 수 있으므로 GPU에 특히 중요합니다. 많은 사용 사례에서 여러 개별 추론 요청은 일괄 처리되지 않으므로 일괄 처리의 처리량 이점이 얻을 수 없기 때문에 Throughput 이득을 챙길 수 없습니다.

추론 서버에는 다양한 모델 유형 및 사용 사례를 지원하는 여러 일정 및 일괄 처리 알고리즘이 포함되어 있습니다. 모델 유형 및 스케줄러에 대한 자세한 내용은 모델 및 스케줄러에서 찾을 수 있습니다.

#### Default Scheduler

스케줄링\_선택 속성(_scheduling\_choice)_이 모델 구성에 지정되지 않은 경우 기본 스케줄러가 모델에 사용됩니다. 기본 스케줄러는 단순히 모델에 대해 구성된 모든 모델 인스턴스에 추론 요청을 분배 합니다.

#### Dynamic Scheduler

동적 일괄 처리는 서버에서 추론 요청을 결합하여 일괄 처리가 동적으로 생성되도록 하는 Triton의 기능입니다. 요청 일괄 처리를 생성하면 일반적으로 처리량이 증가합니다. 동적 배처는 상태 비저장 모델에 사용해야 합니다. 동적으로 생성된 배치는 모델에 대해 구성된 모든 모델 인스턴스에 배포됩니다.

ModelDynamicBatching 속성을 사용하여 각 모델에 대해 독립적으로 활성화 및 구성됩니다. 이러한 설정은 동적으로 생성된 배치의 기본 크기, 다른 요청이 동적 배치에 참여할 수 있도록 스케줄러에서 요청이 지연될 수 있는 최대 시간, 큐 크기와 같은 큐 속성, 우선순위, 타임아웃을 제어합니다.&#x20;

****

**Preferred Batch Sizes**

preferred\_batch\_size 속성은 동적 배처가 생성을 시도해야 하는 배치 크기를 나타냅니다. **** 대부분의 모델의 경우 권장 구성 프로세스에 설명된 대로 preferred\_batch\_size를 지정해서는 안 됩니다. 다른 배치 크기에 대해 여러 최적화 프로필을 지정하는 TensorRT 모델은 예외입니다. 이 경우 일부 최적화 프로필은 다른 프로필에 비해 상당한 성능 향상을 제공할 수 있으므로 이러한 고성능 최적화 프로필에서 지원하는 배치 크기에 대해 preferred\_batch\_size를 사용하는 것이 합리적일 수 있습니다.

다음 예는 기본 배치 크기가 4 및 8인 동적 배치를 활성화하는 구성을 보여줍니다.

```
  dynamic_batching {
    preferred_batch_size: [ 4, 8 ]
  }
```

모델 인스턴스를 추론에 사용할 수 있게 되면 동적 일괄 처리기는 스케줄러에서 사용 가능한 요청에서 일괄 처리를 생성하려고 시도합니다. 요청은 요청을 받은 순서대로 배치에 추가됩니다. 동적 배처가 원하는 크기의 배치를 구성할 수 있는 경우 가능한 가장 큰 기본 크기의 배치를 만들고 추론을 위해 보냅니다. 동적 배처가 원하는 크기의 배치를 구성할 수 없는 경우(또는 동적 배처가 선호하는 배치 크기로 구성되지 않은 경우), 모델에서 허용하는 최대 배치 크기보다 작은 가능한 가장 큰 크기의 배치를 보냅니다(그러나 이 동작을 변경하는 지연 옵션에 대해서는 다음 섹션 참조).

생성된 배치의 크기는 개수 메트릭을 사용하여 집계하여 검사할 수 있습니다.

#### Delayed Batching

동적 일괄 처리는 다른 요청이 동적 일괄 처리에 참여할 수 있도록 스케줄러에서 제한된 시간 동안 요청이 지연되도록 구성할 수 있습니다. 예를 들어 다음 구성은 요청에 대한 최대 지연 시간을 100마이크로초로 설정합니다.

```
  dynamic_batching {
    preferred_batch_size: [ 4, 8 ]
    max_queue_delay_microseconds: 100
  }
```

max\_queue\_delay\_microseconds 속성 설정은 최대 크기(또는 선호하는 크기) 일괄 처리를 생성할 수 없을 때 동적 일괄 처리 동작을 변경합니다. 사용 가능한 요청에서 최대 또는 기본 크기의 배치를 생성할 수 없는 경우 동적 배처는 구성된 max\_queue\_delay\_microseconds 값보다 오래 지연된 요청이 없는 한 배치 전송을 지연합니다. 이 지연 시간 동안 새 요청이 도착하고 동적 배처가 최대 또는 기본 배치 크기의 배치를 형성하도록 허용하면 해당 배치가 추론을 위해 즉시 전송됩니다. 지연이 만료되면 동적 일괄 처리는 최대 또는 기본 크기가 아니더라도 일괄 처리를 있는 그대로 보냅니다.

batcher 는 배치를 구성하는 요청 중 _max\_queue\_delay\_microseconds_ property 를 넘는 요청이 없다면, 계속 딜레이를 줄 수 있다. 만약 딜레이 중 _preferred\_batch\_size_를 만족하는 배치가 만들어지면, inference를 위해 즉시 전송된다. 최대 딜레이 시간을 채우면, 배치 사이즈가 만족스럽지 않더라도 보내게 된다.

```
  dynamic_batching {
    preferred_batch_size: [ 4, 8 ]
    max_queue_delay_microseconds: 100
  }
```

&#x20;

**Preserve Ordering**

_preserve\_ordering_ property는 모든 응답(responses)들이 요청(requests)와 같은 순서로 반환되도록 한다.



**Priority Levels**

기본 설정으로 dynamic batcher는 하나의 모델에 대해 요청을 쌓아두는 하나의 queue 를 가지고 있습니다. 그래서 요청은 순서대로 한번에 처리됩니다. priority\_levels 속성을 사용하여 동적 일괄 처리기 내에 여러 우선 순위 수준을 생성하여 우선 순위가 높은 요청이 우선 순위가 낮은 요청을 우회할 수 있습니다. 같은 수준의 우선순위끼리는 요청된 순서대로 처리합니다. 우선순위를 지정하지 않은 요청은 default\_priority\_level 로 스케쥴링 됩니다.



**Queue Policy**

동적 일괄 처리는 일괄 처리를 위해 요청이 대기하는 방식을 제어하는 ​​여러 설정을 제공합니다.

priority\_levels가 정의되지 않은 경우 default\_queue\_policy로 단일 큐에 대한 ModelQueuePolicy를 설정할 수 있습니다. __ priority\_levels가 정의되면 default\_queue\_policy 및 priority\_queue\_policy에 지정된 대로 각 우선순위 레벨은 서로 다른 ModelQueuePolicy를 가질 수 있습니다.

_priority\_levels_ 가 설정되었다면, default\_queue\_policy and priority\_queue\_policy 에 따라 ModelQueuePolicy가 결정된다. ModelQueuePolicy는 max\_queue\_size, timeout\_action, default\_timeout\_microseconds and allow\_timeout\_override 를 지정할 수 있습니다.

&#x20;

**Sequence Batcher**

동적 일괄 처리와 마찬가지로 시퀀스 일괄 처리는 일괄 처리가 동적으로 생성되도록 일괄 처리되지 않은 추론 요청을 결합합니다. 하지만 inference 요청들의 sequence가 같은 모델 인스턴스로 들어가야하는 stateful 모델을 사용한다는 점이 다릅니다. _ModelSequenceBatching_ property를 통해 sequence batching을 설정한다.이 설정은 sequence의 시작, 끝, 준비, 관계 ID를 정할 수 있다.



**Ensemble Scheduler**

앙상블 스케줄러는 앙상블 모델에 사용해야 하며 다른 유형의 모델에는 사용할 수 없습니다. _ModelEnsembleScheduling_ property를 통해 설정되고 모델간 tensor의 흐름을 지정할 수 있다. 자세한 내용으 [Ensemble Models](https://github.com/triton-inference-server/server/blob/main/docs/architecture.md#ensemble-models) 에서 확인할 수 있습니다.

### Optimization Policy

모델 구성 ModelOptimizationPolicy 속성은 모델에 대한 최적화 및 우선 순위 설정을 지정하는 데 사용됩니다. 이러한 설정은 백엔드에서 모델을 최적화하는지 여부와 Triton에서 예약 및 실행하는 방법을 제어합니다. 현재 사용 가능한 설정은 ModelConfig protobuf 및 최적화 문서를 참조하세요.\


### Model Warmup

모델이 Triton에 의해 로드되면 해당 백엔드가 해당 모델에 대해 초기화됩니다. 일부 백엔드의 경우 모델이 첫 번째 추론 요청(또는 처음 몇 개의 추론 요청)을 수신할 때까지 이 초기화의 일부 또는 전체가 지연됩니다. 결과적으로 지연된 초기화로 인해 첫 번째(몇몇) 추론 요청이 상당히 느려질 수 있습니다.

이러한 초기의 느린 추론 요청을 피하기 위해 Triton은 첫 번째 추론 요청이 수신되기 전에 완전히 초기화되도록 모델을 "워밍업"할 수 있는 구성 옵션을 제공합니다. ModelWarmup 속성이 모델 구성에 정의되면 Triton은 모델 워밍업이 완료될 때까지 모델을 추론할 준비 상태(READY)로 표시하지 않습니다.

모델 구성 ModelWarmup은 모델의 워밍업 설정을 지정하는 데 사용됩니다. 설정은 Triton이 각 모델 인스턴스를 워밍업하기 위해 생성할 일련의 추론 요청을 정의합니다. 모델 인스턴스는 요청을 성공적으로 완료한 경우에만 제공됩니다. 모델 워밍업의 효과는 프레임워크 백엔드에 따라 다르며 이로 인해 Triton이 모델 업데이트에 덜 반응하게 됩니다. 따라서 사용자는 자신의 필요에 맞는 구성을 실험하고 선택해야 합니다. 현재 사용 가능한 설정은 protobuf 설명서를 참조 가능합니다.

### Response Cache

모델 구성 [response\_cache](https://github.com/triton-inference-server/server/blob/main/docs/response\_cache.md) 섹션에는 이 모델에 대한 응답 캐시를 활성화하는 데 사용되는 enable Boolean 설정 값이 있습니다. 모델 구성에서 캐시를 활성화하는 것 외에도 서버를 시작할 때 0이 아닌 --response-cache-byte-size를 설정해야 합니다.

```
response_cache {
  enable: True
}
```
