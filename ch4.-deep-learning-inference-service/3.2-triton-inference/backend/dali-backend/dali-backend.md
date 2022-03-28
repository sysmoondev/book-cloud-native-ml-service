# DALI Backend 소개



## NVIDIA DALI&#x20;

[NVIDIA DALI](https://github.com/NVIDIA/DALI)(데이터 로딩 라이브러리)는 딥 러닝 애플리케이션을 가속화하기 위한 데이터 로딩 및 전처리용 라이브러리입니다. 이미지, 비디오 및 오디오 데이터를 로드하고 처리하기 위해 고도로 최적화된 빌딩 블록 모음을 제공합니다. 널리 사용되는 딥 러닝 프레임워크에서 내장 데이터 로더 및 데이터 반복자를 위한 이식 가능한 drop-in 대체품으로 사용할 수 있습니다.

딥 러닝 애플리케이션에는 로드, 디코딩, 자르기, 크기 조정 및 기타 여러 기능 보강을 포함하는 복잡한 다단계 데이터 처리 파이프라인이 필요합니다. 현재 CPU에서 실행되는 이러한 데이터 처리 파이프라인은 병목 현상이 되어 학습 및 추론의 성능과 확장성을 제한합니다.

DALI는 데이터 전처리를 GPU로 오프로드하여 CPU 병목 현상을 해결합니다. 또한 DALI는 입력 파이프라인의 처리량을 최대화하도록 구축된 자체 실행 엔진에 의존합니다. 프리페치, 병렬 실행 및 일괄 처리와 같은 기능은 사용자를 위해 투명하게 처리됩니다.

또한 딥 러닝 프레임워크에는 여러 데이터 사전 처리 구현이 있으므로 교육 및 추론 워크플로의 이식성, 코드 유지 관리 가능성과 같은 문제가 발생합니다. DALI를 사용하여 구현된 데이터 처리 파이프라인은 TensorFlow, PyTorch, MXNet 및 PaddlePaddle로 쉽게 대상을 변경할 수 있으므로 이식성이 있습니다.

![DALI Backend Architecture](https://github.com/NVIDIA/DALI/raw/main/dali.png)

데이터 로딩 라이브러리인 NVIDIA DALI(R)는 딥 러닝 애플리케이션을 위한 입력 데이터의 사전 처리를 가속화하기 위해 고도로 최적화된 빌딩 블록과 실행 엔진의 모음입니다. DALI는 성능과 유연성을 모두 제공하여 서로 다른 데이터 파이프라인을 하나의 라이브러리로 가속화합니다. 이 라이브러리는 사용된 딥 러닝 프레임워크에 관계없이 다양한 딥 러닝 교육 및 추론 애플리케이션에 쉽게 통합될 수 있습니다.

### Features

* Easy-to-use functional style Python API
* Multiple data formats support - LMDB, RecordIO, TFRecord, COCO, JPEG, JPEG 2000, WAV, FLAC, OGG, H.264, VP9 and HEVC.
* Portable across popular deep learning frameworks: TensorFlow, PyTorch, MXNet, PaddlePaddle.
* Supports CPU and GPU execution.
* Scalable across multiple GPUs.
* Flexible graphs let developers create custom pipelines.
* Extensible for user-specific needs with custom operators.
* Accelerates image classification (ResNet-50), object detection (SSD) workloads as well as ASR models (Jasper, RNN-T).
* Allows direct data path between storage and GPU memory with [GPUDirect Storage](https://developer.nvidia.com/gpudirect-storage).
* Easy integration with [NVIDIA Triton Inference Server](https://developer.nvidia.com/nvidia-triton-inference-server) with [DALI TRITON Backend](https://github.com/triton-inference-server/dali\_backend).
* Open source.



### 사용법

Triton Inference Server 는 NVIDA DALI 를 Build-in backend 로 공식 지원하고 있습니다. 따라서 Pre-Trained 모델을 가지고 있다면, DALI 파이프라인을 구성하고, Triton 모델로 구성하여 GPU 가속화된 전처리 과정을 통해 빠른 추론 서비스가 가능합니다.

#### 1. DALI 파이프라인 생성

DALI 데이터 파이프라인은 Triton 내에서 모델로 표현됩니다. 이러한 모델을 생성하려면 Python에서 DALI 파이프라인을 구성하고 Pipeline.serialize 메서드를 호출하여 .dali 확장자로 생성되는 모델 파일(model.dali)을 생성해야 합니다.

```
 import nvidia.dali as dali

 @dali.pipeline_def(batch_size=256, num_threads=4, device_id=0)
 def pipe():
     images = dali.fn.external_source(device="cpu", name="DALI_INPUT_0")
     images = dali.fn.image_decoder(images, device="mixed")
     images = dali.fn.resize(images, resize_x=224, resize_y=224)
     return images

 pipe().serialize(filename="/my/model/repository/path/dali/1/model.dali")
```

#### 2. DALI 모델 저장소 구조

DALI 파이프라인을 구성하고 serialize 메서드를 통해 mode.dali 파일이 생성되면 아래와 같이 Triton 모델 저장소 구성과 동일한 방법으로 모델 버전별 관리가 가능합니다.

일반적으로 Triton 환경에서 DALI 모델 이름은 model.dali 이름으로 지정되어 사용되고, 모델 저장소 이름은 dali 로 사용합니다. 따라서 아래와 같이 Triton 모델 저장소에 dali 폴더를 생성하고, 모델 이름(dali) 하단에 버전별 모델을 저장고, 모델 구성파일(config.pbtxt) 을 이용하여 모델에 대한 입/출력 관계를 정의 합니다.

```
 model_repository
 └── dali
     ├── 1
     │   └── model.dali
     └── config.pbtxt
```

#### 3. Triton 모델 구성

모델 이름은 config.pbtxt 파일의 default\_model\_filename 항목에 사용자가 임의로 변경하여 적용 가능합니다. 다음은  config.pbtxt 구성은 ResizePipeline 에 대한 예제를 보여줍니다.

```
 name: "dali"
 backend: "dali"
 max_batch_size: 256
 input [
 {
     name: "DALI_INPUT_0"
     data_type: TYPE_UINT8
     dims: [ -1 ]
 }
 ]

 output [
 {
     name: "DALI_OUTPUT_0"
     data_type: TYPE_FP32
     dims: [ 224, 224, 3 ]
 }
 ]
```
