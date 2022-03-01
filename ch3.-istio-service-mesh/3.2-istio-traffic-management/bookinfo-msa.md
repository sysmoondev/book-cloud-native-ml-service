# 3.2.1 맛보기 BookInfo MSA 샘플 애플리케이션

### [https://gruuuuu.github.io/cloud/service-mesh-istio/](https://gruuuuu.github.io/cloud/service-mesh-istio/)

### 예제 이용



mac 기준 설치

window 기준설치

###

### Bookinfo Micro Service

이번 장에서는 Istio 에서 제공하는 샘플 애플리케이션 [Bookinfo](https://istio.io/v1.7/docs/examples/bookinfo/)의 MSA 아키텍처 구조에 대해 소개합니다. Bookinfo는 크게 istio의 주요 기능을 테스트할 수 있도록 Bookinfo 샘플 애플리케이션을 제공하고 있으며, 여기에는 총 4개의 마이크로서비스 (productpage, details, reviews, ratings) 로 구성되어 있으며, 서로 다른 프로그래밍 언어로 개발되어 독립 배포 되고, 서비스들 간 API 통신합니다.

* productpage (python)
* reviews (java)
* details (ruby)
* ratings (nodejs)

각 마이크로 서비스들은 Istio 위에서 어떠한 제약 조건 없이 Service Mesh 를 테스트하기에 좋은 샘플 예제를 제공합니다. 이에 대한 마이크로 서비스 특징과 유기적인 연결관계에 대해 소개하겠습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ee68af1b-55aa-47d2-b70b-0055851b70cf/Untitled.png)

#### Bookinfo 배포

**BookInfo 샘플 애플리케이션 설치**

위 Bookinfo MSA 애플리케이션을 쿠버네티스 클러스터에 배포하여 테스트 환경을 구축합니다. 외부 사용자는 Ingress Envoy 를 통해 product page 에 http 접속 요청이 가능하고, 이후 productpage 와 API 통신하는 각 마이크로서비스들 간 연동을 통해 Bookinfo 서비스 전체 서비스 플로우를 이해할 수 있습니다. 이후 Istio Service Mesh 주요기능을 테스트하기 위한 샘플 애플리케이션으로 활용합니다.

**automatic sidecar injection 을 위한 labeling 설정**

default 네임스페이스에 배포되는 모든 Pod 에 Envoy Sidecar Proxy 를 자동으로 주입하기 위해 다음과 같이 istio-injection 을 라벨링 합니다.

```bash
kubectl label namespace default istio-injection=enabled
```

default 네임스페이스에 istio-injectio=enabled 라벨링 적용 여부를 확인할 수 있습니다.

```bash
kubectl describe ns default                                                                                                                                                            [13:39:51]
Name:         default
Labels:       istio-injection=enabled
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```

B**ookInfo Micro Service 배포**

BookInfo 각 마이크로서비스 모듈 productpage, bookinfo-details, ratings, reviews 가 설치된 것을 확인할 수 있습니다.

```bash
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
```

**BookInfo 애플리케이션 배포 결과 확인**

BookInfo 는 여러개의 MSA 애플리케이션으로 구성되어 있고, 이를 위한 서비스가 정상적으로 실행된 것을 확인할 수 있습니다.

* productpage
* details
* ratings
* reviews

배포된 서비스 항목들은 다음과 같이 확인 가능합니다.

```bash
kubectl get services
NAME          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.0.66.45    <none>        9080/TCP   6m57s
kubernetes    ClusterIP   10.0.0.1      <none>        443/TCP    22d
productpage   ClusterIP   10.0.219.45   <none>        9080/TCP   6m57s
ratings       ClusterIP   10.0.37.69    <none>        9080/TCP   6m57s
reviews       ClusterIP   10.0.228.26   <none>        9080/TCP   6m57
```

BookInfo MSA 각 애플리케이션 Pod 가 Sidecar Pattern 이 적용되어 envoy proxy가 포함되어 배포된 것을 확인할 수 있습니다.

