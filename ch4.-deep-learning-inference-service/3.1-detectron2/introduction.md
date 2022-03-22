# Introduction

![](https://user-images.githubusercontent.com/1381301/66535560-d3422200-eace-11e9-9123-5535d469db19.png)

Detectron2는 Facebook AI Research(FAIR)에서 개발한 Pytorch 머신러닝 프레임워크 기반의 Object Detection, Segmentaion을 위한 [오픈소스](https://github.com/facebookresearch/detectron2) 프로젝트 입니다. 디지털 영상 이미지를 분석하여 객체를 찾아내고 여러 요소들을 분리하기 위한 프로젝트라고 정의할 수 있습니다.

Cloud Natvie 환경에서 영상 데이터를 기반으로 객체를 인식하고 분류하기 위한 Inferencing Service 샘플 모델을 찾기 위해 고민하던 중에 Detectron2를 적용하기로 했습니다. 이 선택의 가장 큰 이유는 Detectron2는 딥러닝 전체 과정을 처음부터 개발하는 과정 없이 모듈화된 디자인을 통해 확장 가능한 프레임워크를 제공합니다. 딥러닝 모델 연구를 흔히 아이(연구자)가 블록(레이어)를 안정적으로 쌓아 올리는 과정에 비유합니다. 그런데 아이가 밑바닥부터 블록을 쌓아 모델을 완성하는 작업은 굉장히 어렵습니다. 그렇기에 완성된 블록(detectron2)를 이용해 개발을 하면 굉장히 편리하게 개발을 할 수있습니다.

Github 딥러닝 관련 오픈프로젝트 중엣는 잘못 구현되거나, 최적화가 되지 않는 코드들이 많습니다. Detectron2 는 FAIR 에서 체계적인 모듈화 방식을 통해 최적화시킨 모델이며 [벤츠마크](https://detectron2.readthedocs.io/en/latest/notes/benchmarks.html) 성능상에서도 다른 오픈소스 대비 좋은 결과를 보여주고 있습니다. Detectron2가 다른 오픈 소스들에 비해 빠른이유는 python 최적화가 잘되어있기도 합니다만, 그 외에 연산량이 많이 드는 부분을 python이 아닌 CUDA와 C로 구현했기에 보다 좋은 성능을 냈습니다. box iou를 계산하는 부분이나, defromable conv 부분 등은 연산량이 많이 드는 부분인데, 이 부분을 CUDA로 구현했고 이는 detectron2 내 코드에서 확인 할 수 있습니다.

Detectron2를 이용해 학습을 하게 되면, 우리가 일반적으로 딥러닝 모델을 짤 때 구현하던 training loop를 짜지 않고 engine을 이용합니다. engine 사용은 학습 과정을 추상화 할 수 있어, 연구자 혹은 개발자는 모델 개발 자체에만 집중하게 됩니다. 즉, 모델을 구축하고 모델에 입력과 출력을 어떻게 할지만 정의하면 나머지 학습하는 부분은 detectron2에서 해줍니다.

Detectron2에서는 Instance Segmentaion, Panoptic Segmentation, DensePose, Person Keypoint Detection 등의 다양한 backbone을 통해 미리 학습된 객체 인식 모델을 제공합니다.

이러한 Detectron2 장점을 이용하여 Cloud Natvie 환경에서 비전 기반의 영상 이미지에 대해 객체 인신과 분류하기 위한 Inferencing Service 할 수 있는 방법에 대해 소개합니다.



## Object Detection & Segmentation

![](https://cdn.substack.com/image/fetch/w\_1456,c\_limit,f\_auto,q\_auto:good,fl\_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F4fe1ccf8-5f2f-4466-9930-7ad2f86d4c4c\_1418x1004.png)

### Object Detection

Object Detection은 컴퓨터 비전 기술의 세부 분야중 하나로써 주어진 영상 이미지 안에 있는 여러 객체들 (사람, 자차, 나무 등..)을 인식하는 기술입니다. 하단 이미지처럼 좌측에 있는 강아지 사진을 강아지로 판별하였다면 해당 모델은 이미지 분류 모델입니다. 하지만 오른쪽 사진처럼 강아지의 위치(Localization)를 탐지하고, 강아지로 분류 한다면 해당 모델은 객체 탐지 모델(Object Detection) 입니다. 따라서 객체 탐지 모델은 분류(Classification) 뿐만 아니라 위치(Localization) 라는 개념까지 포함하고 있습니다. 여기서 말하는 Localization은 객체라고 판단되는 곳에 BBOX (Bounding Box) 글 그려주는 작업입니다.

![Object Detection (Classification & Localization)](https://github.com/Pseudo-Lab/Tutorial-Book/blob/master/book/pics/OD-ch1img01.JPG?raw=true)

### Segmentaion

![](https://miro.medium.com/max/2160/1\*TcPH-XRIlyoB63CqiTCkLw.png)

Sementic Sementaion은 컴퓨터 비전 분야에서 가장 핵심적인 분야중에 하나입니다. 단순히 사진 이미지를 보고 객체를 탐지하고, 분류하는 것에 그치지 않고 해당 장면을 완벽히 이해애햐 하는 고도 수준의 문제입니다. Segmentaion의 사전적 의미를 살펴보 분할한다는 뜻인데 컴퓨터 비전에서는 이미지나 동영상을 분석해서 여러개의 필셀 집합으로 나누는 작업을 의미합니다. 이 픽셀 집합을 Segment 라고 부르는데 이것은 객체가 될 수도 있고, 배경이 될 수도 있습니다. 이미지 분석은 미리 정해진 클래스의 집합을 이용하여 주어진 이미지의 각 픽셀을 특정 클래스로 분류하는 작업입니다. 위 이미지와 같이 사람, 나무, 하늘, 자동차, 도로, 신호등 등의 클래스로 분류 합니다.&#x20;

Semantic Segmentation의 정확한 의미와 목적을 이해할 필요가 있는데 컴퓨터 비전에서 그 동안 고민했던 문제들을 살펴보면 다음과 같은 것들이 있습니다.

* Classification (분류): 입력된 이미지에 대해 하나의 레이블을 예측하는 작업
* Localization / Detection (탐지): 이미지 안의 객체를 탐지하고, BBOX를 통해 물체의 위치정보 제공
* Segmentaion (분할): 모든 픽셀의 레이블 예측

결국 Sementic Segmentaion은 이미지에 있는 모든 픽셀을 미리 지정된 class 개수에 따라 분류하는 것이 최종 목적입니다. 아래 그림은 전체 이미지에서 W\*H\*3 크기의 이미지 픽셀을 분석하여 전체 픽셀의 예측 클래스를 담고 있는 W\*H 배열을 생성하는 과정 입니다. 이 과정을 통해 각 픽셀을 지정된 클래스 (Person, Pure, Plants, Sidewalk, Building)로 분류할 수 있습니다.

![](https://i.imgur.com/Sp5l9P9.png)

여기서 주의할 점은 Segmentaion 과정에서 동일 클래스 내의 Instance(eg: 사람)를 구별하지 않습니다. 에를 들면 사진에서 사은 빨간색으로 분류했는데 사람 이라는 클래스 내에서 남자, 여자, 어린이와 같이 디테일하게 분류하지 않고, 사람 클래스 하나로 분류합니다. 즉 픽셀 자체가 어느 클래스에 속하는지만 분류합니다. 이와 다르게 동일한 클래스 내에서도 Instance 를 구별하는 모델을 Instance Segmentaion 라고 합니다.

### Instance Segmentaion

![](https://blog.kakaocdn.net/dn/CRvWU/btqSsQypZlk/gAMakLhRAykcULSIsSaP60/img.png)

Instacne Segmentaion은 이미지 내에 존재하는 모든 객체를 탐지하고, 동시에 동일 클래스 내의 인스턴스를 정확하게 픽셀 단위로 분류합니다. 위에서 소개한 Sementic Segmenation 이 동일한 클래스 내의 인스턴스를 분류하지 않는다는 점에서 차이가 있습니다. 즉 객체를 탐지하는 Object Detection과 픽셀 단위로 클래스를 분류하는 Sementic Segmentation이 결합된 형태입니다. Instance Segmentation는 [Mask R-CNN](https://paperswithcode.com/paper/mask-r-cnn)를 사용하였습니다. 따라서 Mssk R-CNN에 대한 간단한 이해가 필요합니다.

#### Mask R-CNN

![](https://blog.kakaocdn.net/dn/cR4qxV/btqW8GKjzrf/l9ZJ1d1Aa7Pk8zKMpD2lTk/img.jpg)



이미지가 주어졌을때 객체를 찾아내기 위해서 Faster R-CNN을 통해 얻는 RoI(Region of Interest) 이라고 불리는 영역을 예측하게 됩니다. 그래서 Region의 앞글자를 따서 R-CNN (Regions-Convolution Neuron Networks)라고 부릅니다. 이 영역을 예측하는 일과 동시에 양질의 마스크(Segmentation Mask)를 생성해서 객체를 효과적으로 찾아내는 과정이 이 논문의 목표입니다.

* **Segmentaion Mask 생성 (Classification & Localization (Pixel))**

Mask R-CNN은 이미지 내에서 각 instance에 대한 segmentation mask를 생성한다. (Classification + Localization) 여기서 Mask의 의미는 Object Detection 의 BBOX 가 Pixel 수준으로 정교하게 분할되었다고 이해하면 됩니다. 따라서 BBOX 와 같은 사각형 형태의 모양이 아닌 해당 객체를 정교하게 Pixel 단위로 분할하여 구분할 수 있습니다.

* **Mask R-CNN = Faster R-CNN에 mask branch**

Mask R-CNN는 Faster R-CNN에 mask branch를 더한 것입니다. mask branch는 object의 mask를 예측하는 branch이다. Mask R-CNN은 5fps의 속도이며 human pose estimate에서도 사용된다. COCO 2016 challenge에서 1등을 차지했다고 한다.

**Mask R-CNN**은 Faster R-CNN의 RPN에서 얻은 RoI(Region of Interest)에 대하여 객체의 class를 예측하는 classification branch, bbox regression을 수행하는 bbox regression branch와 평행으로 segmentation mask를 예측하는 **mask branch**를 추가한 구조를 가지고 있습니다. mask branch는 각각의 RoI에 작은 크기의 FCN(Fully Convolutional Network)가 추가된 형태입니다. segmentation task를 보다 효과적으로 수행하기 위해 논문에서는 객체의 spatial location을 보존하는 **RoIAlign** layer를 추가했습니다.&#x20;

### Person Keypoint Detection

Person keypoint detection은 사람의 관절을 keypoint로써 어떻게 구성되어 있는지 위치를 측정하고 추정하는 것입니다. 위에서 언급한 Mask R-CNN 논문에 소개된 내용으로, keypoint의 위치를 one-hot mask로 모델링하고, Mask R-CNN을 사용하여 각각 keypoint에 대해 하나씩 예측합니다.



### Panoptic Segmentaion

Panoptic segmentation은 Instance segmentation과 Semantic segmentation을 통합하여 stuff와 things class를 처리합니다. 여기서 stuff는 하늘, 도로 같은 비슷한 질감이나 텍스쳐 같은 형태가 불분명한 영역이고, things는 사람, 동물 같은 셀 수 있는 객체입니다. 다음의 그림을 보시면 쉽게 이해하실 수 있습니다.

여기서 사용된 추론 모델은 FAIR의 논문 [Panoptic FPN](https://arxiv.org/pdf/1901.02446.pdf)에서 소개되었습니다. panoptic segmentation을 수행할 때, 이전의 방식에서는 instance segmentation과 semantic segmentation을 별도의 네트워크로 사용하였습니다. 이 논문에서는 architectural level에서 이 메소드들을 하나의 방법으로 통합하기 위해, FPN을 backbone으로 하는 semantic segmentation을 이용해, instance segmentation(Mask R-CNN)을 수행합니다.

### DensePose

[DensePose](https://paperswithcode.com/paper/densepose-dense-human-pose-estimation-in-the)는 dense human pose estimation의 일종으로, RGB 이미지의 모든 사람의 픽셀 정보를 사람 신체의 3D 표면으로 매핑하는 것입니다. Input image의 사람의 신체에 3D로 매핑하는 것입니다. 여기서 제공하는 모델은 5만 개의 image를 surface로 annotate한 COCO Dataset을 DensePose-RCNN으로 학습한 모델입니다.


