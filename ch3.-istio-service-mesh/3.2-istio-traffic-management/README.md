# 3.2 Book Info 를 이용한 Traffic Management

이전 장의 내용을 기반으로 실제실습을 진행 해봅시다

## 실습 : BookInfo <a href="#bookinfo" id="bookinfo"></a>

.

4개의 `microservice`로 이루어진 간단한 application을 배포해보겠습니다.

### 기능 <a href="#undefined" id="undefined"></a>

* 책의 정보를 보여주는 페이지.
* 책의 설명, 상세정보, 책의 리뷰정보를 확인하는 페이지.

### 구조 <a href="#undefined" id="undefined"></a>

`productpage` : details와 reviews 서비스를 호출\
`details` : 책의 상세정보\
`reviews` : 책의 리뷰가 담겨있고 ratings서비스를 호출\
`ratings` : 책의 별점정보

추가로 `reviews` 서비스는 세가지 버전이 존재합니다.

* `v1` : `ratings` 서비스를 호출하지 않음 (only 리뷰)
* `v2` : `ratings` 서비스를 호출
* `v3` : `ratings` 서비스를 호출 + 별의 색깔이 빨간색

Istio를 사용하지 않고 서비스를 배치한다면 다음과 같은 구조일 것입니다.

![image](https://user-images.githubusercontent.com/15958325/71655230-3f07f400-2d79-11ea-866a-b497ca90c4b8.png)

Istio를 사용한 구조는 다음과 같습니다. **application을 수정하지 않고** envoy proxy만 sidecar로 각 서비스에 삽입된 것을 확인할 수 있습니다.\


![](https://user-images.githubusercontent.com/15958325/71655801-04538b00-2d7c-11ea-8a1c-2463f6f4e31b.png)

\
\
