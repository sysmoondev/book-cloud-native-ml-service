# Quick Start

Triton 을 빠르게 테스트 하기 위해 소스 빌드 과정 없이 이미 제공하고 있는 Docker Image 이용하고, 샘플로 제공하는 딥러닝 모델을 이용해서 추론 테스트를 진행합니다.

일반적으로 추론을 하기 위해선 미리 학습된 모델이 필요합니다. Triton Inference Server 실행시, 미리 모델을 설정해줘서 관리 및 실행을 할 수 있습니다. Triton Inference Server 는 정해진 모델 폴더 양식이 따로 있습니다. 모델의 저장소는 아래와 같은 레이아웃 형태를 지켜야합니다. 후에, 로컬 서버에 모델과 레이아웃을 지켜준 폴더를 만들어주면, 도커로 run 실행시 docker의 -v 옵션을 통해서 컨테이너와 연결을 시켜 줍니다.&#x20;

### Triton 도커 이미지 설치

Triton Docker 이미지 테스트를 위해 사전에 다음과 사전 작업이 필요합니다.

* [Dokcer](https://docs.docker.com/desktop/)
* [Nvidia Container Toolkit](https://github.com/NVIDIA/nvidia-docker)

위 환경이 준비가 되면 Docker Hub를 통해 tritonserver 이미지를 검색하고, pull 하여 로컬 머신에 다운로드 합니다. 현재 시점 기준 마지막 20.12 버전을 활용하여 다음과 같이 이미지를 풀링합니다. 사이즈가 대략 12GB 정도로 크기 때문에 시간이 좀 걸립니다.

```
# docker search
docker search tritonserver | grep -i nvidia
shwsun/tritonserver                 nvcr.io/nvidia/tritonserver:20.12-py3          1  

# pull image
docker pull nvcr.io/nvidia/tritonserver:20.12-py3
```

Docker Pull 이 정상적으로 완료되면 로컬 환경에서 도커 이미지를 검색할 수 있습니다.

```
docker images | grep -i tritonserver
nvcr.io/nvidia/tritonserver 20.12-py3 73b851354265 14 months ago 12.4GB
```

### Triton 실행

Triton은 GPU 뿐만 아니라 CPU 환경에서도 Inferencing을 성능을 최적화하여 서비스를 제공합니다. 동일한 Triton 도커 이미지를 활용해서 테스트가 가능합니다.

#### Run on System with GPUs

Triton Docker 이미지를 이용하여 GPU 모드로 다음과 같이 실행 가능합니다. 사전 단계에서 Nvidia Container Toolkit 을 설치했기 때문에 Container 안에 있는 Application 에서 CUDA Driver 를 통해 Local Host 의 GPU 자원을 접근하여 사용 가능합니다.&#x20;

```
docker run --name --gpus=1 --rm -p8000:8000 -p8001:8001 -p8002:8002 -v/home/vpsdev/book/server/docs/examples/model_repository:/models nvcr.io/nvidia/tritonserver:21.02-py3 tritonserver --model-repository=/models
```

입력된 파라미터를 살펴보면 다음과 같습니다.

**GPU 옵션**&#x20;

\--gpus=1 파라미터를 이용하여 Trtton 에서 Inferencing 을 위해 필요한 GPU 자원의 수를 설정합니다. --gpus=all 이나, --gpus=2나 --gpus="device=0,1,2,3" 사용하여 추가 작업 없이 Multi GPU를 사용할 수 있습니다. 여기서는 GPU 1개만 사용하여 테스트를 진행합니다.

**컨테이너 실행 옵션**

\--rm 옵션을 이용하여 Docker Container 실행후 종료시 컨테이너 자동 삭제합니다.

**서비스 포트**

* \-p8000:8000: RestAPI 를 위한 HTTPService 포트 번호
* \-p8001:8001: gRPC 를 위한 GRPCInferenceService 포트 번호
* \-p8002:8002: Metric 수집을 위한 포트 번호

**볼륨 마운트**

\-v/home/vpsdev/book/server/docs/examples/model\_repository:/models

로컬 호스트에 있는 서빙할 모델의 위치(/home/vpsdev/book/server/docs/examples/model\_repository)와 컨테안에서 볼륨 마운트할 경로명(/models)을 맵핑합니다.

**도커 이미지 경로**

컨테이서 실행에 필요한 도커 이미지를 설정합니다. 위 docker pull nvcr.io/nvidia/tritonserver:21.02-py3 과정을 통해 이미 Triton 이미지를 pull 했기 때문에 로컬 호스트에 있는 도커 이미지를 사용합니다.

**컨테이너 애플리케이션**

컨테이너 안에서 실행할 애플리케이션(tritonserver) 지정

**모델 레파지토리**

Triton 에서 사용할 모델의 저장소 위치를 설정합니다. 위 볼륨 마운트 과정에서 서빙할 모델의 경로명을 컨테이너 안에서 /models 로 볼륨마운트 설정했기 때문에 --model-repository=/models 와 같이 설정합니다.



위 과정을 통해 tritonserver 가 정상적으로 실행되면 실행된 콘솔 마지막 부분에 다음과 같은 결과를 확인할 수 있습니다.

```

I0209 08:14:55.228701 1 server.cc:538] 
+----------------------+---------+--------+
| Model                | Version | Status |
+----------------------+---------+--------+
| densenet_onnx        | 1       | READY  |
| inception_graphdef   | 1       | READY  |
| simple               | 1       | READY  |
| simple_dyna_sequence | 1       | READY  |
| simple_identity      | 1       | READY  |
| simple_int8          | 1       | READY  |
| simple_sequence      | 1       | READY  |
| simple_string        | 1       | READY  |
+----------------------+---------+--------+
```

모델 레파지토리에 제공하는 모든 모델이 정상적으로 로딩되어  READY 상태가 되어야 합니다. 만약 특정 모델이 로딩 과정에서 실패하면 Fail 상태로 표시됩니다. 따라서 사용자가 개발한 커스텀 모델의 경우 Model Repository 에 저장하고 이후 Triton 실행시 정상적으로 로딩되어 READY 상태 여부를 체크가 필요합니다.

```
I0209 08:14:55.228893 1 tritonserver.cc:1642] 
+----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
| Option                           | Value                                                                                                                                              |
+----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
| server_id                        | triton                                                                                                                                             |
| server_version                   | 2.7.0                                                                                                                                              |
| server_extensions                | classification sequence model_repository schedule_policy model_configuration system_shared_memory cuda_shared_memory binary_tensor_data statistics |
| model_repository_path[0]         | /models                                                                                                                                            |
| model_control_mode               | MODE_NONE                                                                                                                                          |
| strict_model_config              | 1                                                                                                                                                  |
| pinned_memory_pool_byte_size     | 268435456                                                                                                                                          |
| cuda_memory_pool_byte_size{0}    | 67108864                                                                                                                                           |
| min_supported_compute_capability | 6.0                                                                                                                                                |
| strict_readiness                 | 1                                                                                                                                                  |
| exit_timeout                     | 30                                                                                                                                                 |
+----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+

I0209 08:14:55.231599 1 grpc_server.cc:3979] Started GRPCInferenceService at 0.0.0.0:8001
I0209 08:14:55.232107 1 http_server.cc:2717] Started HTTPService at 0.0.0.0:8000
I0209 08:14:55.275595 1 http_server.cc:2736] Started Metrics Service at 0.0.0.0:8002
```

tritonserver 서버 실행상태에 대한 여러 정보를 확인할 수 있고, Inference Test 가능한 프로토콜 별 endpoint 주소 정보와 모니터링을 위한 Metrics Server 정보 확인이 가능합니다.

nvidia-smi 를 통해 tritonserver 가 GPU 1개에 대해서 실행된 것을 확인할 수 있습니다.

```
nvidia-smi
Fri Feb 11 04:12:33 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.103.01   Driver Version: 470.103.01   CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:65:00.0 Off |                  N/A |
| 18%   40C    P8    20W / 250W |   1476MiB /  7982MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A     11890      C   tritonserver                     1473MiB |
+-----------------------------------------------------------------------------+
```

### Triton 실행 검증

Triton 서버가 정상 실행여부 검증을 위해 Metric Server endpoint 주소를 통해 Health 체크가 가능합니다. 이 주소를 이용하여 쿠버네티스 상용 서비스 환경에서 Liveness/Readness 용도로 활용합니다.

다음과 같이 curl 테스트를 통해 health check 를 실행하고, 200OK 응답을 정상적으로 수신하는지 확인합니다.

```
$ curl -v localhost:8000/v2/health/ready
...
< HTTP/1.1 200 OK
< Content-Length: 0
< Content-Type: text/plain
```

### Triton 테스트

Triton 은 클라이언트 사이드에서 다양한 테스트를 위한 도커 이미지를 이미 제공하고 있습니다. 이를 통해 위에서 실행한 tritonserver 에서 Image Classification 모델에 대해 Inference 테스트를 진행합니다. 먼저 21.02 버전과 동일한 nvcr.io/nvidia/tritonserver:20.12-py3-sdk 도커 이미지를 pull 합니다.

```
$ docker pull nvcr.io/nvidia/tritonserver:21.02-py3-sdk
```

이후, 도커 컨테이너를 실행합니다.

```
docker run -it --rm --net=host nvcr.io/nvidia/tritonserver:21.02-py3-sdk
```

컨테이너 실행시, --net=host 옵션을 사용하여 Docker 컨테이너 내부의 프로그램이 네트워크 관점에서 호스트 자체에서 실행중인 것처럼 보이게하는데 사용됩니다. 따라서 tritonserver:21.02-py3-sdk 컨테이너 안에서 위에서 이미 실행한 tritonserver 의 endpoint 주소 0.0.0.0:8000(or 8001) 접근하여 inference 테스트가 가능합니다.

densenet\_onnx 모델에 대해 /workspace/image/mug.jpg 샘플 이미지를 통해 Inference 테스트 결과를 확인할 수 있습니다.

```
$ /workspace/install/bin/image_client -m densenet_onnx -c 3 -s INCEPTION /workspace/images/mug.jpg
Request 0, batch size 1
Image '/workspace/images/mug.jpg':
    15.346230 (504) = COFFEE MUG
    13.224326 (968) = CUP
    10.422965 (505) = COFFEEPOT
```

![/workspace/images/mug.jpg](https://developer.nvidia.com/blog/wp-content/uploads/2018/09/mug-625x469.jpg)

Inference 에 사용된 사용자 이지미 mug.jpg 는 위 사진과 같으며 densenet\_onnx 모델에 의해 분류된 결과물을 확률값 기반으로 소팅된 형태로 확인할 수 있습니다.
