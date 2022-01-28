# Training & Evaluation

Detectron2는 에서 이미 제공하고 있는 스크립트 도구를 이용하여 commandl-line을 통해 학습과 추론 관련한  다양한 테스트가 가능합니다. 또한 [Colab Notebook](https://colab.research.google.com/drive/16jcaJoc6bCFAQ96jDe2HwtXj7BMD\_-m5) 환경에서 이미 학슴된 모델을 이용하여 손쉽게 추론 테스트 가능합니다. 따라서 Detectron2 개발환경을 구축하여 테스트하기 어려운 환경이라면 Colab 환경에서 쉽게 Detectron2 주요기능에 대해 테스트 가능합니다.



## Inference Demo with Pre-trained Model

이미 학습된 모델을 이용하여 추론하기 위한 테스트 준비를 다음과 같이 진행합니다.

1. [model zoo](https://github.com/facebookresearch/detectron2/blob/main/MODEL\_ZOO.md) 에서 이미 학습된 모델 중 Instance Segmentaion 모델(mask\_rcnn\_R\_50\_FPN\_3x.yaml) 선택
2. 파이썬 스크립트 demo.py 스크립트를 이용하여 추론 테스트

추론에 필요한 샘플 입력 이미지 데이터를 다운로드 합니다.

```
cd /demo
wget http://images.cocodataset.org/val2017/000000439715.jpg -q -O input.jpg
```

demo.py 스크립트에 사용되는 설정들은 학습을 위해 구성되어 있습니다. 따라서모델 평가를 위한 Weight 정보를 입력 파라미터로 사용할 수 있고, 그 밖에 필요한 다양한 파라미터는 demo.py 에서 제공하는 입력파라미터 조회를 통해 확인 가능합니다.

```
python demo.py -h

usage: demo.py [-h] [--config-file FILE] [--webcam] [--video-input VIDEO_INPUT] [--input INPUT [INPUT ...]]
               [--output OUTPUT] [--confidence-threshold CONFIDENCE_THRESHOLD] [--opts ...]

Detectron2 demo for builtin configs

optional arguments:
  -h, --help            show this help message and exit
  --config-file FILE    path to config file
  --webcam              Take inputs from webcam.
  --video-input VIDEO_INPUT
                        Path to video file.
  --input INPUT [INPUT ...]
                        A list of space separated input images; or a single glob pattern such as 'directory/*.jpg'
  --output OUTPUT       A file or directory to save output visualizations. If not given, will show output in an
                        OpenCV window.
  --confidence-threshold CONFIDENCE_THRESHOLD
                        Minimum score for instance predictions to be shown
  --opts ...            Modify config options using the command-line 'KEY VALUE' pairs
```

주요 파라미터에 대한 정보를 살펴보면

* \--webcam: 추론을 위한 입력 데이터로 이미지 파일 대신에 웹카메라의 영상 데이터 사용
* \--video-input: 추론을 위한 입력 데이터로 이미지 파일 대신에 동용상 데이터 사용 (ex: video.mp4)
* \--cpu: GPU 지원이 없는 환경에서 CPU 모드로 추론 테스트 실행 (--ops 뒤에 추가)
* \--output: 추론 결과(images 파일 또는 webcam, video 동영상 파일)를 특정 디렉토리 경로명에 저장



GPU가 없는 MacOS 환경의 경우 --cpu 옵션을 사용하여 아래와 같이 추론 테스트가 가능하고, 추론의 결과물은 기본적으로 OpenCV 창을 통해 보여주고, --output 옵션을 지정하는 경우 해당 경로명에 결과값을 저장되어 확인 가능합니다.

```
python demo.py --config-file ../configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml \
--output ./output \
--input input.jpg \
--opts MODEL.WEIGHTS detectron2://COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x/137849600/model_final_f10217.pkl
```

{% hint style="info" %}
GPU가 없는 MacOS 환경의 경우 --cpu 옵션을 사용하여 아래와 같이 추론 테스트가 가능합니다.

```
python demo.py --config-file ../configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml \
--output ./output \
--input input.jpg \
--opts MODEL.DEVICE cpu \
MODEL.WEIGHTS detectron2://COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x/137849600/model_final_f10217.pkl
```
{% endhint %}

위 추론테스트를 실행하면 ./output/input.jpg 결과물 즉, Instance Segmentaion 모델을 통해 각 객체를 인식하고, 해당 객체를 마스킹하여 분류한 결과를 확인할 수 있습니다.

![](../../.gitbook/assets/output\_instance\_segmentaion\_01.png)



## Training & Evaluation in Command Line

