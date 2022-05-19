# 모델 버전 및 디렉토리 구조

### **Model versions**

각 모델은 repository 안에서 여러 버전을 가질 수 있습니다. 각 모델별 버전을 숫자로 지정된 하위 디렉토리에 관리합니다. 버전은 1부터 시작하며, 숫자가 아니거나 0으로 시작하는 디렉토리는 무시됩니다.

```
tree 
├── densenet_onnx
│   ├── 1
│   │   └── model.onnx
│   ├── 2
│   │   └── model.onnx
│   ├── 3
│   │   └── model.onnx
│   ├── config.pbtxt
│   └── densenet_labels.txt
```

### **Model Files**

모델별 하위 디렉토리 구조는 모델의 타입과 모델을 지원하는 Backend 에 따라 모델 [설정 방법](https://github.com/triton-inference-server/server/blob/main/docs/model\_configuration.md)이 결정됩니다.&#x20;

#### TensorRT

모델은 Plan으로 정의되고, 모델의 이름은 반드시 model.plan 파일 이름의 규칙을 따라야 합니다.

```
  <model-repository-path>/
    <model-name>/
      config.pbtxt
      1/
        model.plan 
```

#### ONNX Modles

onnx 모델은 하나의 파일 이거나 여러 파일을 가지는 디렉토리 입니다. 모델의 이름이 model.onnx 이어야 한다. Triton은 Triton에서 사용 중인 ONNX 런타임 버전에서 지원하는 모든 ONNX 모델을 지원합니다. 오래된 ONNX opset 버전을 사용하거나 지원되지 않는 유형의 연산자가 포함된 모델은 지원되지 않습니다.

```
  <model-repository-path>/
    <model-name>/
      config.pbtxt
      1/
        model.onnx
```

#### TorchScript Models

TorchScript 모델은 기본적으로 model.pt로 이름이 지정되어야 하는 단일 파일입니다. 이 기본 이름은 모델 구성의 default\_model\_filename 속성을 사용하여 재정의할 수 있습니다.

```
  <model-repository-path>/
    <model-name>/
      config.pbtxt
      1/
        model.pt
```

#### TensorFlow Models

TensorFlow는 GraphDef 또는 SavedModel의 두 가지 형식의 모델을 저장을 지원하고, Triton 에서 모두 지원합니다.&#x20;

* TensorFlow GraphDef

TensorFlow GraphDef는 기본적으로 model.graphdef로 모델 이름이 지정되어야 하는 단일 파일입니다.

```
  <model-repository-path>/
    <model-name>/
      config.pbtxt
      1/
        model.graphdef
```

* TensorFlow SavedModel

TensorFlow SavedModel은 여러 파일을 포함하는 디렉토리 입니다. 기본적으로 디렉토리 이름은 model.savedmodel이어야 합니다. 이러한 기본 이름은 모델 구성의 default\_model\_filename 속성을 사용하여 재정의할 수 있습니다.

```
  <model-repository-path>/
    <model-name>/
      config.pbtxt
      1/
        model.savedmodel/
           <saved-model files>
```

#### OpenVINO Models

OpenVINO 모델은 \*.xml 및 \*.bin 파일의 두 파일로 표시됩니다. 기본적으로 \*.xml 파일의 이름은 model.xml이어야 합니다. 이 기본 이름은 모델 구성의 default\_model\_filename 속성을 사용하여 재정의할 수 있습니다.

```
  <model-repository-path>/
    <model-name>/
      config.pbtxt
      1/
        model.xml
        model.bin
```

#### Python Models

Python 백엔드를 사용하면 Python 코드를 Triton 내에서 모델로 실행할 수 있습니다. 기본적으로 Python 스크립트의 이름은 model.py 이름 규칙이어야 하고, 이 기본 이름은 모델 구성의 default\_model\_filename 속성을 사용하여 재정의할 수 있습니다.

```
  <model-repository-path>/
    <model-name>/
      config.pbtxt
      1/
        model.py 
```

#### DALI Models

DALI 백엔드는 triton에서 DALI 파이프라인이 하나의 모델로서 실행 가능하도록 합니다. 이를 위해 DALI 파이프라인을 model.dali 라는 이름의 파일을 생성해야 한다. model.dali 파일을 생성하기 위해 [DALI backend documentation](https://github.com/triton-inference-server/dali\_backend#how-to-use) 을 참고할 수 있습니다.

```
  <model-repository-path>/
    <model-name>/
      config.pbtxt
      1/
        model.dali
```

