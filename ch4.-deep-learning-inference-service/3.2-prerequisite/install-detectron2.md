# Install Detectron2

## 사전 요구사항

Detecton2 설치를 위해 필요한 사전요구사항은 다음과 같습니다.

* Linux or macOS with Python ≥ 3.6
* PyTorch ≥ 1.8 and [torchvision](https://github.com/pytorch/vision/) that matches the PyTorch installation. Install them together at [pytorch.org](https://pytorch.org) to make sure of this
* OpenCV is optional but needed by demo and visualization

## 소스 빌드 설치방법

Detectron2 Github 을 통해 소스를 클론하고, 아래와 같이 빌드 작업을 수행합니다.

gcc & g++ ≥ 5.4 are required. [ninja](https://ninja-build.org) is optional but recommended for faster build. After having them, run:

#### ninja install

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 2> /dev/null
brew install ninja
```

#### install detectron2

```
python -m pip install 'git+https://github.com/facebookresearch/detectron2.git'
# (add --user if you don't have permission)

# Or, to install it from a local clone:
git clone https://github.com/facebookresearch/detectron2.git
python -m pip install -e detectron2

# On macOS, you may need to prepend the above commands with a few environment variables:
CC=clang CXX=clang++ ARCHFLAGS="-arch x86_64" python -m pip install ...
```

To **rebuild** detectron2 that’s built from a local clone, use `rm -rf build/ **/*.so` to clean the old build first. You often need to rebuild detectron2 after reinstalling PyTorch.\


{% hint style="info" %}
python setuptools==59.5.0 설치해야 train\_net.py 빌드과정에서 에러 발생하지 않음

python -m pip install setuptools==59.5.0
{% endhint %}



## Pre-Built 설치방법 (리눅스만 지원)

![](<../../.gitbook/assets/detectron2\_support\_table (1).png>)

사전 요구사항을 기준으로 구축한 Pytorch, CUDA 버전을 기반으로 Pre-Built 설치를 위해 필요한 조합을 선택합니다. 현재 테트 환경은 Pytorch Stable (1.10.1), CUDA (11.3) 버전이므로 위 테이블 조합중에 torch(1.10), CUDA(11.3) 을 선택하여 install 버튼을 설치 방법을 확인합니다.

```
pip3 install detectron2 -f \
  https://dl.fbaipublicfiles.com/detectron2/wheels/cu113/torch1.10/index.html
```

{% hint style="info" %}
1. The pre-built packages have to be used with corresponding version of CUDA and the official package of PyTorch. Otherwise, please build detectron2 from source.
2. New packages are released every few months. Therefore, packages may not contain latest features in the main branch and may not be compatible with the main branch of a research project that uses detectron2 (e.g. those in [projects](https://github.com/facebookresearch/detectron2/blob/main/projects)).
{% endhint %}

## Ref

* [https://detectron2.readthedocs.io/en/latest/tutorials/install.html#install-pre-built-detectron2-linux-only](https://detectron2.readthedocs.io/en/latest/tutorials/install.html#install-pre-built-detectron2-linux-only)
* [install detectron on mac cpu mode](https://knowing.net/posts/2021/11/install-detectron2-draft/)
