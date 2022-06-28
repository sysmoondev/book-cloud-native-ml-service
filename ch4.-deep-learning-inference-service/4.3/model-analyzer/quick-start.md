# Quick Start

아래 단계는 모델 분석기를 사용하여 간단한 PyTorch 모델인 add\_sub를 분석하는 방법을 안내합니다.

### Step1. Download the add\_sub model

****

**1. Create a new directory and enter it**

```
mkdir model_analyzer && cd model_analyzer
```

**2. Start a git repository**

```
git init && git remote add -f origin https://github.com/triton-inference-server/model_analyzer.git
```

3\. Git Sparse Checkout을 활성화하고 add\_sub 모델이 포함된 examples 디렉토리를 다운로드합니다.

```
git config core.sparseCheckout true && \
echo 'examples' >> .git/info/sparse-checkout && \
git pull origin main
```

### `Step 2:` Build and Run Model Analyzer Container

\
**1. Pull the SDK container:**

```
docker pull nvcr.io/nvidia/tritonserver:22.05-py3-sdk
```

**2. Run the SDK container**

model\_analyzer 결과물이 생성될 output 폴더를 생성하고, tritonserver:22.05-py3-sdk 컨테이너 실행시에 볼륨 마운트를 설정합니다.

```
docker run -it --gpus all \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v $(pwd)/examples/quick-start:$(pwd)/examples/quick-start \
      -v $(pwd)/output:$(pwd)/output \
      --net=host nvcr.io/nvidia/tritonserver:22.05-py3-sdk
```

\<path-to-output-model-repo>를 출력 모델 리포지토리가 위치할 디렉터리의 절대 경로로 바꿉니다. 이렇게 하면 Triton SDK 컨테이너가 Model Analyzer가 생성하는 모델 구성 변형에 액세스할 수 있습니다.

{% hint style="info" %}
중요: 마운트의 양쪽에서 절대 경로가 동일한지 확인해야 합니다.
{% endhint %}

이후 컨테이너 안에서 볼륨 마운트한 경로명으로 이동하면 model analyzer 테스트를 위한 quick-start 폴더명을 확인할 수 있습니다.

```
cd /home/vpsdev/model_analyzer
```

model\_analyzser 폴더 구조는 다음과 같습니다.

```
$ tree /home/vpsdev/model_analyzer
model_analyzer/
|-- examples
|   `-- quick-start
|       `-- add_sub
|           |-- 1
|           |   `-- model.pt
|           |-- config.pbtxt
|           `-- output0_labels.txt
```

**3. Add PDF support to the container**

Model Analyzer는 보고서 생성을 위해 pdfkit을 사용합니다. Triton SDK 컨테이너 안에 있으면 wkhtmltopdf를 설치해야 합니다.

```
apt-get update && apt-get install wkhtmltopdf
```

### `Step 3:` Profile the `add_sub` model

model\_analyzer 결과를 확인하기 위한 output 폴더를 생성합니다.

```
mkdir output
```

examples/quick-start 디렉토리에는 두 입력의 합과 차를 계산하는 간단한 libtorch 모델이 있습니다. 다음을 사용하여 컨테이너 내에서 Model Analyzer 프로필 하위 명령을 실행합니다.

```
model-analyzer profile \
    --model-repository examples/quick-start \
    --profile-models add_sub --triton-launch-mode=docker \
    --output-model-repository-path output/add_sub
```

{% hint style="info" %}
중요: \<output\_dir> 하위 디렉토리를 지정해야 합니다. --output-model-repository-path가 를 직접 가리키도록 할 수 없습니다.
{% endhint %}

{% hint style="info" %}
중요: 컨테이너에서 이전에 이미 실행했다면 --override-output-model-repository 옵션을 사용하여 이전 결과를 덮어쓸 수 있습니다.
{% endhint %}

{% hint style="info" %}
중요: model-analyzer profile 명령의 연속 실행 사이에 체크포인트 디렉토리를 제거해야 합니다.
{% endhint %}

컨테이너 안에서 위와 같이 model-analyzer를 실행하면, GPU 리소스를 이용하여 add\_sub 모델에 대해 내부적으로 Perf\_Analyazer 를 실행하기 위한 초기 작업을 수행합니다.

* Tesla T4 GPU 핸드러 초기화
* 체크포인트 디렉토리(/home/vpsdev/model\_analyzer/output/add\_sub) 파일 유무 체크
* GPU 서버 주요 메트릭 정보 프로파일링

