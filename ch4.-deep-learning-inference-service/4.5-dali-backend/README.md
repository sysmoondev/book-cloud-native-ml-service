# DALI Backend

## NVIDIA DALI&#x20;

[NVIDIA DALI](https://github.com/NVIDIA/DALI)(데이터 로딩 라이브러리)는 딥 러닝 애플리케이션을 가속화하기 위한 데이터 로딩 및 전처리용 라이브러리입니다. 이미지, 비디오 및 오디오 데이터를 로드하고 처리하기 위해 고도로 최적화된 빌딩 블록 모음을 제공합니다. 널리 사용되는 딥 러닝 프레임워크에서 내장 데이터 로더 및 데이터 반복자를 위한 이식 가능한 drop-in 대체품으로 사용할 수 있습니다.

딥 러닝 애플리케이션에는 로드, 디코딩, 자르기, 크기 조정 및 기타 여러 기능 보강을 포함하는 복잡한 다단계 데이터 처리 파이프라인이 필요합니다. 현재 CPU에서 실행되는 이러한 데이터 처리 파이프라인은 병목 현상이 되어 학습 및 추론의 성능과 확장성을 제한합니다.

DALI는 데이터 전처리를 GPU로 오프로드하여 CPU 병목 현상을 해결합니다. 또한 DALI는 입력 파이프라인의 처리량을 최대화하도록 구축된 자체 실행 엔진에 의존합니다. 프리페치, 병렬 실행 및 일괄 처리와 같은 기능은 사용자를 위해 투명하게 처리됩니다.

또한 딥 러닝 프레임워크에는 여러 데이터 사전 처리 구현이 있으므로 교육 및 추론 워크플로의 이식성, 코드 유지 관리 가능성과 같은 문제가 발생합니다. DALI를 사용하여 구현된 데이터 처리 파이프라인은 TensorFlow, PyTorch, MXNet 및 PaddlePaddle로 쉽게 대상을 변경할 수 있으므로 이식성이 있습니다.

![DALI Backend Architecture](https://github.com/NVIDIA/DALI/raw/main/dali.png)

#### Easy-to-use functional style Python API

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



## DALI Backend

