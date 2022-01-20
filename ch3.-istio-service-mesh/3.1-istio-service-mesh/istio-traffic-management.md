# Istio Traffic Management

서비스 메쉬는 마이크로 서비스들 사이의 네트워킹을 가리키는 용어입니다. 서비스가 성장하면 할수록 서비스 메쉬의 규모가 커지고 복잡해집니다. 서비스 메쉬의 관리가 어려워진다는 의미는 service discovery, load balancing, failure recovery, metrics, monitoring 등의 문제를 해결해야 한다는 것입니다. 이뿐 아니라 A/B testing, canary rollouts, rate limiting, access control and end-to-end authentication 등의 더 많은 기능들 또한 서비스 메쉬가 풀어야할 요구사항들입니다.

이러한 요구사항들을 만족시키기 위한 기본적인 기능이 바로 트래픽 제어 입니다. 본 장에서는 Istio 의 트래픽 제어에 대한 내용들을 소개합니다.

메시 내에서 트래픽을 전달하기 위해 Istio는 모든 엔드포인트가 어디에 있고 어떤 서비스에 속하는지 알아야 합니다. 자체 서비스 레지스트리를 채우기 위해 Istio는 서비스 검색 시스템에 연결합니다. 예를 들어 Kubernetes 클러스터에 Istio를 설치한 경우 Istio는 해당 클러스터의 서비스와 엔드포인트를 자동으로 감지합니다. 이 서비스 레지스트리를 사용하여 Envoy 프록시는 트래픽을 관련 서비스로 보낼 수 있습니다. 대부분의 마이크로 서비스 기반 애플리케이션에는 서비스 트래픽을 처리하기 위해 각 서비스 워크로드의 여러 인스턴스가 있으며, 때때로 로드 밸런싱 풀이라고도 합니다. 기본적으로 Envoy 프록시는 라운드 로빈 모델을 사용하여 각 서비스의 로드 밸런싱 풀 전체에 트래픽을 분산합니다. 이 모델에서는 요청이 각 풀 구성원에게 차례로 전송되고 각 서비스 인스턴스가 요청을 받으면 풀의 맨 위로 돌아갑니다.

Istio의 기본 서비스 검색 및 로드 밸런싱은 작동하는 서비스 메시를 제공하지만 Istio가 할 수 있는 모든 것과는 거리가 멉니다. 대부분의 경우 메시 트래픽에 발생하는 상황을 보다 세밀하게 제어해야 할 수 있습니다. A/B 테스트의 일부로 특정 비율의 트래픽을 새 버전의 서비스로 보내거나 서비스 인스턴스의 특정 하위 집합에 대한 트래픽에 다른 로드 밸런싱 정책을 적용할 수 있습니다. 메시로 들어오거나 나가는 트래픽에 특수 규칙을 적용하거나 메시의 외부 종속성을 서비스 레지스트리에 추가할 수도 있습니다. Istio의 트래픽 관리 API를 사용하여 Istio에 고유한 트래픽 구성을 추가하여 이 모든 작업과 그 이상을 수행할 수 있습니다.

다른 Istio 구성과 마찬가지로 API는 예제에서 볼 수 있듯이 YAML을 사용하여 구성할 수 있는 Kubernetes 사용자 지정 리소스 정의(CRD)를 사용하여 지정됩니다. 이 가이드의 나머지 부분에서는 각 트래픽 관리 API 리소스와 이 리소스로 수행할 수 있는 작업을 살펴봅니다. 이러한 리소스는 다음과 같습니다.

