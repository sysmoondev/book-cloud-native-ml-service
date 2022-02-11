# Deployment with Tracing or Scripting

## Tracing&#x20;

[deployment tutorial](https://github.com/facebookresearch/detectron2/tree/main/tools/deploy)





#### Install CMake



pre install

```
sudo apt install libssl-dev
sudo apt install build-essentials
```

download cmake

```
wget https://github.com/Kitware/Cmake/releases/download/v3.19.6/cmake-3.19.6.tar.gz
```



압축해제

```
tar -xvf cmake-3.19.6.tar.gz
```



```
./bootstrap --prefix=/usr/local
make
sudo make install

```

version 확인

```
cmake --version


PATH=/usr/local/bin:$PATH:$HOME/bin
```
