# Model Analyzer

Triton 모델 분석기는 성능 분석기를 사용하여 GPU 메모리 및 컴퓨팅 사용률을 측정하는 동안 모델에 요청을 보내는 도구입니다. 모델 분석기는 다양한 일괄 처리 및 모델 인스턴스 구성에서 모델에 대한 GPU 메모리 요구 사항을 특성화하는 데 특히 유용합니다. 이 GPU 메모리 사용량 정보가 있으면 GPU의 메모리 용량 내에서 유지하면서 동일한 GPU에서 여러 모델을 결합하는 방법을 보다 지능적으로 결정할 수 있습니다.

### Features

* [Automatic and manual configuration search](https://github.com/triton-inference-server/model\_analyzer/blob/main/docs/config\_search.md)

모델 분석기는 모델 구성의 최대 배치 크기, 동적 배치 및 인스턴스 그룹 매개변수에 대한 최적의 설정을 자동으로 찾는 데 도움이 될 수 있습니다. 모델 분석기는 성능 분석기를 사용하여 요청의 다양한  동접 및 배치 크기로 모델을 테스트합니다. 수동 구성 검색을 사용하여 모델 구성에서 지정할 수 있는 모든 매개변수에 대한 수동 스위프를 생성할 수 있습니다.

* [Detailed and summary reports](https://github.com/triton-inference-server/model\_analyzer/blob/main/docs/report.md)

Model Analyzer는 모델에 사용할 수 있는 서로 다른 모델 구성 간의 균형을 더 잘 이해하는 데 도움이 되는 요약되고 자세한 보고서를 생성할 수 있습니다.

* [QoS Constraints](https://github.com/triton-inference-server/model\_analyzer/blob/main/docs/config.md#constraint)

제약 조건은 QoS 요구 사항에 따라 모델 분석기 결과를 필터링하는 데 도움이 될 수 있습니다. 예를 들어 지연 시간 예산을 지정하여 지정된 지연 시간 임계값을 충족하지 않는 모델 구성을 필터링할 수 있습니다.



추론에 사용할 GPU 사용율에 대한 정보가 없으면 GPU에서 실행할 모델 수를 이해하는데 많은 어려움이 있을 수 있습니다. Hot 및 Cold 스토리지 요구 사항을 수집하여 이를 사용하여 다음과 같은 여러 이점을 얻기 위해 모델 일정을 알릴 수 있습니다.

![Screenshot of Model Analyzer.](https://developer-blogs.nvidia.com/wp-content/uploads/2020/08/model-analyzer-1.png)

* **Maximized model throughput**

각 GPU에 배치된 모델의 합계가 100%와 같은 사용 가능한 메모리 및 GPU 사용률의 특정 임계값을 초과하지 않는지 확인합니다. 이는 하드웨어의 처리량을 최대화합니다.

* **Optimized hardware usage**

GPU 메모리 요구 사항을 조사하여 더 적은 하드웨어에서 더 많은 모델을 실행합니다. 처리량을 최적화하는 대신 이 데이터를 사용하여 GPU당 로드할 수 있는 최대 모델 수를 결정하여 필요한 하드웨어를 줄이거나 처리량과의 균형을 맞출 수 있습니다.

* **Increased reliability**

GPU에 로드하는 모델이 해당 기능을 초과하지 않을 것임을 알고 메모리 부족 오류를 제거합니다.



또한 다음과 같은 두 가지 중요한 비 일정 이점이 있습니다.

* **Efficient models**

모델 성능에 대한 추가 데이터 포인트로 컴퓨팅 요구 사항을 사용하여 다양한 모델을 비교하고 대조합니다. 이렇게 하면 더 가벼운 모델을 생성하고 추론 요구 사항에 필요한 메모리 양을 줄일 수 있습니다.

* **Better hardware sizing**

메모리 요구 사항을 사용하여 모델을 실행하는 데 필요한 정확한 하드웨어 양을 결정합니다.

간단히 말해서, 추론 모델의 컴퓨팅 요구 사항을 이해하면 모델 생성 및 하드웨어 크기 조정에서 안정적이고 효율적인 모델 실행에 이르기까지 많은 이점을 얻을 수 있습니다.
