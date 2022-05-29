# Table of contents

* [README](README.md)

## Ch1. Introduction

* [1.1 Cloud Native 기술소개](ch1.-introduction/1.1-cloud-native.md)
* [1.3 MSA 아키텍처 방법론 소개](ch1.-introduction/1.3-msa.md)

## Ch2. Kubernetes

* [2.1 쿠버네티스 소개](ch2.-kubernetes/2.1-kubernetes/README.md)
  * [쿠버네티스 탄생 배경](ch2.-kubernetes/2.1-kubernetes/undefined.md)
  * [쿠버네티스 컴포넌트](ch2.-kubernetes/2.1-kubernetes/undefined-1.md)
  * [쿠버네티스 구성](ch2.-kubernetes/2.1-kubernetes/undefined-2.md)
  * [쿠버네티스 오브젝트](ch2.-kubernetes/2.1-kubernetes/and.md)
  * [워크로드, 파드(pod)](ch2.-kubernetes/2.1-kubernetes/2.1.1.md)
  * [서비스, 로드밸런싱, 네트워킹](ch2.-kubernetes/2.1-kubernetes/undefined-3.md)
  * [스토리지](ch2.-kubernetes/2.1-kubernetes/undefined-2-1.md)
  * [Configuration](ch2.-kubernetes/2.1-kubernetes/secret.md)
  * [Security](ch2.-kubernetes/2.1-kubernetes/config-map.md)
* [2.2 쿠버네티스 시작하기](ch2.-kubernetes/2.2/README.md)
  * [다양한 환경에서 Kubernetes 구축하기](ch2.-kubernetes/2.2/2.2.1-on-premise-kubernetes.md)
  * [sample application 배포하기](ch2.-kubernetes/2.2/2.1.2-azure-public-cloud-aks.md)
* [2.3 쿠버네티스 활용하기](ch2.-kubernetes/2.3/README.md)
  * [helm vs kustomize](ch2.-kubernetes/2.3/helm-vs-kustomize.md)
  * [ingress controller](ch2.-kubernetes/2.3/ingress-controller.md)
  * [node pool 구분하기](ch2.-kubernetes/2.3/node-pool.md)
* [2.3 Container Registry](ch2.-kubernetes/2.3-container-registry-acr.md)
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
  * [3.2.1 맛보기 BookInfo Traffic Management](ch3.-istio-service-mesh/3.2-istio-traffic-management/bookinfo-msa.md)
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

* [4.1 Triton 소개](ch4.-deep-learning-inference-service/3.2-triton-inference/README.md)
  * [Architecture](ch4.-deep-learning-inference-service/3.2-triton-inference/architecture.md)
  * [Quick Start](ch4.-deep-learning-inference-service/3.2-triton-inference/github-integration.md)
  * [Inference Protocol & API](ch4.-deep-learning-inference-service/3.2-triton-inference/inference-protocol-and-api.md)
  * [Monitoring](ch4.-deep-learning-inference-service/3.2-triton-inference/monitoring.md)
* [4.2 모델 관리](ch4.-deep-learning-inference-service/4.2/README.md)
  * [4.2.1 모델 구성](ch4.-deep-learning-inference-service/4.2/4.2.1.md)
  * [모델 저장소](ch4.-deep-learning-inference-service/4.2/undefined/README.md)
    * [모델 버전 및 디렉토리 구조](ch4.-deep-learning-inference-service/4.2/undefined/undefined.md)
    * [클라우드 저장소](ch4.-deep-learning-inference-service/4.2/undefined/undefined-1.md)
* [4.3 모델 성능 최적화](ch4.-deep-learning-inference-service/4.3/README.md)
  * [Model Analyzer](ch4.-deep-learning-inference-service/4.3/model-analyzer/README.md)
    * [Installation](ch4.-deep-learning-inference-service/4.3/model-analyzer/installation.md)
  * [Performance Analyzer](ch4.-deep-learning-inference-service/4.3/performance-analyzer.md)
* [4.4 추론 서비스 (/w gRPC)](ch4.-deep-learning-inference-service/3.5-grpc/README.md)
  * [Introduction of gRPC Protocol](ch4.-deep-learning-inference-service/3.5-grpc/grpc.md)
  * [Ingress Gateway for gRPC](ch4.-deep-learning-inference-service/3.5-grpc/grpc-ingress-gateway.md)
  * [gRPC Gateway for RESTAPI](ch4.-deep-learning-inference-service/3.5-grpc/restful-api-grpc-gateway.md)
* [4.5 DALI Backend](ch4.-deep-learning-inference-service/4.5-dali-backend/README.md)
  * [DALI Backend 소개](ch4.-deep-learning-inference-service/4.5-dali-backend/dali-backend.md)
  * [DALI backend 예제 (/w Densenet)](ch4.-deep-learning-inference-service/4.5-dali-backend/dali-backend-w-densenet.md)

## CH5.Deploy

* [Github Action](ch5.deploy/github-action/README.md)
  * [Kustomize 패키징](ch5.deploy/github-action/kustomize.md)
* [CI/CD Pipelines](ch5.deploy/ci-cd/README.md)
  * [Page 1](ch5.deploy/ci-cd/page-1.md)
* [ArgoCD Gitops Pattern](ch5.deploy/argocd-gitops.md)

## CH6.Monitoring

* [Prometheus & Grafana](ch6.monitoring/prometheus-and-grafana.md)
* [Datadog Monitoring System](ch6.monitoring/datadog.md)
