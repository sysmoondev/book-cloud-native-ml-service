# Configuration

Detectron2 플랫폼을 이용하면 사용자가 적용할 모델과 loss만 정의하면 학습부터 모델 포팅 그리고  최적화까지 자동화할 수 있습니다.

Detectron2 프로젝트 소스를 github 에서 clone 이후 디렉토리 구조는 아래와 같이 확인할 수 있습니다. 여러개의 폴더가 있지만 주요 기능이 포함된 tools, detectron2 폴더 구조에 대해 소개합니다.

```
tree -L 1 -d detectron2   
├── build
├── configs
├── datasets
├── demo
├── detectron2
├── detectron2.egg-info
├── dev
├── docker
├── docs
├── projects
├── tests
└── tools
```

### tools

학습을 위한 train\_net.py 파이썬 스크립트를 이용하여 model zoo 에서 제공하는 이미 학습된 모델을 이용합니다. 사용자는 학습에 필요한 데이터와 파라미터 값을 입력하여 Transfer Learning 이 가능합니다. 학습에 필요한 파이썬 스크립트는 plain\__train\__net.py, train\_net.py 2개의 파일로 구성되어 있습니다**.**&#x20;

**plain\_**_**train\_**_**net.py**

plain\__train\__net.py **** 파일은 추상화가 덜 되어있어 구조 파악이 쉽지만, SGD-momentum을 이용한 학습만 지원하고 나머지 기능들은 지원하지 않습니다.

**train\_net.py**

train\_net.py **** 파일은 training loop가 추상화가 되있어 실제 training loop가 어떻게 도는지 파악이 힘듭니다. Engine이라는 것을 사용해 학습을 하며, learning rate warmup과 같은 기능들을 사용해 학습을 진행합니다. 즉 이 engine을 이용하면 사용자는 학습과정은 신경쓰지 않고 오로지 모델과 loss만 신경쓰면 되는 구조입니다.

### detectron2

detectron2 폴더는 모델 및 학습을 하는 engine이 정의되어 있습니다. Detectron2는 engine을 통해 학습을 하며, 사용자는 모델구조**를** 변경하고 하이퍼파라미터 튜닝만 하면 됩니다.

detectron2 폴더 구조를 살펴보면 아래와 같이 구성되어 있습니다. 이 중에 사용자가 모 변경 없이 사용하기 위해서 config, engine, model\_zoo 에 대한 이해가 필요합니다.

```
tree -L 1 -d detectron2/detectron2                                                                                              [14:33:01]
detectron2/detectron2
├── __pycache__
├── checkpoint
├── config
├── data
├── engine
├── evaluation
├── export
├── layers
├── model_zoo
├── modeling
├── projects
├── solver
├── structures
├── tracking
└── utils
```

#### detectron2/detectron2/config

```
-rw-r--r--  1 sysmoon staff  599 Jan 25 16:56 __init__.py
drwxr-xr-x 14 sysmoon staff  448 Jan 25 20:29 __pycache__
-rw-r--r--  1 sysmoon staff 7.8K Jan 25 16:56 compat.py
-rw-r--r--  1 sysmoon staff 9.0K Jan 25 16:56 config.py
-rw-r--r--  1 sysmoon staff  29K Jan 25 16:56 defaults.py
-rw-r--r--  1 sysmoon staff 2.7K Jan 25 16:56 instantiate.py
-rw-r--r--  1 sysmoon staff  15K Jan 25 16:56 lazy.py
```

config 폴더는 학습/검증을 위한 파이썬 스크립트 사용시 필요한 하이퍼파라미터들이 정의되어 있고, 변경하여 사용 가능합니다.&#x20;

#### detectron2/detectron2/engine

engine은 FAIR 에서 Caffe 스타일의 training loop를 가져가다 Pytorch 로 구현했습니다. 이를 통해 학습 과정을 추상화할 수 있습니다.

#### detectron2/detectron2/model zoo

FAIR 에서 이미 학습된 모델을 관리하는 폴더 입니다. model\_zoo.py 파일을 열어보면 Detecton2 엔진에서 사용자가 필요한 모델을 사용할 수 있도록 S3 URL\_SUFFIX 통해서 관리하고 있습니다.

**모델 종류**

* COCO Detection with Faster R-CNN
* COCO Detection with RetinaNet
* COCO Detection with RPN and Fast R-CNN
* COCO Instance Segmentation Baselines with Mask R-CNN
* New baselines using Large-Scale Jitter and Longer Training Schedule
* COCO Person Keypoint Detection Baselines with Keypoint R-CNN
* COCO Panoptic Segmentation Baselines with Panoptic FPN
* LVIS Instance Segmentation Baselines with Mask R-CNN
* Cityscapes & Pascal VOC Baselines

```
# model_zoo.py
class _ModelZooUrls(object):
    """
    Mapping from names to officially released Detectron2 pre-trained models.
    """

    S3_PREFIX = "https://dl.fbaipublicfiles.com/detectron2/"

    # format: {config_path.yaml} -> model_id/model_final_{commit}.pkl
    CONFIG_PATH_TO_URL_SUFFIX = {
        # COCO Detection with Faster R-CNN
        "COCO-Detection/faster_rcnn_R_50_C4_1x": "137257644/model_final_721ade.pkl",
        "COCO-Detection/faster_rcnn_R_50_DC5_1x": "137847829/model_final_51d356.pkl",
        "COCO-Detection/faster_rcnn_R_50_FPN_1x": "137257794/model_final_b275ba.pkl",
        "COCO-Detection/faster_rcnn_R_50_C4_3x": "137849393/model_final_f97cb7.pkl",
        "COCO-Detection/faster_rcnn_R_50_DC5_3x": "137849425/model_final_68d202.pkl",
        "COCO-Detection/faster_rcnn_R_50_FPN_3x": "137849458/model_final_280758.pkl",
        "COCO-Detection/faster_rcnn_R_101_C4_3x": "138204752/model_final_298dad.pkl",
        "COCO-Detection/faster_rcnn_R_101_DC5_3x": "138204841/model_final_3e0943.pkl",
        "COCO-Detection/faster_rcnn_R_101_FPN_3x": "137851257/model_final_f6e8b1.pkl",
        "COCO-Detection/faster_rcnn_X_101_32x8d_FPN_3x": "139173657/model_final_68b088.pkl",
        # COCO Detection with RetinaNet
        "COCO-Detection/retinanet_R_50_FPN_1x": "190397773/model_final_bfca0b.pkl",
        "COCO-Detection/retinanet_R_50_FPN_3x": "190397829/model_final_5bd44e.pkl",
        "COCO-Detection/retinanet_R_101_FPN_3x": "190397697/model_final_971ab9.pkl",
        # COCO Detection with RPN and Fast R-CNN
        "COCO-Detection/rpn_R_50_C4_1x": "137258005/model_final_450694.pkl",
        "COCO-Detection/rpn_R_50_FPN_1x": "137258492/model_final_02ce48.pkl",
        "COCO-Detection/fast_rcnn_R_50_FPN_1x": "137635226/model_final_e5f7ce.pkl",
```



