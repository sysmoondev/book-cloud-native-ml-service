# Performance Analyzer

## Perf Analyzer

모델의 추론 성능을 최적화하는데 있어서 중요한 부분은 다양한 최적화 전략을 실험하면서 성능 변화를 측정할 수 있어야 합니다. perf\_analyzer 응용 프로그램(이전에는 perf\_client로 알려짐)은 Triton 추론 서버에 대해 이 작업을 수행합니다. perf\_analyzer는 여러 소스에서 사용할 수 있는 클라이언트 예제에 포함되어 있습니다.

perf\_analyzer 애플리케이션은 모델에 대한 추론 요청을 생성하고 해당 요청의 처리량과 대기 시간을 측정합니다. 대표적인 결과를 얻기 위해 perf\_analyzer는 시간 창에 대한 처리량과 대기 시간을 측정한 다음 안정적인 값을 얻을 때까지 측정을 반복합니다. 기본적으로 perf\_analyzer는 평균 대기 시간을 사용하여 안정성을 결정하지만 --percentile 플래그를 사용하여 해당 신뢰 수준을 기반으로 결과를 안정화할 수 있습니다. 예를 들어 --percentile=95를 사용하면 95번째 백분위수 요청 지연 시간을 사용하여 결과가 안정화됩니다.

Quick Start 과정에서 실행한 Triton Server 를 대상으로 Perf Analyzer 테스트를 위해 tonserver:21.04-py3-sdk 컨테이너를 실행하고 쉘에 접속합니다.

```bash
docker run -it --rm --net=host nvcr.io/nvidia/tritonserver:22.02-py3-sdk
```

perf_analyzer 실행시, 성능 실험 대상모델은 inception\_graphdef 을 설정하고, percentile=95 백분위에 해당하는 Latency 값을 사용하여 추론 성능을 최적화 합니다._

```
$ perf_analyzer -m inception_graphdef --percentile=95
*** Measurement Settings ***
  Batch size: 1
  Measurement window: 5000 msec
  Using synchronous calls for inference
  Stabilizing using p95 latency

Request concurrency: 1
  Client:
    Request count: 348
    Throughput: 69.6 infer/sec
    p50 latency: 13936 usec
    p90 latency: 18682 usec
    p95 latency: 19673 usec
    p99 latency: 21859 usec
    Avg HTTP time: 14017 usec (send/recv 200 usec + response wait 13817 usec)
  Server:
    Inference count: 428
    Execution count: 428
    Successful request count: 428
    Avg request latency: 12005 usec (overhead 36 usec + queue 42 usec + compute input 164 usec + compute infer 11748 usec + compute output 15 usec)

Inferences/Second vs. Client p95 Batch Latency
Concurrency: 1, throughput: 69.6 infer/sec, latency 19673 usec
```

### 동접 요청

기본적으로 perf\_analyzer는 모델에서 가능한 가장 낮은 부하를 주입하기 위해 기본적으로 동접수를 1개로 설정하여 모델의 대기 시간과 처리량을 측정합니다. 이를 위해 perf\_analyzer는 하나의 추론 요청을 Triton에 보내고 응답을 기다립니다. 해당 응답이 수신되면 perf\_analyzer는 즉시 다른 요청을 보내고 측정 기간 동안 이 프로세스를 반복합니다.

\--concurrency-range :: 옵션을 사용하면 perf\_analyzer가 요청 동시성 수준 범위에 대한 데이터를 수집하도록 할 수 있습니다. --help 옵션을 사용하여 이 옵션과 기타 옵션에 대한 전체 문서를 확인하십시오. 예를 들어 1에서 4까지의 요청 동시성 값에 대한 모델의 지연 시간 및 처리량을 보려면 다음을 수행합니다.

