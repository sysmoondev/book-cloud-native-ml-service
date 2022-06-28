---
description: https://github.com/istiokrsg/istio_book_kr/blob/master/chapter-2/kubernetes.md
---

# 쿠버네티스 컴포넌트

AWS(아마존 웹 서비스), GCP(구글 클라우드 플랫폼), Azure (Microsoft 클라우드 서비스) 등 대형 클라우드 제공업체들은 저마다의 방식으로 관리형 쿠버네티스 서비스를 제공하고 있습니다. AWS 는 EKS([Elastic Kubernetes Service](https://aws.amazon.com/ko/eks/)), GCP 는 GKE([Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine)), Azure 는 AKS([Azure Kubernetes Service](https://docs.microsoft.com/ko-kr/azure/aks/)) 라는 이름으로 제공되는 이 서비스들 중 하나를 사용하면 손쉽게 쿠버네티스를 활용하여 운영까지 적용할 수 있습니다.

그러나 실제로 운영에 적용하기 위해서는 아무리 쉬운 관리형 서비스를 사용한다 하더라도, 기본적인 내용을 알고 있어야 합니다.

쿠버네티스를 구성하고 있는 컴포넌트들은 다음과 같습니다.

![참조 : 공식문서 https://kubernetes.io/docs/concepts/overview/components/](<../../.gitbook/assets/image (1) (1).png>)



쿠버네티스 클러스터의 기본요소는 워크로드가 동작하는 워커노드들이 있어야 하고, 워크로드들을 관리하는 마스터노드가 기본 구성입니다. 마스터노드를 공식문서에서는 Control Plane 이라고 하고, 워커노드는 Node 라고 표현하고 있습니다.

단순하게 생각해서 만약 여러분이 직접 클러스터 하나를 구성한다면 마스터노드, 워커노드를 준비해서 각각 컴포넌트들을 설치해주면 됩니다.

그럼 어떤 컴포넌트들이 있는지 확인해봅시다. \[ref: [https://kubernetes.io/docs/concepts/overview/components/](https://kubernetes.io/docs/concepts/overview/components/)]

마스터노드

kube-apiserver

etcd

kube-scheduler

kube-controller-manater

cloud-controller-manater

워커노드

kubelet

kube-proxy

container runtime

애드온(Addons)

DNS

Web UI (Dashboard)

Container Resource Monitoring

CLuster-level Logging