```
[Model Analyzer] Initiliazing GPUDevice handles
[Model Analyzer] Using GPU 0 Tesla T4 with UUID GPU-d5fed071-b02f-1aea-30fd-c58a8dfa0efb
[Model Analyzer] Starting a Triton Server using docker
[Model Analyzer] No checkpoint file found, starting a fresh run.
[Model Analyzer] Profiling server only metrics...
```

이후 Perf\_Analyzer는 추론 서비스를 위한 최적의 배치 사이즈와, 인스턴스 수를 찾 Step by Step 으로 실험하여 Checkpoint  형태로 결과를 저장합니다. 아래와 같이 add\_sub\_config\_default, add\_sub\_config\_0 각 모드별 설정을 통해 Batch Size (수동 or 동적), Concurrency 값을 조절하여 T4 GPU 리소스 내에서 최적의 추론 서비스를 위한 Throughput / Latency 값을 찾아내고 이때 설정한 주요 파라미터 값을 결과 리포팅 합니다.

```
[Model Analyzer] Creating model config: add_sub_config_default
[Model Analyzer]
[Model Analyzer] Profiling add_sub_config_default: client batch size=1, concurrency=1
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_default: client batch size=1, concurrency=2
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_default: client batch size=1, concurrency=4
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_default: client batch size=1, concurrency=8
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_default: client batch size=1, concurrency=16
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_default: client batch size=1, concurrency=32
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] No longer increasing concurrency as throughput has plateaued
[Model Analyzer]
[Model Analyzer] Creating model config: add_sub_config_0
[Model Analyzer]   Enabling dynamic_batching
[Model Analyzer]   Setting instance_group to [{'count': 1, 'kind': 'KIND_GPU'}]
[Model Analyzer]   Setting max_batch_size to 1
[Model Analyzer]
[Model Analyzer] Profiling add_sub_config_0: client batch size=1, concurrency=1
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_0: client batch size=1, concurrency=2
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_0: client batch size=1, concurrency=4
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_0: client batch size=1, concurrency=8
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_0: client batch size=1, concurrency=16
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_0: client batch size=1, concurrency=32
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] No longer increasing concurrency as throughput has plateaued

...

[Model Analyzer] Creating model config: add_sub_config_37
[Model Analyzer]   Enabling dynamic_batching
[Model Analyzer]   Setting instance_group to [{'count': 5, 'kind': 'KIND_GPU'}]
[Model Analyzer]   Setting max_batch_size to 64
[Model Analyzer]
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=1
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=2
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=4
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=8
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=16
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=32
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=64
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=128
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=256
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=512
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] Profiling add_sub_config_37: client batch size=1, concurrency=1024
[Model Analyzer] Stopped Triton Server.
[Model Analyzer] No longer increasing max_batch_size because throughput has plateaued
[Model Analyzer] Saved checkpoint to /workspace/checkpoints/0.ckpt
[Model Analyzer] Profile complete. Profiled 39 configurations for models: ['add_sub']
[Model Analyzer]
[Model Analyzer] To analyze the profile results and find the best configurations, run `model-analyzer analyze --analysis-models add_sub`
```

이렇게 하면 add\_sub 모델의 제한된 구성 가능한 모델 매개변수에서 검색이 수행됩니다. 완료하는 데 최대 60분이 소요될 수 있습니다. 예를 들어 더 짧은 러닝(1\~2분)을 원하는 경우 아래 추가 옵션으로 러닝할 수 있습니다. 이러한 옵션은 최상의 구성을 찾기 위한 것이 아닙니다.

```
model-analyzer profile \
    --model-repository examples/quick-start \
    --profile-models add_sub --triton-launch-mode=docker \
    --output-model-repository-path output/add_sub \
    --run-config-search-max-concurrency 2 \
    --run-config-search-max-model-batch-size 2 \
    --run-config-search-max-instance-count 2
```

* \--run-config-search-max-concurrency: 구성 검색을 실행하는 최대 동시성 값을 초과하지 않도록 설정합니다.
* \--run-config-search-max-model-batch-size: 구성 검색을 실행하는 가장 높은 max\_batch\_size를 설정합니다.
* \--run-config-search-max-instance-count: 구성 검색을 실행하는 최대 인스턴스 수 값을 설정합니다.

이러한 옵션을 사용하면 모델 분석기는 5개의 구성(4개의 새로운 구성 및 수정되지 않은 기본 add\_sub 구성)을 테스트하고 각 구성에는 Perf Analyzer(동시성=1 및 동시성=2)에서 2개의 실험이 실행됩니다.

