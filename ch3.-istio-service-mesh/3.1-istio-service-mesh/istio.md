# Istio 설치



#### Istio 다운로드 및 설치환경 구성

***

Istio 의 Service Mesh를 위한 다양한 기능을 테스트 하기 위해서는 기본적으로 Kubernetes, Kubectl, Helm와 같은 다양한 개발, 테스트 환경이 필요합니다.

Kubernetes는 **sidecar-deployment** 와 같은 배포 개념을 기본으로하는 Istio를 적용하기에 최적의 공간입니다. Kubernetes 를 어떠한 방식으로 설치하고 실행할지는 현재 보유하고 있는 인프라 자원의 상황에 따라 유연성을 가지고 선택할 수 있습니다. 개발자가 가지고 있는 노트북에서부터 퍼블릭 클라우드 사업자가 제공하는 Managed Service (AWS(EKS), Azure(AKS), GCP(Kubernetes Engine) 를 이용할 수도 있고, 베어메탈 서버에 나만의 Kubernetes 클러스터 환경을 구축할 수 있습니다.

이번 장에서는 Kubernetes 클러스터 테스트/개발 환경 구축을 위해 [Minikube](https://kubernetes.io/ko/docs/tasks/xtools/install-minikube/) 와 [Kubectl](https://github.com/istiokrsg/istio\_book\_kr/blob/master/chapter-2/2.1.1.minikube-feature) 설치 방법과 Kubernetes 클러스터에 마이크로서비스 예제 애플리케이션 Bookinfo를 배포하여 Istio를 통해 서비스 간 트래픽 컨트롤, 서비스 제어, 보안, 정책, 모니터링 등 주요 기능을 테스트하기 위한 환경 구축 방벙에 대해 자세히 소개하겠습니다.

**1. Istio 최신버전 다운로드**

istio 설치파일과, 샘플 예제 파일 그리고 istioctl 명령어 바이너리 파일이 포함된 최신 릴리즈 버전을 다운로드 받기 위해 [Istio release](https://github.com/istio/istio/releases/tag/1.7.3) (istio 1.7.3) 페이지에 접속합니다. 다양한 운영체제에서 지원하는 Istio 바이너리 파일을 확인할 수 있고, 사용자의 테스트 환경에 맞는 Istio 버전을 다운로드 받습니다.

만약 사용자 환경이 OSX 또는 Linux 환경일 경우 아래 curl 명령어를 통해 원하는 Istio 릴리즈 버전을 다운로드 받을 수 있습니다.

```markdown
curl -L <https://istio.io/downloadIstio> | ISTIO_VERSION=1.7.3 TARGET_ARCH=x86_64 sh -
```

이 책은 Istio 1.7.3 버전 환경에서 테스트를 진행할 예정입니다. 위 curl 스크립트를 통해 설치하면 istio-1.7.3 폴더가 자동으로 생성된 것을 확인할 수 있습니다. istio-1.7.3 디렉토리 구조를 아래와 같이 확인할 수 있습니다.

```markdown
$ tree -d -L 2 istio-1.7.3
istio-1.7.3
├── bin
├── manifests
│   ├── charts
│   ├── deploy
│   ├── examples
│   └── profiles
├── samples
│   ├── addons
│   ├── bookinfo
│   ├── certs
│   ├── cross-network-gateway
│   ├── custom-bootstrap
│   ├── external
│   ├── fortio
│   ├── health-check
│   ├── helloworld
│   ├── httpbin
│   ├── https
│   ├── kubernetes-blog
│   ├── operator
│   ├── rawvm
│   ├── security
│   ├── sleep
│   ├── tcp-echo
│   └── websockets
└── tools
    └── certs`
```

각 디렉토리 구조는 다음과 같은 정보들을 포함하고 있습니다.

* /bin Istio를 제어하기 위해 사용되는 istioctl 바이너리 파일로 구성되어 있습니다.
* /samples istio 샘플 애플리케이션들을 포함하고 있으며, Istio 주요 기능을 확인하기 위해 사용자가 배포하여 테스트 가능합니다. 이 책에서는 bookinfo MSA 를 배포하여 istio 주요 기능을 소개할 예정입니다.
* manifests 쿠버네티스에 istio 를 배포하기 위한 helm chart 와 manifest 파일로 구성되어 있습니다.

**2. 환경변수 PATH 등록**

istioctl 명령어를 어느 위치에서든 실행가능하도록 환경변수에 등록합니다.

```markdown
$ cd istio-1.7.3
$ export PATH=$PWD/bin:$PATH
```

**4. auto-completion 옵션 적용**

istioctl 명령어에 대한 자동 완성 기능을 위해 bash 또는 zsh에 이 기능을 활성화 할 수 있습니다.

**ZSH auto-completion**

ZHS 사용자의 경우 istio auto-completion 설정을 위한 파일은 istio-1.7.3/tools/\_istioctl 파일 입니다. vi 편집기를 이용하여 \~/.zshrc 파일을 열고, 맨 하단에 아래와 같이 source 명령어를 이용하여 \_istioctl 파일의 위치를 설정하여 저장하여 나옵니다.

```markdown
# .zshrc 파일 편집
vim ~/.zshrc

# .zshrc 파일 하단에 아래 내용 추가
source ~/istio-1.4.5/tools/_istioctl
```

이후 위 설정을 zsh에 반영하기 위해 source 명령어를 이용하여 \~/.zshrc 파일을 활성화 합니다.

```markdown
source ~/.zshrc
```

zsh 환경에서 istioctl 명령어 입력한후, tab 키를 클릭하면 아래 그림과 같이 istioctl 명령어에서 사용 가능한 다양한 sub command(ex: authm, dashboard 등..) 명령어 확인이 가능합니다.

```bash
(⎈ |prl-kc-k8s-istiobooks:default) sysmoon  ~/workspace/istiobooks/02.setup_install/install_istio/istio-1.7.3/tools  istioctl version
analyze          convert-ingress  deregister       install          manifest         profile          proxy-status     upgrade          verify-install
authz            dashboard        experimental     kube-inject      operator         proxy-config     register         validate         version
```

**BASH auto-completion**

BASH 사용자의 경우 ZSH 사용자와 설치 방법은 동일하며, 단지 \~/.bashrc 파일을 오픈 편집하여 istio auto-completion 설정을 위해 istio-1.4.5/tools/istioctl.bash 파일을 적용하면 됩니다.

```bash
# .bashrc 파일 편집
vim ~/.bashrc

# .bashrc 파일 하단에 아래 내용 추가
source ~/istio-1.4.5/tools/istioctl.bash
```

이후 위 설정을 bash에 반영하기 위해 source 명령어를 이용하여 \~/.bashrc 파일을 활성화 합니다.

```bash
source ~/.bashrc
```

이후 bash shell 환경에서 istioctl 명령어를 위한 auto-completion 기능을 tab 키를 통해 확인할 수 있습니다.

#### Istio 설치

***

istio 최신 버전을 다운로드하여 istioctl 명령어를 실행하기 위한 환경설정을 모두 마무리 했습니다. 이제 Istio를 Kubernetes에 설치하는 방법을 소개하겠습니다.

istioctl cli 명령 도구를 이용해서 istio 의 control plane 과 data plane 의 sidecar 에 대한 풍부한 사용자 정의를 제공합니다. 또한 설치 과정에서 오류를 방지하고, 설정 과정에서의 다양한 유효성 검증 기능을 제공합니다.

istio의 이러한 설치 지침을 사용하여 istio 에서 제공하는 내장 profile 중 하나를 선택하여 특정 요구에 맞게 구성하여 설치하고, 이후 필요에 따라 사용자 정의에 따라 확장 설치 가능합니다.

[Istio Profile](https://www.notion.so/48f448adc5bf4be483eb74722704a161)

istio 는 크게 5가지의 [프로파일](https://istio.io/v1.7/docs/setup/additional-setup/config-profiles/)을 제공하고 있습니다. 아래 표와 같이 각 프로파일별 Core component 와 Addon 기본 설치 여부를 표시하고 있습니다. (X마크 = 해당 프로파일 설치된 항목)

* **default** 상용서비스 환경에서 추천되며, IstioOperator API의 기본 설정에 따라 구성 요소를 활성화 합니다.
* **demo** istio 실행에 필요한 리소스를 최소화하고, 전체적인 기능을 보여주도록 설계된 프로파일 입니다. Istio MSA 아키텍처 구조의 예제인 BookInfo 응용 프로그램을 실행하고, istio의 주요 기능을 테스트 해보는데 적합합니다. 사용자의 빠른 시작을 위해 대부분의 istio 의 Core component와 Addons이 설치되고, 이러한 추가 기능을 사용하도록 사용자 정의하여 구성할 수 있습니다.
* **minimal** istio의 traffic management 기능을 사용하기 위한 최소한의 컴포넌트만을 포함하고 있습니다.
* **remote** 멀티 쿠버네티스 클러스터를 shared control plane 구성을 통해 Mesh 서비스하기 위한 원격 클러스터를 구성하는데 사용합니다.
* **empty** empty 프로파일은 아무것도 배포하지 않습니다. 따라서 base profile로써 사용자 필요에 의한 구성을 하나씩 구축할 때 유용하게 활용할 수 있습니다.

위 5개의 프로파일 중, default 프로파일을 이용하여 istio을 쉽게 설치할 수 있습니다. defulat 프로파일은 상용 서비스 환경에서 사용자 정의에 맞게 필요한 컴포넌트를 확장하기 위해 좋은 출발점 입니다. 하지만 이 책에서는 MSA 아키텍처 기반의 BookInfo 애플리케이션 설치하여 Istio 의 주요 기능을 테스트하기 위해 demo 프로파일을 이용하여 설치하겠습니다.

멀티 쿠버네티스를 사용하고 있을 경우, 현재 활성화된 쿠버네티스 컨텍스트를 확인한 후, 해당 클러스터에 istio 설치해야 합니다. 위 경우 drl-kc-k8s-aks, minikbe 2개의 쿠버네티스 클러스터를 사용하고 있고, 현재 활성화된 클러스터는 minikube 임을 확인할 수 있습니다.

```bash
kubectl config get-context
CURRENT   NAME             CLUSTER          AUTHINFO                                   NAMESPACE
          drl-kc-k8s-aks   drl-kc-k8s-aks   clusterUser_drl-kc-aks-rg_drl-kc-k8s-aks
*         minikube         minikube         minikube
```

minikube 쿠버네티스 클러스터에 istio 의 demo profile을 이용하여 설치합니다.

```bash
$ cd istio-1.7.3

$ istioctl manifest apply --set profile=demo
- Applying manifest for component Base...
✔ Finished applying manifest for component Base.
- Applying manifest for component Pilot...
✔ Finished applying manifest for component Pilot.
  Waiting for resources to become ready...
  Waiting for resources to become ready...
  Waiting for resources to become ready...
- Applying manifest for component EgressGateways...
- Applying manifest for component IngressGateways...
- Applying manifest for component AddonComponents...
✔ Finished applying manifest for component EgressGateways.
✔ Finished applying manifest for component IngressGateways.
✔ Finished applying manifest for component AddonComponents.
```

**Istio Injection 설정**

default 네임스페이스에 배포되는 모든 Pod 에 Envoy Sidecar Proxy 를 자동으로 주입하기 위해 다음과 같이 istio-injection 을 라벨링 합니다.

```bash
kubectl label namespace default istio-injection=enabled
```

default 네임스페이스에 istio-injectio=enabled 라벨링 적용 여부를 확인할 수 있습니다.

```bash
kubectl describe ns default
Name:         default
Labels:       istio-injection=enabled
Annotations:  <none>
Status:       Active

No resource quota.

No resource limits.`
```
