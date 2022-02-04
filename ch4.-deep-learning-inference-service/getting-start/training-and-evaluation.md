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

Detecton2는 학습에 필요한 여러 config 환경설정을 이용하여 쉽게 학습 가능하도록 2개의 파이썬 스크립트를 제공합니다. 제공되는 파이썬 학습코드를 이용하여 사용자의 용도에 맞게 커스터마이징 하여 사용 가능합니다.

* tools/plain\__train\_net.py_
* tools/train\_net.py

plain_train_net.py 스크립트는 plain\_net.py 에 비해 더 적은 기능들로 추상화 되어있기 때문에 사용자가 커스터마이징하여 학습에 필요한 추가적인 기능을 확장하기 더 용이합니다.

### Datasets 준비

train\_net.py 스크립트를 이용하여 학습을 시작하기 전에 학습에 필요한 COCO 형식의 [Datasets](https://github.com/facebookresearch/detectron2/blob/main/datasets/README.md) 준비가 필요합니다. Detectron2 는 Built-in Datasets 을 제공하고 있고, 환경변수(DETECTRON2\_DATASETS)에 지정된 경로명에 있는 Datasets 을 이용하여 학습데이터로 활용합니다. Datasets 파일 디렉토리 구조는 다음과 같습니다.

```
$DETECTRON2_DATASETS/
  coco/
  lvis/
  cityscapes/
  VOC20{07,12}/
```

기본적으로 Datasets 경로명을 환경변수로 지정하지 않으면 train\_net.py 스크립트는 현재 디렉토리 기준으로 ./datasets 경로명을 참조합니다.

model zoo 에서 제공하는 model 과 config 는 builtin datasets 을 사용하기 위해 DETECTRON2\_DATASETS 환경변수를 이용하여 dataset 경로명을 참조하기 때문에 학습에 필요한 데이터를 해당 경로명에 저장하여 관리가 필요합니다.

#### Buitin Dataset 폴더 구조

model zoo 에서 제공하는 이미 학습된 모델 Instance Segmentaion 을 이용하여 COCO Instance/keypoint detection datasets 으로 Transfer Learning 하는 과정을 소개합니다.

$DETECTRON\_DATASET 경로명에 아래와 같은 폴더 구조로 데이터 저장이 필요합니다.

```
coco/
  annotations/
    instances_{train,val}2017.json
    person_keypoints_{train,val}2017.json
  {train,val}2017/
    # image files that are mentioned in the corresponding json
```

annotaions 폴더에는 학습(train)/검증(val) 위한 실제 이미지 데이터에 대한 메타정보를 포함하고 있고, {train,val}2017/ 폴더에는 실제 학습할 이미지 데이터를 포함하고 있습니다.



#### COCO Instance/keypoint  Datasets 다운로드

\
[COCO Site](https://cocodataset.org/#download) 홈페이지에 접속하여 annotations(2017 Train/Val annotations), Images(2017 Train Images) 2개 Datasets 을 다운로드 합니다.

```
# Annotaions
wget http://images.cocodataset.org/annotations/annotations_trainval2017.zip
unzip annotations_trainval2017.zip

# Train
http://images.cocodataset.org/zips/train2017.zip
unzip train2017.zip
```

압축을 풀면 다음과 같은 데이터를 확인할 수 있습니다.

* &#x20;Annotations

```
# Annotations
vpsdev@bd-k8s-vps-dev-worker05:~/book/cocodataset$ tree annotations/
annotations/
├── captions_train2017.json
├── captions_val2017.json
├── instances_train2017.json
├── instances_val2017.json
├── person_keypoints_train2017.json
└── person_keypoints_val2017.json
```

* train2017&#x20;

```
# Train
vpsdev@bd-k8s-vps-dev-worker05:~/book/cocodataset$ tree train2017
├── 000000581909.jpg
├── 000000581913.jpg
├── 000000581921.jpg
└── 000000581929.jpg

0 directories, 118287 files
```

위 압축을 풀은 데이터를 model zoo 에서 읽어서 학습할 수 있도록 $DETECTRON\_DATASETS 환경변수 경로명에 아래과 같은 형태로 복사/이동 합니다.

```
detectron2/datasets/coco/
├── annotations
    ├── instances_train2017.json
    ├── instances_val2017_100.json
    ├── person_keypoints_train2017.json
    └── person_keypoints_val2017_100.json
├── train2017
    ├── 000000581909.jpg
    ├── 000000581913.jpg
    ├── 000000581921.jpg
    └── 000000581929.jpg
└── val2017
    ├── 000000546964.jpg
    ├── 000000550797.jpg
    ├── 000000554291.jpg
    ├── 000000556498.jpg
    └── 000000570688.jpg
```



### Training

Detectron2 에서 builint datasets 으로 사용할 경로명을 $DETECTRON\_DATASETS 환경변수를 통해 설정합니다. 현재 저자의 경우 instance/keypoint dataset 다운로드 받은 후, detectron2/datasets/coco/ 에 저장/관리하고 있으므로 아래와 같은 전체 경로명으로 설정합니다.

```
export DETECTRON_DATASETS=/home/vpsdev/book/detectron2/datasets
```

이후 train\_net.py 파이썬 스크립트를 통해 학습을 시작합니다.

```
cd tools/
./train_net.py --num-gpus 8 \
  --config-file ../configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_1x.yaml

```

만약 GPU 리소스가 부족한 환경이라면 [파라미터](https://arxiv.org/abs/1706.02677) 조정을 통해 아래와 같이 GPU 1개와 Batch Job 스케쥴링 정책을 수정하여 환경에 맞게 학습을 진행합니다.

```
cd demo/
./train_net.py \
  --config-file ../configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_1x.yaml \
  --num-gpus 1 SOLVER.IMS_PER_BATCH 2 SOLVER.BASE_LR 0.0025
```

train\_net.py 파라미터&#x20;

* \--config-file: model zoo 에 이미 학습된 모델의 정보가 담긴 yaml 파일의 경로명 설정 (../configs/COCO-InstanceSegmentation/mask\_rcnn\_R\_50\_FPN\_1x.yaml)
* \--num-gpus: 학습에 사용된 GPU 개수, SOVER Batch 개수, SOLVER LR 값&#x20;

Training 각 Iteration 각 과정의 상태정보는 다음과 같이 확인 가능합니다.&#x20;

```
[01/28 05:50:20 d2.utils.events]:  eta: 3:38:57  iter: 46419  total_loss: 0.9065  loss_cls: 0.2416  loss_box_reg: 0.2231  loss_mask: 0.2867  loss_rpn_cls: 0.03078  loss_rpn_loc: 0.05532  time: 0.2958  data_time: 0.0030  lr: 0.0025  max_mem: 2890M
[01/28 05:50:26 d2.utils.events]:  eta: 3:38:58  iter: 46439  total_loss: 1.126  loss_cls: 0.3322  loss_box_reg: 0.3052  loss_mask: 0.3295  loss_rpn_cls: 0.06258  loss_rpn_loc: 0.08123  time: 0.2958  data_time: 0.0033  lr: 0.0025  max_mem: 2890M
[01/28 05:50:31 d2.utils.events]:  eta: 3:38:45  iter: 46459  total_loss: 1.14  loss_cls: 0.3257  loss_box_reg: 0.2855  loss_mask: 0.3244  loss_rpn_cls: 0.07231  loss_rpn_loc: 0.08013  time: 0.2958  data_time: 0.0033  lr: 0.0025  max_mem: 2890M
[01/28 05:50:37 d2.utils.events]:  eta: 3:38:46  iter: 46479  total_loss: 1.008  loss_cls: 0.2984  loss_box_reg: 0.2428  loss_mask: 0.32  loss_rpn_cls: 0.05181  loss_rpn_loc: 0.07054  time: 0.2958  data_time: 0.0033  lr: 0.0025  max_mem: 2890M
[01/28 05:50:43 d2.utils.events]:  eta: 3:39:02  iter: 46499  total_loss: 1.225  loss_cls: 0.327  loss_box_reg: 0.313  loss_mask: 0.3425  loss_rpn_cls: 0.07357  loss_rpn_loc: 0.1112  time: 0.2958  data_time: 0.0036  lr: 0.0025  max_mem: 2890M
[01/28 05:50:49 d2.utils.events]:  eta: 3:38:38  iter: 46519  total_loss: 0.9217  loss_cls: 0.2329  loss_box_reg: 0.2465  loss_mask: 0.316  loss_rpn_cls: 0.06321  loss_rpn_loc: 0.09226  time: 0.2958  data_time: 0.0035  lr: 0.0025  max_mem: 2890M
[01/28 05:50:55 d2.utils.events]:  eta: 3:38:32  iter: 46539  total_loss: 0.9715  loss_cls: 0.2674  loss_box_reg: 0.2509  loss_mask: 0.2951  loss_rpn_cls: 0.03995  loss_rpn_loc: 0.05087  time: 0.2958  data_time: 0.0033  lr: 0.0025  max_mem: 2890M
[01/28 05:51:01 d2.utils.events]:  eta: 3:38:26  iter: 46559  total_loss: 1.064  loss_cls: 0.2992  loss_box_reg: 0.2865  loss_mask: 0.319  loss_rpn_cls: 0.05752  loss_rpn_loc: 0.08471  time: 0.2958  data_time: 0.0032  lr: 0.0025  max_mem: 2890M
```

학습이 정상적으로 종료되면 학습을 실행한 위치에 output 폴더가 생성되고 학습과정의 각 Epoch 과정에서 생성된 상태 정보(Checkpoint) 관련 파일들과 학습에 사용된 설정파일(config.yaml) 파일 및 관련 로그파일들을 확인할 수 있습니다.&#x20;

```
tree output/
output/
├── config.yaml
├── events.out.tfevents.1643334974.bd-k8s-vps-dev-worker05.11572.0
├── events.out.tfevents.1643335271.bd-k8s-vps-dev-worker05.26319.0
├── events.out.tfevents.1643355431.bd-k8s-vps-dev-worker05.21886.0
├── events.out.tfevents.1643845843.bd-k8s-vps-dev-worker05.9764.0
├── inference
│   ├── coco_instances_results.json
│   └── instances_predictions.pth
├── last_checkpoint
├── log.txt
├── metrics.json
├── model_0004999.pth
├── model_0009999.pth
├── model_0014999.pth
├── model_0019999.pth
├── model_0024999.pth
├── model_0029999.pth
├── model_0034999.pth
├── model_0039999.pth
├── model_0044999.pth
├── model_0049999.pth
├── model_0054999.pth
├── model_0059999.pth
├── model_0064999.pth
├── model_0069999.pth
├── model_0074999.pth
├── model_0079999.pth
├── model_0084999.pth
├── model_0089999.pth
└── model_final.pth
```

last\_checkpoint 는 각 Epoch 단계별 마지막 학습된 모델 정보를 포함하고 있으며, 위 과정은 모든 학습과정이 완료되었기 때문에 model-final.pth 파일의 정보를 담고 있는 것을 확인할 수 있습니다. 따라서 최종 학습이 완료되어 생성된 모델 파일을 이용하여 검증용 데이터를 이용하여 추론 성능을 평가할 수 있습니다.

```
cat output/last_checkpoint
model_final.pth
```

### Evaluation

학습이 끝난 이후, 생성된 모델 (/path/to/checkpoint\_file) 에 대해 Weight 값(MODEL.WEIGHTS)로 입력하여 평가 합니다.

```
./train_net.py --config-file ../configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_1x.yaml \
               --eval-only MODEL.WEIGHTS \
               output/model_final.pth
```

평가가 정상적으로 완료되면 아래와 같이 각 이미지 카테고리 별 성능 평가에 관련 주요 지표를 테이블 형태로 확인할 수 있습니다.

```
[02/04 00:30:40 d2.evaluation.coco_evaluation]: Evaluation results for segm: 
|   AP   |  AP50  |  AP75  |  APs   |  APm   |  APl   |
|:------:|:------:|:------:|:------:|:------:|:------:|
| 34.791 | 53.597 | 36.895 | 15.648 | 37.585 | 51.922 |
[02/04 00:30:40 d2.evaluation.coco_evaluation]: Per-category segm AP: 
| category      | AP      | category     | AP     | category       | AP     |
|:--------------|:--------|:-------------|:-------|:---------------|:-------|
| person        | 39.900  | bicycle      | 13.020 | car            | 31.346 |
| motorcycle    | 25.898  | airplane     | nan    | bus            | 62.575 |
| train         | 80.000  | truck        | 8.412  | boat           | 18.781 |
| traffic light | 13.642  | fire hydrant | 31.386 | stop sign      | 0.000  |
| parking meter | nan     | bench        | 26.006 | bird           | 24.157 |
| cat           | 46.238  | dog          | 22.574 | horse          | 73.119 |
| sheep         | nan     | cow          | 10.000 | elephant       | 51.584 |
| bear          | 56.950  | zebra        | 40.706 | giraffe        | 72.525 |
| backpack      | 24.017  | umbrella     | 45.961 | handbag        | 15.045 |
| tie           | 3.200   | suitcase     | nan    | frisbee        | 42.067 |
| skis          | 4.147   | snowboard    | 0.000  | sports ball    | 72.051 |
| kite          | 41.963  | baseball bat | 26.386 | baseball glove | 80.000 |
| skateboard    | 0.000   | surfboard    | 6.733  | tennis racket  | 25.439 |
| bottle        | 14.572  | wine glass   | 59.142 | cup            | 22.239 |
| fork          | 7.574   | knife        | nan    | spoon          | 0.000  |
| bowl          | 0.578   | banana       | 90.000 | apple          | 92.525 |
| sandwich      | 10.033  | orange       | nan    | broccoli       | 40.290 |
| carrot        | 3.472   | hot dog      | nan    | pizza          | 10.173 |
| donut         | 36.142  | cake         | nan    | chair          | 6.535  |
| couch         | 30.579  | potted plant | 10.769 | bed            | 55.347 |
| dining table  | 8.339   | toilet       | 50.297 | tv             | 69.950 |
| laptop        | 71.584  | mouse        | 39.901 | remote         | 1.030  |
| keyboard      | 55.248  | cell phone   | 9.010  | microwave      | 90.000 |
| oven          | 45.446  | toaster      | nan    | sink           | 45.446 |
| refrigerator  | 100.000 | book         | 5.885  | clock          | 41.485 |
| vase          | 16.359  | scissors     | nan    | teddy bear     | 90.000 |
| hair drier    | nan     | toothbrush   | nan    |                |        |
```