```bash
kubectl get pod
                                                                                                                                               [13:49:59]
NAME                              READY   STATUS              RESTARTS   AGE
details-v1-79f774bdb9-8tlck       2/2     Running             0          5m9s
productpage-v1-6b746f74dc-4gcfw   2/2     Running             0          5m8s
ratings-v1-b6994bb9-425mk         2/2     Running             0          5m9s
reviews-v1-545db77b95-7v8zg       2/2     Running             0          5m8s
reviews-v2-7bf8c9648f-vwd8b       2/2     Running             0          5m9s
reviews-v3-84779c7bbc-c2vhx       2/2     Running             0          5m9s
```

**Ingress IP/Port 정보 확인**

이제 Bookinfo 서비스가 실행되고 있으므로 Kubernetes 클러스터 외부(예: 브라우저)에서 애플리케이션에 액세스할 수 있도록 해야 합니다. 이를 위해 Istio 게이트웨이가 사용됩니다.

1. Ingress Gateway 배포

```bash
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

* samples/bookinfo/networking/bookinfo-gateway.yaml

```bash
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

1. Virtual Gateway, Virtual Service 배포 확인

```bash
kubectl get gw,vs
                                                                                                                                                [15:01:55]
NAME                                           AGE
gateway.networking.istio.io/bookinfo-gateway   9s

NAME                                          GATEWAYS               HOSTS   AGE
virtualservice.networking.istio.io/bookinfo   ["bookinfo-gateway"]   ["*"]   9s
```

1. Ingress Gateway IP, Port 확인

Ingress Gateway 의 External-IP 와 포트 정보를 확인하여, 외부 테스트에 필요한 endpoint 정보를 확인합니다.

```bash
kubectl get svc istio-ingressgateway -n istio-system

NAME                   TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)                                                                                                                    AGE
istio-ingressgateway   LoadBalancer   10.0.6.39    20.196.242.199   15021:32522/TCP,80:32220/TCP,443:30688/TCP,31400:31853/TCP,15443:32487/TCP,50051:31924/TCP,8000:31511/TCP,8001:31407/TCP   64d
```

```bash
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
export TCP_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')
```

1. GATEWAY\_URL 설정

```bash
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

**BookInfo 애플리케이션 동작 검증**

BookInfo 애플리케이션의 정상 동작 확인을 위해 Istio Ingres Gateway URL 주소를 이용하여 curl 테스트 할 수 있습니다. 아래와 같이 curl 명령어를 통해 Bookinfo 서비스의 $GATEWAY\_URL/productpage GET 요청에 대한 응답 값을 확인할 수 있습니다.

```bash
curl -s "<http://$>{GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"

<title>Simple Bookstore App</title>
```

**Destination Rule 적용**

Istio를 사용하여 Bookinfo 각 마이크로 서비스의 버전별 라우팅을 제어하려면 먼저 Destination Rule 에서 subset 정의를 통해 버전을 정의해야 합니다.

아래와 같이 Bookinfo 샘플 애플리케이션에서 이미 제공하고 있는 Destination Rule 설정 파일 이용하여 배포 합니다.

```bash
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

samples/bookinfo/networking/destination-rule-all.yaml 파일을 살펴보면 아래와 같이 구성되어 있습니다.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ratings
spec:
  host: ratings
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v2-mysql
    labels:
      version: v2-mysql
  - name: v2-mysql-vm
    labels:
      version: v2-mysql-vm
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: details
spec:
  host: details
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
---
```

#### Bookinfo 비지니스 로직

**Productpage**

productapage 는 고객이 직접 브라우징하여 접속할 수 있는 페이지 입니다. 온라인 서점과 동일한 형식으로 책에 대한 주요 정보를 고객에게 전달하기 위한 화면을 제공하고, 이를 위해 Detail, Review 마이크로서비스들과 API 통신하여 책에 대한 정보를 가져옵니다.

**Details**

* Type
* ISBN-NO
* number of pages
* Publisher
* Language
*

[2.3.2 Bookinfo MSA 설치시](https://github.com/istiokrsg/istio\_book\_kr/blob/master/chapter-2/2.1.1.minikube-feature/2.2.1.istio-install-guide/2.3.2-bookinfo-msa.md), Load Balancer Type 으로 서비스를 오픈하지 않았기 때문에 내부 포트포워딩을 통해서만 로컬호스트 페이지 접속이 가능합니다. 아래와 같이 현재 Bookinfo 애플리케이션들을 구성하는 pod, service 를 확인하고, 서비스 포트를 확인하여 포트포워딩 설정을 진행합니다.

```yaml
kubectl get service,pod -n default

NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/dcgm-exporter   ClusterIP   10.0.35.130   <none>        9400/TCP   65d
service/details         ClusterIP   10.0.2.223    <none>        9080/TCP   44h
service/kubernetes      ClusterIP   10.0.0.1      <none>        443/TCP    66d
service/productpage     ClusterIP   10.0.56.117   <none>        9080/TCP   44h
service/ratings         ClusterIP   10.0.243.65   <none>        9080/TCP   44h
service/reviews         ClusterIP   10.0.86.150   <none>        9080/TCP   44h

NAME                                  READY   STATUS    RESTARTS   AGE
pod/details-v1-79f774bdb9-8tlck       2/2     Running   0          44h
pod/productpage-v1-6b746f74dc-4gcfw   2/2     Running   0          44h
pod/ratings-v1-b6994bb9-425mk         2/2     Running   0          44h
pod/reviews-v1-545db77b95-7v8zg       2/2     Running   0          44h
pod/reviews-v2-7bf8c9648f-vwd8b       2/2     Running   0          44h
pod/reviews-v3-84779c7bbc-c2vhx       2/2     Running   0          44h
```

productpage 는 service/prodcutpage 이름으로 ClusterIP(10.0.219.45) 로 배포되었고, 서비스 접속을 위한 컨테이너 포트는 9080/TCP 로 오픈되었습니다. 아래와 같이 포트포워딩 설정을 통해 내부 로컬호스트에서 접속 가능하도록 설정합니다.

```yaml
kubectl port-forward service/productpage -n default 8080:9080
```

이후,[http://localhost:8080](http://localhost:8080) 브라우징을 통해 접속 가능합니다. 브라우저 접속을 하면 아래와 같이 /productpage 가 포함하고 있는 각 마이크 서비스들의 endpoint 정보를 확인할 수 있습니다.

![Bookinfo /product 페이지](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1fd3d328-edd8-4c67-9829-3ea92fcbc3d9/Untitled.png)

Bookinfo /product 페이지

**Endpoint 정보**

* details ([http://details:9080](http://details:9080))
* reviews ([http://reviews:9080](http://reviews:9080))
* ratings ([http://ratings:9080](http://ratings:9080))

/productpage 서비스의 경우 사용자가 외부 퍼블릭 망에서 접속 가능하도록 Ingress Gateway 설정을 통해 endpoint 를 구성했습니다. 사용자의 http request 메시지가 Istio Ingress Gateway 를 거쳐 이후 producpage 하위에 있는 서비스들 까지 연동하기 위해 사용되는 Endpont 정보는 쿠버네티스 클러스터내 Core DNS 에서 제공하는 내부 서비스 도메인을 활용합니다.

Bookinfo 는 사용자 계정별 다른 서비스 시나리오 테스트를 위해 미리 생성해놓은 2가지 타입의 계정 Normal user, Test user 를 제공하고 있습니다. 3. Traffic Management 에서 사용자 계정을 이용한 트래픽 처리 방안에 대해 자세히 설명 하겠습니다. 여기서는 productpage 화면 확인을 위해 둘중, 아무 계정을 클릭하여 페이지 구성 확인이 가능합니다. 여기서는 Test user 계정을 선택했습니다.

**Reviews**

책에 대한 독자의 평가 후기를 서평으로 남길 수 있고, 이에 대한 rating을 별점 (0\~5개)을 연계하여 책에 대한 평가를 할 수 있습니다. Bookinfo 애플리케이션 아키텍처를 보면 reviews 는 총 3개의 버전으로 배포되어 있습니다.

* Reviews-v1 Ratings 서비스와 연동하지 않는 버전입니다.
* Reviews-v2 Ratings 서비스와 연동하고, 별점을 1\~5점까지 검은색으로 표시하는 버전입니다.
* Reviews-v3 Ratings 서비스와 연동하고, 별점을 1\~5점까지 빨간색으로 표시하는 버전입니다.

**Ratings**

독자는 리뷰 작성과 함께 ratings(별점) 정보를 제공 합니다. 특정 책에 대해 별점을 1\~5개까지 선택하여 책을 평가할 수 있습니다. 따라서 Review 애플리케이션은 Ratings 서비스와 API 연동이 필요합니다.
