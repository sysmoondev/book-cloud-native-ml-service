# Table of contents

* [README](README.md)

## Ch1. Introduction

* [1.1 Cloud Native 기술소개](ch1.-introduction/1.1-cloud-native.md)
* [1.3 MSA 아키텍처 방법론 소개](ch1.-introduction/1.3-msa.md)

## Ch2. Kubernetes

* [2.1 쿠버네티스 소개](ch2.-kubernetes/2.1-kubernetes/README.md)
  * [2.1.1 쿠버네티스 탄생 배경](ch2.-kubernetes/2.1-kubernetes/2.1.1.md)
  * [2.1.2 쿠버네티스 구조](ch2.-kubernetes/2.1-kubernetes/undefined-1.md)
  * [2.1.3 쿠버네티스 네트워크](ch2.-kubernetes/2.1-kubernetes/2.1.3.md)
  * [2.1.1 컨테이너 소개](ch2.-kubernetes/2.1-kubernetes/2.1.1-1.md)
  * [쿠버네티스 소개](ch2.-kubernetes/2.1-kubernetes/undefined.md)
  * [레이블과 셀렉터 & 어노테이션](ch2.-kubernetes/2.1-kubernetes/and.md)
  * [인증과 권한](ch2.-kubernetes/2.1-kubernetes/undefined-2.md)
  * [Secret](ch2.-kubernetes/2.1-kubernetes/secret.md)
  * [Config Map](ch2.-kubernetes/2.1-kubernetes/config-map.md)
  * [오브젝트와 컨트롤러](ch2.-kubernetes/2.1-kubernetes/undefined-3.md)
* [2.2 쿠버네티스 시작하기](ch2.-kubernetes/2.2/README.md)
  * [2.2.1 다양한 환경에서 Kubernetes 구축하기](ch2.-kubernetes/2.2/2.2.1-on-premise-kubernetes.md)
  * [2.2.2 application 배포하기](ch2.-kubernetes/2.2/2.1.2-azure-public-cloud-aks.md)
* [2.3 쿠버네티스 활용하기](ch2.-kubernetes/2.3/README.md)
  * [2.3.1 helm vs kustomize](ch2.-kubernetes/2.3/2.3.1-helm-vs-kustomize.md)
  * [2.3.2 ingress controller](ch2.-kubernetes/2.3/2.3.2-ingress-controller.md)
  * [2.3.3 node pool 구분하기](ch2.-kubernetes/2.3/2.3.3-node-pool.md)
* [2.3 Container Registry (ACR)](ch2.-kubernetes/2.3-container-registry-acr.md)
* [2.4 Ceph 클러스터 구축](ch2.-kubernetes/2.4-ceph/README.md)
  * [Rook 오케스트레이션을 이용한 Ceph 구축](ch2.-kubernetes/2.4-ceph/rook-ceph.md)

## Ch3. Istio Service Mesh

* [3.1 서비스 메쉬](ch3.-istio-service-mesh/3.1-istio/README.md)
  * [3.1.1 ISTIO?](ch3.-istio-service-mesh/3.1-istio/3.1.1-istio.md)
  * [3.1.2 ISTIO 기능](ch3.-istio-service-mesh/3.1-istio/3.1.2-istio.md)
  * [3.1.3 Traffic management](ch3.-istio-service-mesh/3.1-istio/3.1.3-traffic-management.md)
  * [3.1.4 Security](ch3.-istio-service-mesh/3.1-istio/3.1.4-security.md)
  * [3.1.5 Observability](ch3.-istio-service-mesh/3.1-istio/3.1.5-observability.md)
* [3.2 Book Info 를 이용한 Traffic Management](ch3.-istio-service-mesh/3.2-istio-traffic-management/README.md)
  * [3.2.1 맛보기 BookInfo MSA 샘플 애플리케이션](ch3.-istio-service-mesh/3.2-istio-traffic-management/bookinfo-msa.md)
  * [3.2.2 Advance Traffic Management](ch3.-istio-service-mesh/3.2-istio-traffic-management/3.2.2-advance-traffic-management/README.md)
    * [3.2.2.1 Request Routing](ch3.-istio-service-mesh/3.2-istio-traffic-management/3.2.2-advance-traffic-management/3.2.2.1-request-routing.md)
    * [3.2.2.2 Fault Injection](ch3.-istio-service-mesh/3.2-istio-traffic-management/3.2.2-advance-traffic-management/3.2.2.2-fault-injection.md)
    * [3.2.2.3 Traffic Shifting](ch3.-istio-service-mesh/3.2-istio-traffic-management/3.2.2-advance-traffic-management/3.2.2.3-traffic-shifting.md)
    * [3.2.2.4 TCP Traffic shifting](ch3.-istio-service-mesh/3.2-istio-traffic-management/3.2.2-advance-traffic-management/3.2.2.4-tcp-traffic-shifting.md)
  * [2.2.4 Ingress HTTPS 인증서 적용](ch3.-istio-service-mesh/3.2-istio-traffic-management/2.2.4-ingress-https.md)
  * [2.2.3 Ingress Gateway](ch3.-istio-service-mesh/3.2-istio-traffic-management/2.2.3-ingress-gateway.md)
  * [2.2.1 Request Routing](ch3.-istio-service-mesh/3.2-istio-traffic-management/2.2.1-request-routing.md)