```
root@t4-vm01:/workspace# perf_analyzer -m inception_graphdef --concurrency-range 1:4
*** Measurement Settings ***
  Batch size: 1
  Using "time_windows" mode for stabilization
  Measurement window: 5000 msec
  Latency limit: 0 msec
  Concurrency limit: 4 concurrent requests
  Using synchronous calls for inference
  Stabilizing using average latency

Request concurrency: 1
  Client:
    Request count: 527
    Throughput: 105.4 infer/sec
    Avg latency: 9478 usec (standard deviation 165 usec)
    p50 latency: 9465 usec
    p90 latency: 9702 usec
    p95 latency: 9734 usec
    p99 latency: 9803 usec
    Avg HTTP time: 9470 usec (send/recv 117 usec + response wait 9353 usec)
  Server:
    Inference count: 633
    Execution count: 633
    Successful request count: 633
    Avg request latency: 8435 usec (overhead 17 usec + queue 19 usec + compute input 155 usec + compute infer 8239 usec + compute output 5 usec)

Request concurrency: 2
  Client:
    Request count: 573
    Throughput: 114.6 infer/sec
    Avg latency: 17466 usec (standard deviation 195 usec)
    p50 latency: 17473 usec
    p90 latency: 17682 usec
    p95 latency: 17749 usec
    p99 latency: 17931 usec
    Avg HTTP time: 17455 usec (send/recv 126 usec + response wait 17329 usec)
  Server:
    Inference count: 687
    Execution count: 687
    Successful request count: 687
    Avg request latency: 16359 usec (overhead 18 usec + queue 7638 usec + compute input 157 usec + compute infer 8540 usec + compute output 6 usec)

Request concurrency: 3
  Client:
    Request count: 569
    Throughput: 113.8 infer/sec
    Avg latency: 26318 usec (standard deviation 231 usec)
    p50 latency: 26332 usec
    p90 latency: 26614 usec
    p95 latency: 26693 usec
    p99 latency: 26812 usec
    Avg HTTP time: 26309 usec (send/recv 129 usec + response wait 26180 usec)
  Server:
    Inference count: 684
    Execution count: 684
    Successful request count: 684
    Avg request latency: 25218 usec (overhead 18 usec + queue 16456 usec + compute input 155 usec + compute infer 8583 usec + compute output 6 usec)

Request concurrency: 4
  Client:
    Request count: 567
    Throughput: 113.4 infer/sec
    Avg latency: 35214 usec (standard deviation 320 usec)
    p50 latency: 35210 usec
    p90 latency: 35642 usec
    p95 latency: 35743 usec
    p99 latency: 35949 usec
    Avg HTTP time: 35202 usec (send/recv 130 usec + response wait 35072 usec)
  Server:
    Inference count: 681
    Execution count: 681
    Successful request count: 681
    Avg request latency: 34101 usec (overhead 20 usec + queue 25308 usec + compute input 156 usec + compute infer 8611 usec + compute output 6 usec)
```

### 성능측정 결과에 대한 이해

각 요청 동시성 수준에 대해 perf\_analyzer는 클라이언트에서 볼 때(즉, perf\_analyzer에서 볼 때) 대기 시간 및 처리량과 서버의 평균 요청 대기 시간을 보고합니다.

서버 대기 시간은 서버에서 요청을 받은 후 서버에서 응답을 보낼 때까지의 총 시간을 측정합니다. 서버 끝점을 구현하는 데 사용되는 HTTP 및 GRPC 라이브러리로 인해 총 서버 대기 시간은 일반적으로 HTTP 요청에 대해 더 정확합니다. 이는 수신된 첫 번째 바이트에서 마지막 바이트가 전송될 때까지의 시간을 측정하기 때문입니다. HTTP 및 GRPC 모두에 대해 총 서버 대기 시간은 다음 구성 요소로 나뉩니다.

* queue: 모델의 인스턴스가 사용 가능해질 때까지 기다리는 요청에 의해 추론 일정 대기열에서 소요된 평균 시간입니다.
* compute: GPU로/에서 데이터를 복사하는 데 필요한 시간을 포함하여 실제 추론을 수행하는 데 소요된 평균 시간입니다.

클라이언트 대기 시간은 HTTP 및 GRPC에 대해 다음과 같이 세분화됩니다.

* HTTP: send/recv는 클라이언트가 요청을 보내고 응답을 받는 데 소요한 시간을 나타냅니다. 응답 대기는 서버의 응답을 기다리는 시간을 나타냅니다.
* GRPC: (un)marshal request/response는 요청 데이터를 GRPC protobuf로 마샬링하고 GRPC protobuf에서 응답 데이터를 unmarshalling하는 데 소요된 시간을 나타냅니다. 응답 대기는 네트워크에 GRPC 요청을 쓰고 응답을 기다리고 네트워크에서 GRPC 응답을 읽는 시간을 나타냅니다.

각 요청 동시성 수준에 대해 실행되는 안정화 단계를 포함하여 더 많은 출력을 보려면 perf\_analyzer에 대한 자세한 정보(-v) 옵션을 사용하십시오.



### Latency & Throughput 시각화

perf\_analyzer는 -f 옵션을 제공하여 결과의 ​​CSV 출력이 포함된 파일을 생성합니다.

```
perf_analyzer -m inception_graphdef --concurrency-range 1:4 -f perf.csv
```

성능 측정결과 perf.csv 파일을 살펴보면 각 동접에 대한 Latency 와 Throughput 에 대한 성능 결과를 요약 정리하여 나타냅니다.

```
Concurrency,Inferences/Second,Client Send,Network+Server Send/Recv,Server Queue,Server Compute Input,Server Compute Infer,Server Compute Output,Client Recv,p50 latency,p90 latency,p95 latency,p99 latency
1,105,118,962,21,168,8233,5,0,9539,9714,9758,9796
4,113.6,127,1037,25267,159,8604,6,0,35208,35622,35702,35900
3,114,128,991,16438,157,8573,6,0,26280,26611,26705,26842
2,114.8,127,969,7638,158,8525,6,0,17449,17661,17720,17800
```

perf.csv 파일의 결과를 엑셀에서 copy & paste 하여 Throughput & Latency 관계를 차트로 표현 가능합니다. 차트를 분석해보면 동접이 2개 부터 Throughput (Inference / Second) 값이 114 근처로 Saturation 되고, 그 이후 동접이 증가하더라도 Latency 만 증가할뿐 Throughput 이 증가하지 않는 현상을 확인할 수 있습니다.

![](<../../.gitbook/assets/스크린샷 2022-05-18 오후 11.17.07 (1).png>)



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



