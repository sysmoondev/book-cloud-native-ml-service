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
# create output dir
mkdir output_add_sub

# run docker
docker run -it --gpus all \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v $(pwd)/examples/quick-start:$(pwd)/examples/quick-start \
      -v $(pwd)/output_add_sub:$(pwd)/output_add_sub \
      --net=host nvcr.io/nvidia/tritonserver:22.05-py3-sdk
```

\<path-to-output-model-repo>를 출력 모델 리포지토리가 위치할 디렉터리의 절대 경로로 바꿉니다. 이렇게 하면 Triton SDK 컨테이너가 Model Analyzer가 생성하는 모델 구성 변형에 액세스할 수 있습니다.\
중요: 마운트의 양쪽에서 절대 경로가 동일한지 확인해야 합니다.



**3. Add PDF support to the container**

Model Analyzer는 보고서 생성을 위해 pdfkit을 사용합니다. Triton SDK 컨테이너 안에 있으면 wkhtmltopdf를 설치해야 합니다.

```
apt-get update && apt-get install wkhtmltopdf
```