* [3.3 Book Info 를 이용한 Observability](ch3.-istio-service-mesh/3.2-observability/README.md)
  * [3.3.1 맛보기 BookInfo Observability](ch3.-istio-service-mesh/3.2-observability/3.3.1-bookinfo-observability.md)
  * [3.3.2 Advance Observability](ch3.-istio-service-mesh/3.2-observability/3.3.2-advance-observability/README.md)
    * [3.3.2.1 Metrics prometheus](ch3.-istio-service-mesh/3.2-observability/3.3.2-advance-observability/3.3.2.1-metrics-prometheus.md)
    * [3.3.2.2 Metrics grafana](ch3.-istio-service-mesh/3.2-observability/3.3.2-advance-observability/3.3.2.2-metrics-grafana.md)
    * [3.3.2.3 Envoy Access Logs](ch3.-istio-service-mesh/3.2-observability/3.3.2-advance-observability/3.3.2.3-envoy-access-logs.md)
    * [3.3.2.4 Distributed Tracing Jaeger](ch3.-istio-service-mesh/3.2-observability/3.3.2-advance-observability/3.3.2.4-distributed-tracing-jaeger.md)
    * [3.3.2.5 Visualizing Service Mesh](ch3.-istio-service-mesh/3.2-observability/3.3.2-advance-observability/3.3.2.5-visualizing-service-mesh.md)
  * [Service Mesh 모니터링 (Kiali)](ch3.-istio-service-mesh/3.2-observability/service-mesh-kiali.md)
  * [Open Tracing (Yeager)](ch3.-istio-service-mesh/3.2-observability/open-tracing-yeager.md)
  * [Distributed Tracing](ch3.-istio-service-mesh/3.2-observability/distributed-tracing.md)

## Ch4. Deep Learning Inference Service

* [3.1 Detectron2](ch4.-deep-learning-inference-service/3.1-detectron2.md)
  * [Introduction](ch4.-deep-learning-inference-service/3.1-detectron2/introduction.md)
  * [Installation](ch4.-deep-learning-inference-service/3.2-prerequisite/README.md)
    * [Requirements](ch4.-deep-learning-inference-service/3.2-prerequisite/requirements.md)
    * [Install Detectron2](ch4.-deep-learning-inference-service/3.2-prerequisite/install-detectron2.md)
  * [Configuration](ch4.-deep-learning-inference-service/3.1-detectron2/configuration.md)
  * [Getting Start](ch4.-deep-learning-inference-service/getting-start/README.md)
    * [Inference Demo with Pre-trained Models](ch4.-deep-learning-inference-service/getting-start/inference-demo-with-pre-trained-models.md)
    * [Training & Evaluation](ch4.-deep-learning-inference-service/getting-start/training-and-evaluation.md)
    * [Detectron2 API in Your Code](ch4.-deep-learning-inference-service/3.1-detectron2/getting-start/detectron2-api-in-your-code.md)
* [3.2 Nvidia TritonServer](ch4.-deep-learning-inference-service/3.2-triton-inference/README.md)
  * [Architecture](ch4.-deep-learning-inference-service/3.2-triton-inference/architecture.md)
  * [Quick Start](ch4.-deep-learning-inference-service/3.2-triton-inference/github-integration.md)
  * [모델의 자동 변환과 배포](ch4.-deep-learning-inference-service/3.2-triton-inference/undefined.md)
  * [모델 성능 최적화](ch4.-deep-learning-inference-service/3.2-triton-inference/undefined-1/README.md)
    * [Model Analyzer](ch4.-deep-learning-inference-service/3.2-triton-inference/undefined-1/model-analyzer.md)
    * [Performance Analyzer](ch4.-deep-learning-inference-service/3.2-triton-inference/undefined-1/performance-analyzer.md)
  * [Monitoring](ch4.-deep-learning-inference-service/3.2-triton-inference/monitoring.md)
* [3.3 AML(Azure Machine Learning) Service](ch4.-deep-learning-inference-service/3.3-aml-azure-machine-learning-service/README.md)
  * [Azure ARC enabled Kubernetes](ch4.-deep-learning-inference-service/3.3-aml-azure-machine-learning-service/azure-arc-enabled-kubernetes.md)
  * [Azure ML Endpoint](ch4.-deep-learning-inference-service/3.3-aml-azure-machine-learning-service/azure-ml-endpoint.md)
* [3.4 Deploy Model](ch4.-deep-learning-inference-service/3.4.md)
* [3.5 Inference Service with gRPC](ch4.-deep-learning-inference-service/3.5-grpc/README.md)
  * [Introduction of gRPC Protocol](ch4.-deep-learning-inference-service/3.5-grpc/grpc.md)
  * [Ingress Gateway for gRPC](ch4.-deep-learning-inference-service/3.5-grpc/grpc-ingress-gateway.md)
  * [gRPC Gateway for RESTAPI](ch4.-deep-learning-inference-service/3.5-grpc/restful-api-grpc-gateway.md)

## CH5.Deploy

* [Github Action](ch5.deploy/github-action/README.md)
  * [Kustomize 패키징](ch5.deploy/github-action/kustomize.md)
* [CI/CD Pipelines](ch5.deploy/ci-cd/README.md)
  * [Page 1](ch5.deploy/ci-cd/page-1.md)
* [ArgoCD Gitops Pattern](ch5.deploy/argocd-gitops.md)

## CH6.Monitoring

* [Prometheus & Grafana](ch6.monitoring/prometheus-and-grafana.md)
* [Datadog Monitoring System](ch6.monitoring/datadog.md)
