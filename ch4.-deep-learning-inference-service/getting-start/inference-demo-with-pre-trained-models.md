# Inference Demo with Pre-trained Models

이번 장에서는 Detectron2 에서 제공하는 built-in command-line tools 이용하여 미리 학습된 모델을 이용하여 추론하는 방법과, 사용자가 가진 Custom Data 를 이용하여 새롭게 학습(Transfer Learning) 하고, 평가하는 방법에 대해 소개합니다.

## Pre-trained Model 추론 데모

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



이 중에 Instance Segmentation 모델(mask\_rcnn\_R\_50\_FPN\_3x.yaml)을 이용하여 추론 테스트가 가능합니다.

1. model zoo 에서 mask\_rcnn\_R\_50\_FPN\_3x.yaml 선택
2. demo.py 파이썬 데모 스크립트를 통해 model 경로를 설정하고, 추론에 필요한 input 이미지 데이터와 model weight 값 설정하여 추론 테스트 실행

```
cd demo/

# download sample image for inferencing
wget http://images.cocodataset.org/val2017/000000439715.jpg -q -O input.jpg

# inference
python demo.py --config-file ../configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml \
  --input input.jpg \
  --opts MODEL.WEIGHTS detectron2://COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x/137849600/model_final_f10217.pkl
```

&#x20;
