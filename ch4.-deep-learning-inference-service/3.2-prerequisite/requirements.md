# Requirements

## 사전준비 사항

Detectron2 설치에 필요한 [사전준비](https://detectron2.readthedocs.io/en/latest/tutorials/install.html#install-pre-built-detectron2-linux-only) 요구사항은 다음과 같습니다.

* Linux or macOS with Python ≥ 3.6
* PyTorch ≥ 1.8 and [torchvision](https://github.com/pytorch/vision/) that matches the PyTorch installation. Install them together at [pytorch.org](https://pytorch.org) to make sure of this
* OpenCV is optional but needed by demo and visualization





## 파이썬 설치 (Anaconda)

### MacOS 설치

blabla

### 우분투 설치

install the PyTorch binaries, you will need to use one of two supported package managers: [Anaconda](https://www.anaconda.com/download/#linux) or [pip](https://pypi.org/project/pip/). Anaconda is the recommended package manager as it will provide you all of the PyTorch dependencies in one, sandboxed install, including Python.

#### pip



python3.8-dev 설치

```
sudo apt-get install python3.8-dev
```

**Anaconda**

To install Anaconda, you will use the [command-line installer](https://www.anaconda.com/download/#linux). Right-click on the 64-bit installer link, select `Copy Link Location`, and then use the following commands:

```
# The version of Anaconda may be different depending on when you are installing`
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
# and follow the prompts. The defaults are generally good.`
```

> You may have to open a new terminal or re-source your `~/.bashrc` to get access to the `conda` command.

## Pytorch 설치

### MacOS 설치

pytorch 는 멀티 OS (Linux, Window, Mac) 환경에서 설치할 수 있습니다. [pytorch](https://pytorch.org) 공식 홈페이지에서 제공하는 [Install Guide](https://pytorch.org/get-started/locally/) 통해서 설치를 시작합니다.

Detectron2 의 경우 pytorch >= 1.8 이상을 지원하기 때문에 LTS(1.8.2) 사용이 가능합니다. 여기서는 가장 최신 안정 버전인 Stable (1.10.1) 버전을 선택합니다. 그 외에 필요한 환경은 각 컴퓨팅 환경과 파이선 패키지의 기호에 맞게 선택하여 설치를 진행합니다.

* Pytorch Build: Stable (1.10.1)
* OS: Mac
* Package:
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


Proceed ([y]/n)? y


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

### Pytorch Ubuntu 설치

![](../../.gitbook/assets/pytorch\_support\_ubuntu2.png)

pip 패키지 도구를 이용하여 pytroch 를 설치합니다.

```
pip3 install torch==1.8.2+cu111 torchvision==0.9.2+cu111 torchaudio==0.8.2 -f https://download.pytorch.org/whl/lts/1.8/torch_lts.html
```



## OpenCV 설치

```
pip install opencv-python
```

* [pytroch 개발환경 설정](https://blog.daum.net/geoscience/1565)
* [pip issue](https://river.me/blog/change-pip-version/)
