# pytorch 설치



## 파이썬 설치

It is recommended that you use Python 3.7 or greater, which can be installed either through the Anaconda package manager (see [below](https://pytorch.org/get-started/locally/#anaconda)), [Homebrew](https://brew.sh), or the [Python website](https://www.python.org/downloads/mac-osx/).



## 아나콘다 설치

## Pytorch MacOS 설치

pytorch 는 멀티 OS (Linux, Window, Mac) 환경에서 설치할 수 있습니다. [pytorch](https://pytorch.org) 공식 홈페이지에서 제공하는 [Install Guide](https://pytorch.org/get-started/locally/) 통해서 설치를 시작합니다.

Detectron2 의 경우 pytorch >= 1.8 이상을 지원하기 때문에 LTS(1.8.2) 사용이 가능합니다. 여기서는 가장 최신 안정 버전인 Stable (1.10.1) 버전을 선택합니다. 그 외에 필요한 환경은 각 컴퓨팅 환경과 파이선 패키지의 기호에 맞게 선택하여 설치를 진행합니다.&#x20;

* Pytorch Build: Stable (1.10.1)
* OS: Mac
* Package:&#x20;
* Language: Python
* Compute Platform: CPU

![](../../.gitbook/assets/pytorch\_support\_program.png)

위와 같이 각자 개발환경에 맞는 설정을 선택할 경우 conda 패키지 도구를 이용하여 pytorch 설치할 수 있는 방법을 "Run this Command"에 가이드 합니다. 다음은 conda 명령어를 통해 pytorch 를 설치하는 과정입니다.

```
conda install pytorch torchvision torchaudio -c pytorch
```

설치 진행과정에서 pytorch torchvision torchaudio 각 패키지 설치여부를 묻고, yes 입력하여 설치를 진행하면 다음과 같이 설치가 성공적으로 끝나게 됩니다.

```
Collecting package metadata (current_repodata.json): done
Solving environment: done

## Package Plan ##

  environment location: /Users/sysmoon/opt/anaconda3

  added / updated specs:
    - pytorch
    - torchaudio
    - torchvision


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    conda-4.11.0               |   py38hecd8cb5_0        14.4 MB
    ninja-1.10.2               |       hf7b0b51_1         106 KB
    pytorch-1.7.1              |          py3.8_0        64.5 MB  pytorch
    torchaudio-0.7.2           |             py38         4.0 MB  pytorch
    torchvision-0.8.2          |         py38_cpu         6.5 MB  pytorch
    ------------------------------------------------------------
                                           Total:        89.5 MB

The following NEW packages will be INSTALLED:

  ninja              pkgs/main/osx-64::ninja-1.10.2-hf7b0b51_1
  pytorch            pytorch/osx-64::pytorch-1.7.1-py3.8_0
  torchaudio         pytorch/osx-64::torchaudio-0.7.2-py38
  torchvision        pytorch/osx-64::torchvision-0.8.2-py38_cpu

The following packages will be UPDATED:

  conda                               4.10.1-py38hecd8cb5_1 --> 4.11.0-py38hecd8cb5_0


Proceed ([y]/n)? ㅛy


Downloading and Extracting Packages
conda-4.11.0         | 14.4 MB   | ##################################################################################################################################################################################################################################### | 100%
torchvision-0.8.2    | 6.5 MB    | ##################################################################################################################################################################################################################################### | 100%
ninja-1.10.2         | 106 KB    | ##################################################################################################################################################################################################################################### | 100%
pytorch-1.7.1        | 64.5 MB   | ##################################################################################################################################################################################################################################### | 100%
torchaudio-0.7.2     | 4.0 MB    | ##################################################################################################################################################################################################################################### | 100%
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

이후 다음과 같이 파이썬 콜솔 창에서 pytorch 라브러리를 import 하고, 5\*3 Random Matrix 를 생성하여 설치 동작여부를 확인할 수 있습니다.

```
python                                                                                                                                                                                                             [14:55:05]
Python 3.8.8 (default, Apr 13 2021, 12:59:45)
[Clang 10.0.0 ] :: Anaconda, Inc. on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
x = torch.rand(5,3)
print(x)
>>> x = torch.rand(5,3)
>>> print(x)
tensor([[0.9369, 0.2089, 0.2880],
        [0.8379, 0.9975, 0.4953],
        [0.6017, 0.6604, 0.8723],
        [0.2103, 0.7909, 0.7647],
        [0.5019, 0.1360, 0.3487]])
>>>
```
