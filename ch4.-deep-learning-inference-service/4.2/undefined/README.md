# 모델 저장소

Triton Inference Server 서버 실행시 Model Repository(모델 저장소) 마운트를 통해 하나 또는 여러개의 모델을 서빙할 수 있습니다.&#x20;

### Model Repository Layout

Model Repository 는 Triton 실행 시,  --model-repository 옵션을 이용하여 저장된 모델 저장소의 경로명을 지정할 수 있습니다.

```
$ tritonserver --model-repository=<model-repository-path>
```

&#x20;

Model Repository 의 Layout은 반드시 아래와 같은 형식을 따라야 합니다.

```
  <model-repository-path>/
    <model-name>/
      [config.pbtxt]
      [<output-labels-file> ...]
      <version>/
        <model-definition-file>
      <version>/
        <model-definition-file>
      ...
    <model-name>/
      [config.pbtxt]
      [<output-labels-file> ...]
      <version>/
        <model-definition-file>
      <version>/
        <model-definition-file>
      ...
    ...
```

위 형식의 요구사항은 다음의 규칙을 가지고 있습니다.

* Root 상단의 repository 디렉토리에는 1개 이상의 서브 디렉토리 필요 합니다.
* 각 서브 디렉토리는 해당되는 모델에 대한 repository 정보 포함해야 합니다.
* config.pbtxt 파일은 모델의 model configuration 정보 포함해야 합니다.
* config.pbtxt 는 특정 모델에서는 항상 필요하지 않을 수 있습니다. (auto-generated model configuration&#x20;

Triton Server 에서 제공하는 샘플 모델 저장소의 경우 아래와 같이 model\_repository Root 폴더에 하나 이상의 모델을 관리할 수 있습니다.&#x20;

```
tree server/docs/examples/model_repository

├── densenet_onnx
│   ├── 1
│   │   └── model.onnx
│   ├── config.pbtxt
│   └── densenet_labels.txt
├── inception_graphdef
│   ├── 1
│   │   └── model.graphdef
│   ├── config.pbtxt
│   └── inception_labels.txt
├── simple
│   ├── 1
│   │   └── model.graphdef
│   └── config.pbtxt
├── simple_dyna_sequence
│   ├── 1
│   │   └── model.graphdef
│   └── config.pbtxt
├── simple_identity
│   ├── 1
│   │   └── model.savedmodel
│   │       └── saved_model.pb
│   └── config.pbtxt
├── simple_int8
│   ├── 1
│   │   └── model.graphdef
│   └── config.pbtxt
├── simple_sequence
│   ├── 1
│   │   └── model.graphdef
│   └── config.pbtxt
└── simple_string
    ├── 1
    │   └── model.graphdef
    └── config.pbtxt
```
