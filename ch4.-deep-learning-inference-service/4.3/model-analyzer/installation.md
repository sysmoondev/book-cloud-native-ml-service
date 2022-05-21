# Installation

Model Analyzser 를 설치하는 방법은 크게 4가지가 있습니다.

* Dockerfile Build
* Triton SDK Container
* Pip3
* Source Build

위 방법 중에서 perf\_analyzer _를 테스트 했던 방식과 동일한 Triton SDK Container 환경에서 model\_analyzer 를 활용하기 위한 방법에 대해 소개합니다._

### _Triton SDK Container_

다음 명령을 사용하여 Triton SDK Container를 가져와 실행할 수 있습니다.

```
$ docker pull nvcr.io/nvidia/tritonserver:22.04-py3-sdk
```

\--triton-launch-mode=docker를 사용하여 모델 분석기를 실행할 계획이 아닌 경우 다음 명령을 사용하여 SDK 컨테이너를 실행할 수 있습니다.

```
$ docker run -it --gpus all --net=host nvcr.io/nvidia/tritonserver:22.04-py3-sdk
```

SDK 컨테이너를 사용할 때 Model Analyzer를 사용하는 이 방법과 함께 권장되는 --triton-launch-mode=docker를 사용하려는 경우 다음을 탑재해야 합니다.

* \-v /var/run/docker.sock:/var/run/docker.sock\
  Triton SDK 컨테이너 내부에서 형제 컨테이너로 도커 컨테이너를 실행할 수 있습니다. 모델 분석기는 --triton-launch-mode=docker와 함께 실행하는 경우 이를 요구합니다.
* `-v <path-to-output-model-repo>:<path-to-output-model-repo>`\
  ``출력 모델 리포지토리가 위치할 디렉터리의 절대 경로(즉, 출력 모델 리포지토리의 상위 디렉터리)입니다. 이는 실행된 Triton 컨테이너가 Model Analyzer가 생성하는 모델 구성 변형에 액세스할 수 있도록 하기 위한 것입니다.

```
$ docker run -it --gpus all \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v <path-to-output-model-repo>:<path-to-output-model-repo> \
      --net=host nvcr.io/nvidia/tritonserver:22.04-py3-sdk
```

Model Analyzer는 보고서 생성을 위해 pdfkit을 사용합니다. Triton SDK 컨테이너 내에서 Model Analyzer를 실행하는 경우 wkhtmltopdf를 다운로드해야 합니다.

```
$ sudo apt-get update && sudo apt-get install wkhtmltopdf
```

이렇게 하면 Model Analyzer가 pdfkit을 사용하여 보고서를 생성할 수 있습니다.
