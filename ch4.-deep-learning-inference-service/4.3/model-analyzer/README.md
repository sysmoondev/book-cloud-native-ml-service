# Model Analyzer

Triton 모델 분석기는 Triton 추론 서버 모델의 컴퓨팅 및 메모리 요구 사항을 더 잘 이해하는 데 도움이 되는 CLI 도구입니다. 이 보고서는 사용자가 다양한 구성의 장단점을 더 잘 이해하고 Triton Inference Server의 성능을 최대화하는 구성을 선택하는 데 도움이 됩니다.

### Features

* [Automatic and manual configuration search](https://github.com/triton-inference-server/model\_analyzer/blob/main/docs/config\_search.md)

모델 분석기는 모델 구성의 최대 배치 사이즈, 동적 배치 및 인스턴스 그룹 매개변수에 대한 최적의 설정을 자동으로 찾는 데 도움이 될 수 있습니다. 모델 분석기는 성능 분석기를 사용하여 요청의 다양한  동접 및 배치 크기로 모델을 테스트합니다. 수동 구성 검색을 사용하여 모델 구성에서 지정할 수 있는 모든 매개변수에 대한 수동 스위프를 생성할 수 있습니다.

* [Detailed and summary reports](https://github.com/triton-inference-server/model\_analyzer/blob/main/docs/report.md)

Model Analyzer는 모델에 사용할 수 있는 서로 다른 모델 구성 간의 균형을 더 잘 이해하는 데 도움이 되는 요약되고 자세한 보고서를 생성할 수 있습니다.

* [QoS Constraints](https://github.com/triton-inference-server/model\_analyzer/blob/main/docs/config.md#constraint)

제약 조건은 QoS 요구 사항에 따라 모델 분석기 결과를 필터링하는 데 도움이 될 수 있습니다. 예를 들어 지연 시간 예산을 지정하여 지정된 지연 시간 임계값을 충족하지 않는 모델 구성을 필터링할 수 있습니다.

