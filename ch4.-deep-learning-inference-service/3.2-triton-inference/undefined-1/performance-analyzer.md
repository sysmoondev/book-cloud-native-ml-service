# Performance Analyzer

## Perf Analyzer

TRTIS를 서버를 실행했으면 TRTIS 클라이언트를 통해서 추론을 해봅시다.

TRTIS 클라이언트도 따로 아래의 명령어로 TRTIS 클라이언트 도커 이미지를 pull 하고 컨테이너에 들어갑니다.

```bash
$ sudo docker pull nvcr.io/nvidia/tritonserver:21.04-py3-sdk
$ sudo docker run --rm -it --net=host nvcr.io/nvidia/tritonserver:21.04-py3-sdk
```

성공적으로 attach 하면 이 명령어를 통해서 성능을 확인해봅니다.

```bash
$ perf_analyzer -m EDSR -u localhost:8001 -i grpc --concurrency-range 1:8 -f result.csv
```

\-m 옵션은 서버의 모델명을 적어줍니다. test\_models 하위에 있는 폴더 이름을 의미합니다. -u 는 서버의 IP를 정하고 포트번호를 정해줍니다. http는 8000번 gRPC는 8001 번으로 설정하였고, 여기서는 8001번인 gRPC 프로토콜을 이용해서 접속하겠습니다. -i 는 defaults는 http고 gRPC 통신을 사용하기 위해 grpc를 넣어주었습니다. --concureency-range는 말 그대로 동시 접속을 의미하고 1:8은 동시접속 1\~8까지 테스팅을 하겠다는 의미입니다. -f 옵션은 성능 분석 결과를 csv로 저장하는 옵션입니다. 보다 자세한 옵션은 -h 옵션을 통해 확인하시기 바랍니다.

참고로 --streaming 옵션도 있는데 gRPC 환경에서만 이 옵션을 사용할 수 있고, 거의 모든 상황에서 streaming 옵션이 더 좋은 성능을 제공하기 때문에, --streaming 옵션을 사용하시는게 좋습니다. 여기서는 streaming 옵션을 사용하지 않겠습니다.

성능 측정 결과는 csv 형태로 다음과 같이 확인 가능합니다.

```
Concurrency: 1, throughput: 18.2 infer/sec, latency 55233 usec
Concurrency: 2, throughput: 35 infer/sec, latency 56711 usec
Concurrency: 3, throughput: 52.8 infer/sec, latency 56981 usec
Concurrency: 4, throughput: 49.8 infer/sec, latency 80075 usec
Concurrency: 5, throughput: 55.4 infer/sec, latency 90266 usec
Concurrency: 6, throughput: 59.6 infer/sec, latency 101349 usec
Concurrency: 7, throughput: 56.2 infer/sec, latency 124410 usec
Concurrency: 8, throughput: 55.6 infer/sec, latency 143304 usec
```

EDSR 모델의 동접 성능은 throughput 값이 Saturation 되기 전 가장 높을 떄의 시점 즉, 동시접속이 6개 일때 가장 좋은 성능(infer/sec)을 보여줍니다. 그 이상은 오히려 추론 성능이 좋아지지 않고 latency 만 증가하고 있습니다. Throughput 성능(infer/sec)을 중요시하면 동시접속 6이 효율적이고 latency를 중요시하면 동시접속 3이 효율적인 적이기 때문에 서비스 특정에 맞게 조절이 필요합니다.

이처럼 perf\_analyzer 도구를 통해 서비스 특성에 맞는 효율적인 동접 기준을 찾아내어 Inferencing 성능을 최적화할 수 있습니다.