* [Virtual services](https://istio.io/latest/docs/concepts/traffic-management/#virtual-services)
* [Destination rules](https://istio.io/latest/docs/concepts/traffic-management/#destination-rules)
* [Gateways](https://istio.io/latest/docs/concepts/traffic-management/#gateways)
* [Service entries](https://istio.io/latest/docs/concepts/traffic-management/#service-entries)
* [Sidecars](https://istio.io/latest/docs/concepts/traffic-management/#sidecars)

### Virtual Service

대상 규칙과 함께 가상 서비스는 Istio 트래픽 라우팅 기능의 핵심 빌딩 블록입니다. 가상 서비스를 사용하면 Istio 및 플랫폼에서 제공하는 기본 연결 및 검색을 기반으로 Istio 서비스 메시 내에서 서비스로 요청을 라우팅하는 방법을 구성할 수 있습니다. 각 가상 서비스는 순서대로 평가되는 일련의 라우팅 규칙으로 구성되어 Istio가 가상 서비스에 대한 각 요청을 메시 내의 특정 실제 대상과 일치시킬 수 있습니다. 사용 사례에 따라 메시에 여러 가상 서비스가 필요하거나 전혀 필요하지 않을 수 있습니다.

가상 서비스는 Istio의 트래픽 관리를 유연하고 강력하게 만드는 데 핵심적인 역할을 합니다. 클라이언트가 실제로 구현하는 대상 워크로드에서 요청을 보내는 위치를 강력하게 분리하여 이를 수행합니다. 가상 서비스는 또한 트래픽을 해당 워크로드로 보내기 위한 다양한 트래픽 라우팅 규칙을 지정하는 다양한 방법을 제공합니다.

이것이 왜 그렇게 유용합니까? 가상 서비스가 없으면 Envoy는 도입부에 설명된 대로 모든 서비스 인스턴스 간에 라운드 로빈 로드 밸런싱을 사용하여 트래픽을 분산합니다. 워크로드에 대해 알고 있는 정보로 이 동작을 개선할 수 있습니다. 예를 들어, 일부는 다른 버전을 나타낼 수 있습니다. 이는 다양한 서비스 버전의 백분율을 기반으로 트래픽 경로를 구성하거나 내부 사용자의 트래픽을 특정 인스턴스 집합으로 보내려는 A/B 테스트에서 유용할 수 있습니다.

가상 서비스를 사용하면 하나 이상의 호스트 이름에 대한 트래픽 동작을 지정할 수 있습니다. 가상 서비스의 트래픽을 적절한 대상으로 보내는 방법을 Envoy에 알려주는 가상 서비스의 라우팅 규칙을 사용합니다. 경로 대상은 동일한 서비스의 버전이거나 완전히 다른 서비스일 수 있습니다.

일반적인 사용 사례는 서비스 하위 집합으로 지정된 서비스의 다른 버전으로 트래픽을 보내는 것입니다. 클라이언트는 가상 서비스 호스트가 단일 엔터티인 것처럼 요청을 보내고 Envoy는 가상 서비스 규칙에 따라 트래픽을 다른 버전으로 라우팅합니다. 이 사용자에서 버전 2로 이동합니다." 이를 통해 예를 들어 새 서비스 버전으로 전송되는 트래픽의 비율을 점진적으로 늘리는 카나리아 롤아웃을 만들 수 있습니다. 트래픽 라우팅은 인스턴스 배포와 완전히 별개입니다. 즉, 새 서비스 버전을 구현하는 인스턴스의 수는 트래픽 라우팅을 전혀 참조하지 않고 트래픽 부하에 따라 확장 및 축소할 수 있습니다. 대조적으로 Kubernetes와 같은 컨테이너 오케스트레이션 플랫폼은 인스턴스 확장을 기반으로 하는 트래픽 분산만 지원하므로 빠르게 복잡해집니다. Istio를 사용한 Canary Deployments에서 가상 서비스가 Canary 배포에 어떻게 도움이 되는지 자세히 알아볼 수 있습니다.

**Virtual Service 기능**

단일 가상 서비스를 통해 여러 애플리케이션 서비스를 처리합니다. 예를 들어 메시가 Kubernetes를 사용하는 경우 특정 네임스페이스의 모든 서비스를 처리하도록 가상 서비스를 구성할 수 있습니다. 단일 가상 서비스를 여러 "실제" 서비스에 매핑하는 것은 서비스 소비자가 전환에 적응할 필요 없이 모놀리식 애플리케이션을 별개의 마이크로서비스로 구축된 복합 서비스로 전환하는 데 특히 유용합니다. 라우팅 규칙은 "monolith.com의 이러한 URI에 대한 호출은 마이크로서비스 A로 이동" 등을 지정할 수 있습니다. 아래 예제 중 하나에서 이것이 어떻게 작동하는지 확인할 수 있습니다.

게이트웨이와 함께 트래픽 규칙을 구성하여 수신 및 송신 트래픽을 제어합니다.

어떤 경우에는 서비스 하위 집합을 지정하는 곳이므로 이러한 기능을 사용하도록 대상 규칙을 구성해야 합니다. 서비스 하위 집합 및 기타 대상별 정책을 별도의 개체에 지정하면 가상 서비스 간에 이를 완전히 재사용할 수 있습니다. 다음 섹션에서 대상 규칙에 대해 자세히 알아볼 수 있습니다.

#### Virtual Service 샘플 예제

다음 가상 서비스는 요청이 특정 사용자로부터 오는지 여부에 따라 다른 버전의 서비스로 요청을 라우팅합니다.

```bash
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v3
```

**The hosts field**

호스트 필드는 가상 서비스의 호스트를 나열합니다. 즉, 이러한 라우팅 규칙이 적용되는 사용자 주소 지정 대상 또는 대상입니다. 이것은 클라이언트가 서비스에 요청을 보낼 때 사용하는 주소입니다.

```bash
hosts:
- reviews
```

가상 서비스 호스트 이름은 IP 주소, DNS 이름 또는 플랫폼에 따라 암시적 또는 명시적으로 FQDN(정규화된 도메인 이름)으로 확인되는 짧은 이름(예: Kubernetes 서비스 짧은 이름)일 수 있습니다. 와일드카드("\*") 접두사를 사용하여 일치하는 모든 서비스에 대한 단일 라우팅 규칙 집합을 만들 수도 있습니다. 가상 서비스 호스트는 실제로 Istio 서비스 레지스트리의 일부일 필요가 없으며 단순히 가상 대상입니다. 이를 통해 메시 내부에 라우팅 가능한 항목이 없는 가상 호스트에 대한 트래픽을 모델링할 수 있습니다.

**Routing Rules**

http 섹션에는 호스트 필드에 지정된 대상으로 전송되는 HTTP/1.1, HTTP2 및 gRPC 트래픽을 라우팅하기 위한 일치 조건 및 작업을 설명하는 가상 서비스의 라우팅 규칙이 포함되어 있습니다(tcp 및 tls 섹션을 사용하여 라우팅을 구성할 수도 있습니다. TCP 및 종료되지 않은 TLS 트래픽에 대한 규칙). 라우팅 규칙은 사용 사례에 따라 트래픽을 보낼 대상과 0개 이상의 일치 조건으로 구성됩니다.

* **Match Condition**

예제의 첫 번째 라우팅 규칙에는 조건이 있으므로 일치 필드로 시작합니다. 이 경우 이 라우팅이 사용자 "jason"의 모든 요청에 적용되기를 원하므로 헤더, 최종 사용자 및 정확한 필드를 사용하여 적절한 요청을 선택합니다.

```bash
- match:
   - headers:
       end-user:
         exact: jason
```

* **Destination**

경로 섹션의 목적지 필드는 이 조건과 일치하는 트래픽의 실제 목적지를 지정합니다. 가상 서비스의 호스트와 달리 대상의 호스트는 Istio의 서비스 레지스트리에 존재하는 실제 대상이어야 합니다. 그렇지 않으면 Envoy는 트래픽을 어디로 보낼지 모릅니다. 이것은 프록시가 있는 메시 서비스이거나 서비스 항목을 사용하여 추가된 비 메시 서비스일 수 있습니다. 이 경우 Kubernetes에서 실행 중이고 호스트 이름은 Kubernetes 서비스 이름입니다.

```bash
route:
- destination:
    host: reviews
    subset: v2
```

이 페이지와 이 페이지의 다른 예에서는 단순성을 위해 대상 호스트에 Kubernetes 짧은 이름을 사용합니다. 이 규칙이 평가되면 Istio는 라우팅 규칙이 포함된 가상 서비스의 네임스페이스를 기반으로 도메인 접미사를 추가하여 호스트의 정규화된 이름을 가져옵니다. 예제에서 짧은 이름을 사용한다는 것은 원하는 네임스페이스에서 복사하고 시도할 수 있음을 의미합니다.

대상 섹션은 또한 이 규칙의 조건과 일치하는 요청을 보낼 이 Kubernetes 서비스의 하위 집합(이 경우 v2라는 하위 집합)을 지정합니다. 아래의 대상 규칙 섹션에서 서비스 하위 집합을 정의하는 방법을 볼 수 있습니다.

**Routing rule precedence**

라우팅 규칙은 위에서 아래로 순차적으로 평가되며 가상 서비스 정의의 첫 번째 규칙에 가장 높은 우선 순위가 부여됩니다. 이 경우 첫 번째 라우팅 규칙과 일치하지 않는 모든 항목이 두 번째 규칙에 지정된 기본 대상으로 이동하기를 원합니다. 이 때문에 두 번째 규칙에는 일치 조건이 없으며 트래픽을 v3 하위 집합으로 보냅니다.

```bash
- route:
  - destination:
      host: reviews
      subset: v3
```

가상 서비스에 대한 트래픽이 항상 하나 이상의 일치하는 경로를 갖도록 각 가상 서비스의 마지막 규칙으로 이와 같은 기본 "조건 없음" 또는 가중치 기반 규칙(아래 설명)을 제공하는 것이 좋습니다.

**More about routing rules**

위에서 보았듯이 라우팅 규칙은 트래픽의 특정 하위 집합을 특정 대상으로 라우팅하는 강력한 도구입니다. 트래픽 포트, 헤더 필드, URI 등에 대해 일치 조건을 설정할 수 있습니다. 예를 들어, 이 가상 서비스를 통해 사용자는 마치[http://bookinfo.com/에](http://bookinfo.com/%EC%97%90) 있는 더 큰 가상 서비스의 일부인 것처럼 평가 및 리뷰라는 두 개의 개별 서비스로 트래픽을 보낼 수 있습니다. 가상 서비스 규칙은 요청 URI를 기반으로 트래픽을 일치시키고 적절한 서비스로 요청을 보냅니다.

```bash
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - bookinfo.com
  http:
  - match:
    - uri:
        prefix: /reviews
    route:
    - destination:
        host: reviews
  - match:
    - uri:
        prefix: /ratings
    route:
    - destination:
        host: ratings
```

일부 일치 조건의 경우 정확한 값, 접두사 또는 정규식을 사용하여 선택하도록 선택할 수도 있습니다.

AND 조건에 동일한 일치 블록에 여러 일치 조건을 추가하거나 조건을 OR에 동일한 규칙에 여러 일치 블록을 추가할 수 있습니다. 주어진 가상 서비스에 대해 여러 라우팅 규칙을 가질 수도 있습니다. 이를 통해 단일 가상 서비스 내에서 원하는 대로 라우팅 조건을 복잡하거나 간단하게 만들 수 있습니다. 일치 조건 필드의 전체 목록과 가능한 값은 HTTPMatchRequest 참조에서 찾을 수 있습니다.

일치 조건을 사용하는 것 외에도 "가중치" 비율로 트래픽을 분산할 수 있습니다. 이는 A/B 테스트 및 카나리아 롤아웃에 유용합니다.

```bash
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 75
    - destination:
        host: reviews
        subset: v2
      weight: 25
```

라우팅 규칙을 사용하여 트래픽에 대한 몇 가지 작업을 수행할 수도 있습니다. 예를 들면 다음과 같습니다.

* Append or remove headers.
* Rewrite the URL.
* Set a [retry policy](https://istio.io/latest/docs/concepts/traffic-management/#retries) for calls to this destination.

To learn more about the actions available, see the `[HTTPRoute` reference]\([https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPRoute](https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPRoute)).

### Destination Rules

가상 서비스와 함께 대상 규칙은 Istio의 트래픽 라우팅 기능의 핵심 부분입니다. 가상 서비스는 트래픽을 지정된 대상으로 라우팅한 다음 대상 규칙을 사용하여 해당 대상의 트래픽에 발생하는 상황을 구성하는 방법으로 생각할 수 있습니다. 대상 규칙은 가상 서비스 라우팅 규칙이 평가된 후에 적용되므로 트래픽의 "실제" 대상에 적용됩니다.

특히 대상 규칙을 사용하여 지정된 서비스의 모든 인스턴스를 버전별로 그룹화하는 것과 같이 명명된 서비스 하위 집합을 지정합니다. 그런 다음 가상 서비스의 라우팅 규칙에서 이러한 서비스 하위 집합을 사용하여 서비스의 다른 인스턴스에 대한 트래픽을 제어할 수 있습니다.

또한 대상 규칙을 사용하면 전체 대상 서비스 또는 선호하는 로드 밸런싱 모델, TLS 보안 모드 또는 회로 차단기 설정과 같은 특정 서비스 하위 집합을 호출할 때 Envoy의 트래픽 정책을 사용자 지정할 수 있습니다. 대상 규칙 참조에서 대상 규칙 옵션의 전체 목록을 볼 수 있습니다.

**Load balancing options**

기본적으로 Istio는 인스턴스 풀의 각 서비스 인스턴스가 차례로 요청을 받는 라운드 로빈 로드 밸런싱 정책을 사용합니다. Istio는 또한 특정 서비스 또는 서비스 하위 집합에 대한 요청에 대한 대상 규칙에서 지정할 수 있는 다음 모델을 지원합니다.

* Random: 요청이 풀의 인스턴스에 무작위로 전달됩니다.
* 가중치: 요청이 특정 비율에 따라 풀의 인스턴스로 전달됩니다.
* 최소 요청: 요청은 요청 수가 가장 적은 인스턴스로 전달됩니다.

See the [Envoy load balancing documentation](https://www.envoyproxy.io/docs/envoy/v1.5.0/intro/arch\_overview/load\_balancing) for more information about each option.

#### Destination Rules 샘플 예제

다음 예제 대상 규칙은 부하 분산 정책이 서로 다른 my-svc 대상 서비스에 대해 세 가지 하위 집합을 구성합니다.

```bash
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
```

각 하위 집합은 Kubernetes에서 Pod와 같은 객체에 연결된 키/값 쌍인 하나 이상의 레이블을 기반으로 정의됩니다. 이러한 레이블은 Kubernetes 서비스 배포에 메타데이터로 적용되어 다른 버전을 식별합니다.

이 대상 규칙에는 하위 집합을 정의할 뿐만 아니라 이 대상의 모든 하위 집합에 대한 기본 트래픽 정책과 해당 하위 집합에 대해서만 재정의하는 하위 집합별 정책이 있습니다. 하위 집합 필드 위에 정의된 기본 정책은 v1 및 v3 하위 집합에 대한 단순 임의 로드 밸런서를 설정합니다. v2 정책에서 라운드 로빈 로드 밸런서는 해당 하위 집합의 필드에 지정됩니다.
