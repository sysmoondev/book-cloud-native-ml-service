---
description: >-
  이 장에서는 Triton Inference Server에서 이미지 분류를 위한 DALI(Data Loading Library) Backend
  사용법에 대새 소개합니다. 샘플 예제에서는 Pytorch ResNet Image Classification 모델을 DALI Backend 와
  연동하여 Triton 앙상블 구조로 추론 하는 방법에 대해 소개합니다.
---

# DALI backend 예제 (/w Densenet)

## 사전 준비사항

DALI Backend `tritonserver 20.11`  릴리즈 버전을 out-of-box 형태로 공하고 있으며 [dali\_backend](https://github.com/sysmoon/dali\_backend.git) 오픈소스를 통해 아래 빌드 및 Triton Server 실행에 필요한 모든 도커 이미지와 클라이언트 라이브러를 제공하고 있습니다.

* ONNX 변환 and build TensorRT
  * `nvcr.io/nvidia/pytorch:21.07-py3`
* Triton Inference Server with DALI Backend
  * `nvcr.io/nvidia/tritonserver:21.07-py3`
  * `https://github.com/triton-inference-server/dali_backend`.
* Client
  * `nvcr.io/nvidia/tritonserver:21.07-py3-sdk`



## DALI Backend 빌드

DALI backend 를 테스트할 수 있는 desenet 분류 모델을 이용한 샘플 예제를 github 에서 clone 합니다.

```
git clone https://github.com/sysmoon/dali_backend.git
```

해당 예제는 densenet 모델을 이용하여 DALI backend 를 빌드하는데 필요한 모든 과정을 setup\_resnet50\_trt\_example.sh 스크립트 파일로 제공하고 있습니다.

```
cd dali_backend/docs/examples/resnet50_trt
sh setup_resnet50_trt_example.sh
```

스크립트 파일이 정상적으로 실행되면 densenet 모델을 ONNX 포멧으로 변환 후, trtexec 이용하여 TensorRT 빌드하고, 마지막으로 DALI 파이프라인으로 빌드하여 최종적으로 ./model\_repository/dali/1/model.dali 파일이 생성됩니다.

```
...
&&&& PASSED TensorRT.trtexec [TensorRT v8001] # trtexec --onnx=model.onnx --saveEngine=./model_repository/resnet50_trt/1/model.plan --explicitBatch --minShapes=input:1x3x224x224 --optShapes=input:1x3x224x224 --maxShapes=input:256x3x224x224 --fp16
[03/28/2022-01:56:23] [I] [TRT] [MemUsageChange] Init cuBLAS/cuBLASLt: CPU +0, GPU +0, now: CPU 1386, GPU 1826 (MiB)
+ python serialize_dali_pipeline.py --save ./model_repository/dali/1/model.dali
Saved ./model_repository/dali/1/model.dali
Resnet50 model ready.
```

### ONNX 이용한 TensorRT 변환

위 setup\_resnet50\_trt\_example.sh 스크립트 파일에서 실행되는 각 과정을 살펴봅니다.

#### 1. PyTorch 모델을 ONNX 모델로 변환

onnx\_exporter.py를 실행하여 ResNet50 PyTorch 모델을 ONNX 형식으로 변환합니다. 너비 및 높이는 224로 고정되지만 Dynamic Batch 처리에 대한 인수 값이 사용됩니다.

```
docker run -it --gpus=all -v $(pwd):/workspace nvcr.io/nvidia/pytorch:21.07-py3 bash
python onnx_exporter.py --save model.onnx
```

**2. Building ONNX-model to TensorRT engine**

fp16 정밀도를 위해 --fp16 인수를 활성화 합니다.  Dynamic Shape를 활성화 하기 위해 --explicitBatch와 함께 --minShapes, --optShapes 및 maxShapes를 사용할 수 있습니다.

```
trtexec --onnx=model.onnx --saveEngine=./model_repository/resnet50_trt/1/model.plan --explicitBatch --minShapes=input:1x3x224x224 --optShapes=input:1x3x224x224 --maxShapes=input:256x3x224x224 --fp16
```

#### 3. Serialize DALI 파이프라인

serialize\_dali\_pipeline.py를 실행하여 DALI 파이프라인을 생성합니다. 이 스크립트에서 DALI 파이프라인의 모양과 파이썬 수준에서 직렬화하는 방법에 대한 세부 정보를 찾을 수 있습니다.

```
python serialize_dali_pipeline.py --save ./model_repository/dali/1/model.dali
```

## Triton 서버에서 DALI Backend 실행

![](https://github.com/sysmoon/dali\_backend/raw/main/docs/examples/resnet50\_trt/images/ensemble.PNG)

### Triton 앙상블 구조의 모델 저장소

위 DALI Backend 빌드 과정을 수행하면 모델 저장소는 아래 폴더구조에 각 모델별 보전 폴에 mode.dali, model.plan 모델 파일이 생성됩니다.

```
model_repository
├── dali
│   ├── 1
│   │   └── model.dali
│   └── config.pbtxt
├── ensemble_dali_resnet50
│   ├── 1
│   └── config.pbtxt
└── resnet50_trt
    ├── 1
    │   └── model.plan
    ├── config.pbtxt
    └── labels.txt
```

### 모델 저장소 구성파일 (config.pbtxt)

* dali

```
name: "dali"                     # 모델이름
backend: "dali"                  # Triton Backend (dali)
max_batch_size: 256              # Dynamic Batch Max Size
input [
{
    name: "DALI_INPUT_0"        # 입력변수
    data_type: TYPE_UINT8       # 입력변수 타입
    dims: [ -1 ]                # 입력변수 차원
}
]
 
output [
{
    name: "DALI_OUTPUT_0"        # 출력변수 타입
    data_type: TYPE_FP32         # 출력변수 타입    
    dims: [ 3, 224, 224 ]        # 출력변수 차원
}
]
```

위 DALI 파이프라인 모델 구성은 입력/출력 포멧을 정의하고, 입력데이터에 대해 \[3, 224, 224] 차원으로 ReShape 하는 과정입니다.

* resenet50\_trt

```
name: "resnet50_trt"                # 모델이름
platform: "tensorrt_plan"           # Triton Backend (tensorrt_plan)
max_batch_size: 256                 # Dynamic Batch Max Size
input [
{
    name: "input"                    # 입력변수 이름    
    data_type: TYPE_FP32             # 입력변수 타입
    dims: [ 3, -1, -1 ]              # 입력변수 차원
    
}
]
output[
{
    name: "output"                  # 출력변수 이름
    data_type: TYPE_FP32            # 출력 타입   
    dims: [ 1000 ]                  # 출력 차원
    label_filename: "labels.txt"    # Image Classification 라벨링 매칭에 사용되는 파일            
}
]
```

* ensemble\__dali\__resnet50

```
name: "ensemble_dali_resnet50"
platform: "ensemble"
max_batch_size: 256
input [                      # 앙상블 모델의 입력값 정의 
  {
    name: "INPUT"
    data_type: TYPE_UINT8
    dims: [ -1 ]
  }
]
output [                    # 앙상블 모델의 출력값 정의
  {
    name: "OUTPUT"
    data_type: TYPE_FP32
    dims: [ 1000 ]
  }
]
ensemble_scheduling {        # 앙상블 모델 스케쥴링 정의
  step [
    {
      model_name: "dali"       # 첫번째 스탭 모델 설정 (dali)
      model_version: -1        # -1: 가장 마지막 모델 버전
      input_map {              # dali 모델 입력
        key: "DALI_INPUT_0"
        value: "INPUT"
      }
      output_map {             # dali 모델 출력
        key: "DALI_OUTPUT_0"
        value: "preprocessed_image"
      }
    },
    {
      model_name: "resnet50_trt"  # 두번째 스탭 모델 설정 (resnet50_trt)
      model_version: -1           # -1: 가장 마지막 모데 버전
      input_map {                 # resnet50_trt 모델 입력
        key: "input"
        value: "preprocessed_image"
      }
      output_map {                # resnet50_trt 모델 출력
        key: "output"
        value: "OUTPUT"
      }
    }
  ]
}
```

Triton 앙상블을 모델의 입/출력을 정의하고, 앙상블 안의 여러 개의 모델을 순서대로 스케쥴링 할 수 있습니다. 위 ensemble\__dali\__resnet50 모델의 config.pbtxt 설정은 dali, resnet50_trt 모델을 순서대로 스케쥴링하여  dali 에서 입력 데이터 값을 GPU 가속을 통해 데이터 로딩 및 전처리 하고, resnet50\_trt 모델_ 에서 image classification 하기 위한 설정입니다.

#### Triton Server 실행&#x20;

위 모델 저장소가 준비되면 Triton Server 를 이용하여 densenet 앙상블 모델에 대한 추론 서비스가 가능합니다. 현재 디렉토리 위치를 기준으로 컨테이너에서 필요한 /workspace, /model 에 대한 볼륨 마운트를 설정합니다.

* \-v$(pwd):/workspace
* \-v$(pwd)/model\_repository:/models

이후 Triton Server 의 Model Repository 위치를 --model-repository=/modes 로 설정하여 실행합니다.

```
docker run --gpus=all --rm -p8000:8000 -p8001:8001 -p8002:8002 -v$(pwd):/workspace/ -v/$(pwd)/model_repository:/models nvcr.io/nvidia/tritonserver:21.07-py3 tritonserver --model-repository=/models
```

### Triton Client

지금까지 Triton Server 환경에서 앙상블 모델(dali, densenet)을 구성하여 입력 데이터에 대한 Image Classification 추론 테스트가 가능한 환경을 구성했습니다. 이번 섹센에서는 Triton Client 환경에서 테스트 하기 위한 방법에 대해 소개합니다.

#### 1. 추론을 위한 샘플 이미지(mug.jpg) 다운로드

```
wget https://raw.githubusercontent.com/triton-inference-server/server/master/qa/images/mug.jpg -O "mug.jpg"
```

#### 2. Triton Client 컨테이너 실행

```
docker run --rm --net=host -v $(pwd):/workspace/ nvcr.io/nvidia/tritonserver:21.07-py3-sdk python client.py --image mug.jpg 
```

Triton Client 도커 이미 nvcr.io/nvidia/tritonserver:21.07-py3-sdk 를 이용하여 테스트 가능한 컨테이너를 실행합니다. 이때 DALI Backend 빌드 위치 (dali\_backend/docs/examples/resnet50\_trt)를 컨테이너 안에 /workspace 위치로 볼륨 마운트하고, client.py 실행시 mug.jpg 파일 경로명을 인자로 입력으로 하여 Triton Server 로 추론 요청합니다. 다음은 추론 결과로써 응답 속도(ms)와 분류 결과물 중에 확률 값이 가장 높은 클래스를 출력합니다.

```
0.02642226219177246ms class:COFFEE MUG
```

#### client.py 테스트 파일 구조 분석

위 과정에서는 도커 컨테이너 환경에서 [client.py](https://github.com/triton-inference-server/dali\_backend/blob/main/docs/examples/resnet50\_trt/client.py) 가 실행 가능한 환경이 모두 구성되어 있었기 때문에 간편하게 추론 테스트가 가능합니다. 하지만 상용 서비스 환경에서는 클라이언트 디바이스에 python DALI 라이브러리를 설치 후 직접 추론하기 위한 환경이 필요하므로 client.py 소스에 대한 이해가 필요합니다.

#### python 패키지 & 인자 설정

tritongrpcclient 파이썬 모듈을 이용하여 gRPC 프로토콜을 이용하여 Triton Server 에 추론 요청합니다. 그 밖에  필요한 기타 built-in 파이썬 패키지를 import 하고, 실행에 필요한 각 인자들을 정의합니다.

#### **Arguments**

* \--model_name: Triton 모델 이름 (기본 값으로는 현재 테스트 중인 ensemble\_dali\_resnet50 을 사용)_
* \--image: 추론에 사용할 이미지 경로명
* \--url: Triton Server 주소 (기본 서버주소: localhost, 포트: 8081(gRPC))
* \-v: verbose 디버깅 로그 분석 지원 (기본값:
* \--label\_file: Image Classificiation 을 위한 라벨링 리스트 파일 경로명

```
import os, sys
import numpy as np
import json
import tritongrpcclient
import argparse
import time

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--model_name",
                        type=str, required=False,
                        default="ensemble_dali_resnet50",
                        help="Model name")
    parser.add_argument("--image",
                        type=str,
                        required=True,
                        help="Path to the image")
    parser.add_argument("--url",
                        type=str,
                        required=False,
                        default="localhost:8001",
                        help="Inference server URL. Default is localhost:8001.")
    parser.add_argument('-v', "--verbose",
                        action="store_true",
                        required=False,
                        default=False,
                        help='Enable verbose output')
    parser.add_argument("--label_file",
                        type=str,
                        default="./model_repository/resnet50_trt/labels.txt",
                        help="Path to the file with text representation of available labels")
    args = parser.parse_args()
```

****

#### **입력 데이터 로딩**

load\_image 함수는 입력된 데이터를 uint8 numpy 포멧으로 로딩하는 함수 입니다. 앙상블 모델의 입/출력 값을 정의한 모델 설정(config.pbtxt) 파일에서 입력변수 타입으로 TYPE\_UINT8 정의했기 때문에  uint8 타입의 numpy 형식으로 데이터를 로딩합니다.

```
def load_image(img_path: str):
    """
    Loads an encoded image as an array of bytes.

    This is a typical approach you'd like to use in DALI backend.
    DALI performs image decoding, therefore this way the processing
    can be fully offloaded to the GPU.
    """
    return np.fromfile(img_path, dtype='uint8')

```

**gRPC tritonclient 생성**

tritongrpcclient 패키지를 이용하여 Triton gRPC Server 와 통신 가능한 클라이언트를 생성합니다. Triton Server 의 기본 주소는 localhost:8081 입니다.

```
triton_client = tritongrpcclient.InferenceServerClient(url=args.url, verbose=args.verbose)
```

#### 입/출력 구성

numpy에서 원시 이미지를 로드하고 모델 구성(config.pbtxt)에서 설정한 변수 이름, 모양(Shape), 및 데이터 유형으로 입력 및 출력을 구성합니다.

```
    inputs = []
    outputs = []
    input_name = "INPUT"
    output_name = "OUTPUT"
    image_data = load_image(args.image)
    image_data = np.expand_dims(image_data, axis=0)

    inputs.append(tritongrpcclient.InferInput(input_name, image_data.shape, "UINT8"))
    outputs.append(tritongrpcclient.InferRequestedOutput(output_name))

    inputs[0].set_data_from_numpy(image_data)

```

#### 추론 요청 & 결과값 확인

추론 요청 함수 infer() 에 추론에 사용할 모델 이름과 위에서 정의한 입/출력 구성된 내용을 인자로 입력하여 추론 결과값을 수신합니다.&#x20;

```
results = triton_client.infer(model_name=args.model_name,
                                    inputs=inputs,
                                    outputs=outputs)
output0_data = results.as_numpy(output_name)
```
