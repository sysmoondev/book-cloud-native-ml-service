# Inference Demo with Pre-trained Models

이번 장에서는 Detectron2 에서 제공하는 built-in command-line tools 이용하여 미리 학습된 모델을 이용하여 추론하는 방법과, 사용자가 가진 Custom Data 를 이용하여 새롭게 학습(Transfer Learning) 하고, 평가하는 방법에 대해 소개합니다.

## Pre-trained Model 구성 및 사용법

Detectron2는 이미 학습된 모델을 [model zoo](https://github.com/facebookresearch/detectron2/blob/main/MODEL\_ZOO.md) 를 통해 제공하고 있습니다.

#### COCO Instance Segmentation Baselines with Mask R-CNN

| Name                                                                                                                                           | <p>lr<br>sched</p> | <p>train<br>time<br>(s/iter)</p> | <p>inference<br>time<br>(s/im)</p> | <p>train<br>mem<br>(GB)</p> | <p>box<br>AP</p> | <p>mask<br>AP</p> | model id  | download                                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | -------------------------------- | ---------------------------------- | --------------------------- | ---------------- | ----------------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [R50-C4](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_C4\_1x.yaml)            | 1x                 | 0.584                            | 0.110                              | 5.2                         | 36.8             | 32.2              | 137259246 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_C4\_1x/137259246/model\_final\_9243eb.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_C4\_1x/137259246/metrics.json)                   |
| [R50-DC5](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_DC5\_1x.yaml)          | 1x                 | 0.471                            | 0.076                              | 6.5                         | 38.3             | 34.2              | 137260150 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_DC5\_1x/137260150/model\_final\_4f86c3.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_DC5\_1x/137260150/metrics.json)                 |
| [R50-FPN](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_FPN\_1x.yaml)          | 1x                 | 0.261                            | 0.043                              | 3.4                         | 38.6             | 35.2              | 137260431 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_FPN\_1x/137260431/model\_final\_a54504.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_FPN\_1x/137260431/metrics.json)                 |
| [R50-C4](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_C4\_3x.yaml)            | 3x                 | 0.575                            | 0.111                              | 5.2                         | 39.8             | 34.4              | 137849525 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_C4\_3x/137849525/model\_final\_4ce675.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_C4\_3x/137849525/metrics.json)                   |
| [R50-DC5](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_DC5\_3x.yaml)          | 3x                 | 0.470                            | 0.076                              | 6.5                         | 40.0             | 35.9              | 137849551 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_DC5\_3x/137849551/model\_final\_84107b.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_DC5\_3x/137849551/metrics.json)                 |
| [R50-FPN](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_FPN\_3x.yaml)          | 3x                 | 0.261                            | 0.043                              | 3.4                         | 41.0             | 37.2              | 137849600 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_FPN\_3x/137849600/model\_final\_f10217.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_FPN\_3x/137849600/metrics.json)                 |
| [R101-C4](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_C4\_3x.yaml)          | 3x                 | 0.652                            | 0.145                              | 6.3                         | 42.6             | 36.7              | 138363239 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_C4\_3x/138363239/model\_final\_a2914c.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_C4\_3x/138363239/metrics.json)                 |
| [R101-DC5](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_DC5\_3x.yaml)        | 3x                 | 0.545                            | 0.092                              | 7.6                         | 41.9             | 37.3              | 138363294 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_DC5\_3x/138363294/model\_final\_0464b7.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_DC5\_3x/138363294/metrics.json)               |
| [R101-FPN](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_FPN\_3x.yaml)        | 3x                 | 0.340                            | 0.056                              | 4.6                         | 42.9             | 38.6              | 138205316 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_FPN\_3x/138205316/model\_final\_a3ec72.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_R\_101\_FPN\_3x/138205316/metrics.json)               |
| [X101-FPN](https://github.com/facebookresearch/detectron2/blob/main/configs/COCO-InstanceSegmentation/mask\_rcnn\_X\_101\_32x8d\_FPN\_3x.yaml) | 3x                 | 0.690                            | 0.103                              | 7.2                         | 44.3             | 39.5              | 139653917 | [model](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_X\_101\_32x8d\_FPN\_3x/139653917/model\_final\_2d9806.pkl) \| [metrics](https://dl.fbaipublicfiles.com/detectron2/COCO-InstanceSegmentation/mask\_rcnn\_X\_101\_32x8d\_FPN\_3x/139653917/metrics.json) |

이 중에 Instance Segmentation 모델(mask\_rcnn\_R\_50\_FPN\_3x.yaml)을 이용하여 추론 테스트를 진행합니다.

1. model zoo 에서 mask\_rcnn\_R\_50\_FPN\_3x.yaml 선택
2. demo.py 파이썬 데모 스크립트를 통해 model 경로를 설정하고, 추론에 필요한 input 이미지 데이터와 model weight 값 설정하여 추론 테스트 실행

Detectron2 Github 소스를 클론받은 이후 demo 폴더에 들어가면 아래와 같이 model zoo 에 있는 pre-trained 모델들을 이용하여 추론테스트가 가능합니다. 실행 방법은 아래와 같습니다.

```
cd demo/
python demo.py --config-file ../configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml \
  --input input1.jpg input2.jpg \
  [--other-options]
  --opts MODEL.WEIGHTS detectron2://COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x/137849600/model_final_f10217.pkl
```

The configs are made for training, therefore we need to specify `MODEL.WEIGHTS` to a model from model zoo for evaluation. This command will run the inference and show visualizations in an OpenCV window.

추론을 위한 모델 설정은 학습을 위해 만들어졌으므로 평가를 위해 model zoo 에 있는 모델의 MODEL.WEIGHTS를 지정해야 합니다. 위 명령은 사용자 입력 이미지(--input imput1.jpg)에 대해 추론을 실행하고 OpenCV 창에 시각화를 Instance Segmentaion에 대한 결과를 보여줍니다.

demo.py 스크립에서 사용가능한 인자들은 -h 옵션을 사용하여 확인 가능합니다.

```
python demo.py -h
usage: demo.py [-h] [--config-file FILE] [--webcam]
               [--video-input VIDEO_INPUT] [--input INPUT [INPUT ...]]
               [--output OUTPUT] [--confidence-threshold CONFIDENCE_THRESHOLD]
               [--opts ...]

Detectron2 demo for builtin configs

optional arguments:
  -h, --help            show this help message and exit
  --config-file FILE    path to config file
  --webcam              Take inputs from webcam.
  --video-input VIDEO_INPUT
                        Path to video file.
  --input INPUT [INPUT ...]
                        A list of space separated input images; or a single
                        glob pattern such as 'directory/*.jpg'
  --output OUTPUT       A file or directory to save output visualizations. If
                        not given, will show output in an OpenCV window.
  --confidence-threshold CONFIDENCE_THRESHOLD
                        Minimum score for instance predictions to be shown
  --opts ...            Modify config options using the command-line 'KEY
                        VALUE' pairs
```

이 중에서 공통으로 사용 가능한 옵션들을 보면

* 입력 데이터를 **webcam 영상데이터로 사용하기 위해**, `--input file 대신에 --webcam 파라미터 사용.`
* 입력 데이터를 **on a video 영상데이터로 사용하기 위해** `--input files` 대신에 `--video-input video.mp4 파라미터 사용`
* **cpu 모드로 추론하기 위해** --opts 뒤에 `MODEL.DEVICE cpu 파라터터 사용`
* 추론 결과물 파일 이미지 또는 동영상 파일 (webcam or video)로 저장기 위한 경로명으로 --output 파라미터 사용.

### Instance Segmentaion 추론 테스트

GPU가 없는 MacOS 환경에서 위 Instance Segmetaion 추론 테스트를 위해 샘플 이미지를 다운로드 받고, 해당 이미지로 추론 결과물을 확인하는 과정을 소개합니다.

**샘플 이미지 다운로드**

추론에 필요한 샘플 이미지를 cocodataset.org 사이트를 통해 다운로 합니다.

```
cd demo
wget http://images.cocodataset.org/val2017/000000439715.jpg -q -O input.jpg
```

![](../../.gitbook/assets/input.jpg)

#### 추론 테스트 실행 (demo.py)

위 샘플 이미지를 이용하여 CPU 모드로 추론하도록 아래 파라미터 값들을 입력하여 demo.py 스크립트를 실행합니다. 이후 OpenCV 창이 뜨면서 Instance Segmentaion 의 결과값을 확인할 수 있습니다.

```
python demo.py --config-file ../configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml \                                                                                [20:28:41]
  --input input.jpg \
  --opts MODEL.DEVICE cpu MODEL.WEIGHTS detectron2://COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x/137849600/model_final_f10217.pkl
```

![](../../.gitbook/assets/output\_instance\_segmentaion\_01.png)

추론 결과물과 같이 원본 사진에서 Instance Segmentaion 을 통해 Object Detection 하고, 분류(Classification)알고리즘을 통해 Person, Horse, Umbrella 객체를 마스킹 하여 표현합니다.

## Ref

* [QT plugin install](https://log-mylife.tistory.com/entry/Could-not-load-the-Qt-platform-plugin-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%EB%B2%95)