위 model-analyzer profile 명령을 실행하면 내부적으로 Perf Analyzer 가 실행되고, add\_sub 모델에 대한 성능 테스트를 위해 tritonserver (CONTAINER ID: 76972501493e) 가 실행된 것을 확인할 수 있습니다.

```
docker ps
CONTAINER ID   IMAGE                                       COMMAND                  CREATED          STATUS          PORTS                                                           NAMES
76972501493e   nvcr.io/nvidia/tritonserver:22.05-py3       "/opt/nvidia/nvidia_…"   1 second ago     Up 1 second     0.0.0.0:8000-8002->8000-8002/tcp, :::8000-8002->8000-8002/tcp   fervent_nobel
bd8fc4db4a25   nvcr.io/nvidia/tritonserver:22.05-py3-sdk   "/opt/nvidia/nvidia_…"   19 minutes ago   Up 19 minutes                                                                   sleepy_khorana
```

### `Step 4:` Generate Tables and Summary Reports

테이블 및 요약 보고서를 생성하려면 다음과 같이 분석 하위 명령을 사용합니다.

```
$ mkdir analysis_results
$ model-analyzer analyze --analysis-models add_sub -e analysis_results
```

model-analyzer profile의 결과물은 /home/vpsdev/model_analyzer/checkpoints/checkpoints 폴더 밑에 .ckpt 파일 형태로 생성되고, 이 파일을 이용하여 model-analyzer analyze가 요약 분석하여_ ./analysis\_results _디렉토리에 저장합니다._

```
Model Analyzer] Loaded checkpoint from file /home/vpsdev/model_analyzer/checkpoints/0.ckpt
[Model Analyzer] WARNING: No non-GPU metric corresponding to tag 'cpu_used_ram' found in the model's measurement. Possibly comparing measurements across devices.
[Model Analyzer] WARNING: No non-GPU metric corresponding to tag 'cpu_used_ram' found in the model's measurement. Possibly comparing measurements across devices.
[Model Analyzer] WARNING: No non-GPU metric corresponding to tag 'cpu_used_ram' found in the model's measurement. Possibly comparing measurements across devices.
[Model Analyzer] Exporting Summary Report to analysis_results/reports/summaries/add_sub/result_summary.pdf
[Model Analyzer] Exporting server only metrics to analysis_results/results/metrics-server-only.csv
[Model Analyzer] Exporting inference metrics to analysis_results/results/metrics-model-inference.csv
[Model Analyzer] Exporting GPU metrics to analysis_results/results/metrics-model-gpu.csv
```

디렉토리는 다음과 같이 구성되어야 합니다.

```
tree analysis_results/

analysis_results/
|-- plots
|   `-- simple
|       `-- add_sub
|           |-- gpu_mem_v_latency.png
|           `-- throughput_v_latency.png
|-- reports
|   `-- summaries
|       `-- add_sub
|           `-- result_summary.pdf
`-- results
    |-- metrics-model-gpu.csv
    |-- metrics-model-inference.csv
    `-- metrics-server-only.csv
```

### Summary & Report

model_analyzer 에 대한 요약 정보는 reports/summaries/add\_sub/result\_summary.pdf 파일을 열어서 확인할 수 있습니다._

주요 요약정보는 다음과 같이 확인 가능합니다.

* 5개의 구성으로 동시접속 (1 or 2) 각각에 대해 총 10번의 측정 수행
* 10번의 수행 중, add\_sub\_config\_1 구성이 가장 좋은 throughput (7122 infer/sec) 성능을 보여줌.
* add\__sub\_config\_1: GPU 당 1개의 인스턴스와 Max Batch Size=2 로 구성_

```
Model: add_sub
GPU(s): Tesla T4
Total Available GPU Memory: 15.7 GB
Constraint targets: None
In 350 measurements across 37 configurations, add_sub_config_21 provides the best throughput: 50831 infer/sec.
This is a 534% gain over the default configuration (8020 infer/sec), under the given constraints on GPU(s) Tesla T4. add_sub_config_21: 3/GPU model instances with a max batch size of 64 on platform pytorch_libtorch
Curves corresponding to the 3 best model configuration(s) out of a total of 37 are shown in the plots.
```

총 5개 중에서 최상위 3개에 해당하는 그래프를 그려보면 다음과 같습니다.

### Throughput vs. Latency

![](<../../../.gitbook/assets/image (2).png>)



### GPU Memory vs. Latency

![](<../../../.gitbook/assets/image (3).png>)
