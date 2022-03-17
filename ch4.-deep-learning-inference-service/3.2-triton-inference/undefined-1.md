# 모델 구성

* [Model Configuration](https://benlee73.tistory.com/46?category=1030892)
* [Model Management](https://benlee73.tistory.com/58?category=1030892)
* [Model Repositor](https://benlee73.tistory.com/56?category=1030892)
* [Triton Backend](https://benlee73.tistory.com/51?category=1030892)



## 모델 저장소 설정

일반적으로 추론을 하기 위해선 미리 학습된 모델이 필요합니다. TRTIS도 미리 모델을 설정해줘서 관리 및 실행을 할 수 있습니다. TRTIS는 정해진 모델 폴더 양식이 따로 있습니다. 모델의 저장소는 아래와 같은 레이아웃 형태를 지켜야합니다. 후에, 로컬 서버에 모델과 레이아웃을 지켜준 폴더를 만들어주면, 도커로 run 할 때 docker의 -v 옵션을 통해서 컨테이너와 연결을 시켜 줍니다.&#x20;

* 서버에 올릴 모델을 관리할 폴더 생성 (ex. /home/models)
* 폴더안에 모델을 포함한 폴더들을 넣어준다(ex. /home/models/feature\_onnx).
* 이때 모델을 포함한 폴더들은 특정한 구조를 가짐
* models 폴더를 컨테이너에 볼륨 마운트를 해준다

&#x20;

Triton 구성 디렉토리 구조

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

위 디렉토리 구조에 맞게 Dense 샘플 모델을 아래와 같이 구성 가능합니다.

```
desnset_models/        	  		<모델 이름을 이용하여 모델을 관리할 폴더 생성>
	densenet/             		<지정하고 싶은 모델명 설정>
		config.pbtxt		<모델 구성 파일. 반드시 config.pbtxt 이름으로 지정>
		1/              	<모델 버전>
			model.plan	<TensorRT: plan, onnx: onnx ML프레임워크 별 모델 확장자를 선택하고, 파일 이름은 model로 고정>
		2/
			model.plan
		...
	...
```

모델 저장소 안에는 여러 모델 이름 폴더들이 있고 그 안에는 각 모델의 정보가 담긴 config.pbtxt 파일과 그 아래에는 그 모델의 버전 폴더가 있고 각 버전안에는 모델이 들어가 있습니다. 로컬 서버에 첨부된 test\_models 폴더를 넣어줍니다.

config.pbtxt는 모델의 초기 설정과 같은 config 파일이고, 인풋 사이즈와 아웃풋 사이즈를 정해주고 이 모델이 TensorRT인지 Pytorch인지 Tensorflow인지 알려주면서, 배치사이즈 최대 크기는 몇으로 둬야 할지 등등 미리 정해줍니다.

참고로, 후에 docker run 할때 명령어 중 --strict-model-config=false 옵션을 사용하면, ONNX나 TensorFlow 모델에선 따로 config.pbtxt 파일을 제공할 필요가 없습니다. 여기선 TensorRT이므로 설정을 해주어야 합니다.

아래는 Detectron2 모델을 위한 TensorRT의 config.pbtxt 파일 입니다.

```
platform: "tensorrt_plan"
max_batch_size: 1
input [
  {
    name: "input"
    data_type: TYPE_FP32
    dims: [ 3, 540, 960 ]
  }
]
output [
  {
    name: "output"
    data_type: TYPE_FP32
    dims: [ 3, 1080, 1920 ]
  }
]
dynamic_batching { }
```

name은 Pytorch에서 onnx를 만들때 넣은 input 이름을 넣어주면 되고 output도 마찬가지로 onnx 만들때의 output 이름을 넣어주면 됩니다. 데이터 타입은 TensorRT의 Input, Output의 데이터 타입을 넣어주면 되고, dims는 onnx를 만들때 shape를 넣어주면 됩니다. 같이 첨부된 EDSR 모델은 이미지 input이 960x540이고 output이 1920x1080이므로 dims를 위와 같이 작성하였습니다.

dynamic\_batching은 작업이 여러개 들어오면 동적으로 배치를 할당하여 더 빠르게 추론을 하는 것인데, max\_batch\_size가 1이면 이점을 가져갈순 없지만, max\_batch\_size가 2 이상일때 이점을 가져갈 수 있습니다. max\_batch\_size가 1이여도 오류가 따로 생기진 않으니 넣어줍시다.

\


\
\
\
\
