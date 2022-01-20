# Table of contents

* [What is MyProduct?](README.md)

## Ch1. Introduction

* [1.1 Cloud Native 기술소개](ch1.-introduction/1.1-cloud-native.md)
* [1.3 MSA 아키텍처 방법론 소개](ch1.-introduction/1.3-msa.md)

## Ch2. Kubernetes

* [2.1 Kubernetes 소개](ch2.-kubernetes/2.1-kubernetes/README.md)
  * [2.1.1 컨테이너 소개](ch2.-kubernetes/2.1-kubernetes/2.1.1.md)
  * [쿠버네티스 소개](ch2.-kubernetes/2.1-kubernetes/undefined.md)
  * [쿠버네티스 구조](ch2.-kubernetes/2.1-kubernetes/undefined-1.md)
  * [레이블과 셀렉터 & 어노테이션](ch2.-kubernetes/2.1-kubernetes/and.md)
  * [인증과 권한](ch2.-kubernetes/2.1-kubernetes/undefined-2.md)
  * [Secret](ch2.-kubernetes/2.1-kubernetes/secret.md)
  * [Config Map](ch2.-kubernetes/2.1-kubernetes/config-map.md)
  * [오브젝트와 컨트롤러](ch2.-kubernetes/2.1-kubernetes/undefined-3.md)
* [2.2 쿠버네티스 구축](ch2.-kubernetes/2.2/README.md)
  * [2.2.1 On-Premise 환경 Kubernetes 구축](ch2.-kubernetes/2.2/2.2.1-on-premise-kubernetes.md)
  * [2.1.2 Azure Public Cloud 환경  쿠버네티스(AKS) 구축](ch2.-kubernetes/2.2/2.1.2-azure-public-cloud-aks.md)
* [2.3 Container Registry (ACR)](ch2.-kubernetes/2.3-container-registry-acr.md)
* [2.4 Ceph 클러스터 구축](ch2.-kubernetes/2.4-ceph/README.md)
  * [Rook 오케스트레이션을 이용한 Ceph 구축](ch2.-kubernetes/2.4-ceph/rook-ceph.md)

## Ch3. Istio Service Mesh

* [3.1 Istio 설치](ch3.-istio-service-mesh/3.1-istio/README.md)
  * [Bookinfo MSA 샘플 애플리케이션](ch3.-istio-service-mesh/3.1-istio/bookinfo-msa.md)
  * [2.2.4 Ingress HTTPS 인증서 적용](ch3.-istio-service-mesh/3.1-istio/2.2.4-ingress-https.md)
* [3.2 Istio Traffic Management](ch3.-istio-service-mesh/3.2-istio-traffic-management/README.md)
  * [2.2.3 Ingress Gateway](ch3.-istio-service-mesh/3.2-istio-traffic-management/2.2.3-ingress-gateway.md)
  * [2.2.1 Request Routing](ch3.-istio-service-mesh/3.2-istio-traffic-management/2.2.1-request-routing.md)
* [3.2 Observability](ch3.-istio-service-mesh/3.2-observability/README.md)
  * [Service Mesh 모니터링 (Kiali)](ch3.-istio-service-mesh/3.2-observability/service-mesh-kiali.md)
  * [Open Tracing (Yeager)](ch3.-istio-service-mesh/3.2-observability/open-tracing-yeager.md)
  * [Distributed Tracing](ch3.-istio-service-mesh/3.2-observability/distributed-tracing.md)

## Ch4. Deep Learning Inference Service

* [3.1 Detectron2 소개](ch4.-deep-learning-inference-service/3.1-detectron2/README.md)
  * [Figma Integration](ch4.-deep-learning-inference-service/3.1-detectron2/figma-integration.md)
* [3.2 Triton 이용한 Inference 서비스](ch4.-deep-learning-inference-service/3.2-triton-inference/README.md)
  * [GitHub Integration](ch4.-deep-learning-inference-service/3.2-triton-inference/github-integration.md)
* [3.3 AML(Azure Machine Learning) Service](ch4.-deep-learning-inference-service/3.3-aml-azure-machine-learning-service/README.md)
  * [Azure ARC enabled Kubernetes](ch4.-deep-learning-inference-service/3.3-aml-azure-machine-learning-service/azure-arc-enabled-kubernetes.md)
  * [Azure ML Endpoint](ch4.-deep-learning-inference-service/3.3-aml-azure-machine-learning-service/azure-ml-endpoint.md)
* [3.4 모델 배포](ch4.-deep-learning-inference-service/3.4.md)
* [3.5 gRPC 이용한 추론 서비스 구축](ch4.-deep-learning-inference-service/3.5-grpc/README.md)
  * [gRPC 프로토콜 소개](ch4.-deep-learning-inference-service/3.5-grpc/grpc.md)
  * [gRPC 위한 Ingress Gateway](ch4.-deep-learning-inference-service/3.5-grpc/grpc-ingress-gateway.md)
  * [RESTful API 지원을 위한 gRPC Gateway 구축](ch4.-deep-learning-inference-service/3.5-grpc/restful-api-grpc-gateway.md)
  * [gRPC Gateway 샘플 애플리케이션 구축](ch4.-deep-learning-inference-service/3.5-grpc/grpc-gateway.md)

## CH4.Deploy

* [Github Action](ch4.deploy/github-action/README.md)
  * [Kustomize 패키징](ch4.deploy/github-action/kustomize.md)
* [CI/CD 파이프라인 구축](ch4.deploy/ci-cd/README.md)
  * [Page 1](ch4.deploy/ci-cd/page-1.md)
* [ArgoCD Gitops 패턴 적용](ch4.deploy/argocd-gitops.md)

## CH5.Monitoring

* [Prometheus & Grafana 모니터링 시스템](ch5.monitoring/prometheus-and-grafana.md)
* [Datadog 모니터링 시스템](ch5.monitoring/datadog.md)
