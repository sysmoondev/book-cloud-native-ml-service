---
description: >-
  Triton은 모델 저장소에 변화가 있을때, 모델 로딩 관점에서 model control 모드(NONE, EXPLICIT, POLL)
  3가지를 제공하고, 모델 관리를 위한 Inference protocols API (HTTP/REST & gRPC, C API) 를
  제공합니다.
---

# 모델 관리

### **Model Control Mode**

\
triton은 시작할 때, model repository에 있는 모든 모델을 가져옵니다. 가져오지 못한 모델은 UNAVAILABLE 상태로 표시하고, inferencing 할 수 없습니다. 서버가 실행되는 동안에는 model repository 에 모델 관련 정보들이 변경 되도 무시됩니다.

model control protocol을 사용하여 모델을 불러오려는 요청에는 에러를 응답으로 반환한다. triton이 시작할 때, --model-control-mode=none 이 기본으로 설정된다. triton이 실행되는 동안, model repository를 변경시키는 것은 조심해야 한다.

### Model Control Mode EXPLICIT

시작 시 Triton은 --load-model 명령줄 옵션으로 명시적으로 지정된 모델만 로드합니다. --load-model을 지정하지 않으면 시작 시 모델이 로드되지 않습니다. Triton이 로드할 수 없는 모델은 UNAVAILABLE로 표시되고 추론에 사용할 수 없습니다.

시작 후 모든 모델 로드 및 언로드 작업은 모델 제어 프로토콜을 사용하여 명시적으로 시작되어야 합니다. 모델 제어 요청의 응답 상태는 로드 또는 언로드 작업의 성공 또는 실패를 나타냅니다. 이미 로드된 모델을 다시 로드하려고 할 때 어떤 이유로든 다시 로드가 실패하면 이미 로드된 모델은 변경되지 않고 로드된 상태로 유지됩니다. 다시 로드에 성공하면 모델의 가용성 손실 없이 새로 로드된 모델이 이미 로드된 모델을 대체합니다.

이 모델 제어 모드는 --model-control-mode=explicit을 지정하여 활성화됩니다. Triton이 실행되는 동안 모델 리포지토리를 변경하는 것은 모델 리포지토리 수정에서 설명한 대로 신중하게 수행해야 합니다.

### Model Control Mode POLL

Triton은 시작 시 모델 리포지토리의 모든 모델을 로드하려고 시도합니다. Triton이 로드할 수 없는 모델은 UNAVAILABLE로 표시되고 추론에 사용할 수 없습니다.\


모델 리포지토리에 대한 변경 사항이 감지되고 Triton은 이러한 변경 사항을 기반으로 필요에 따라 모델을 로드 및 언로드하려고 시도합니다. 이미 로드된 모델을 다시 로드하려고 할 때 어떤 이유로든 다시 로드가 실패하면 이미 로드된 모델은 변경되지 않고 로드된 상태로 유지됩니다. 다시 로드에 성공하면 모델에 대한 가용성 손실 없이 새로 로드된 모델이 이미 로드된 모델을 대체합니다.

Triton은 주기적으로 리포지토리를 폴링하기 때문에 모델 리포지토리에 대한 변경 사항이 즉시 감지되지 않을 수 있습니다. --repository-poll-secs 옵션을 활용하여 polling interval 을 제어할 수 있습니다. 이러한 특성으로 모델 저장소  변경 중에 Pooling 발생시, 변경이 부분적으로만 탐지될 수 있기 때문에 상용 서비스 환경에서는 POLL 모드는 권장하지 않습니다.\
\




