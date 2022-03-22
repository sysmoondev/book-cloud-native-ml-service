---
description: >-
  Client 는 HTTP/REST or gRPC 프로토콜 또는 C API 를 이용하여 Triton Server 와 통신할 수 있습니다. 각
  프로토콜과 API 에 대한 사용법에 대해 소개 합니다. Triton은 KFServing 프로젝트에서 제안한 standard inference
  protocos 를 기반으로, HTTP/REST 와 GRPC endpoints
---

# Inference Protocol & API

## HTTP Options

Triton은 HTTP 프로토콜을 통한 서버-클라이언트 네트워크 트랜잭션에 대해 다음과 구성 옵션을 제공합니다.

### Compression

#### **Triton Client(C++)**

Triton은 Client HTTP Request/Response 메시지에 대해 실시간 압축할 수 있습니다.

c++ 클라이언트의 경우 [http\_client.h](https://github.com/triton-inference-server/client/blob/main/src/c%2B%2B/library/http\_client.h) 파일 안에 Infer 와 SyncInfer 함수에 정의된 request\_compression\_algorithm 과 response\_compression\_algorithm 파라미터 확인할 수 있습니다. 기본적으로 제공되는 압축옵션은 CompressionType::NONE 입니다.

* AsyncInfer 함수

```
Error AsyncInfer(
      OnCompleteFn callback, const InferOptions& options,
      const std::vector<InferInput*>& inputs,
      const std::vector<const InferRequestedOutput*>& outputs =
          std::vector<const InferRequestedOutput*>(),
      const Headers& headers = Headers(),
      const Parameters& query_params = Parameters(),
      const CompressionType request_compression_algorithm =
          CompressionType::NONE,
      const CompressionType response_compression_algorithm =
          CompressionType::NONE);
```

* Infer 함수

```
Error Infer(
      InferResult** result, const InferOptions& options,
      const std::vector<InferInput*>& inputs,
      const std::vector<const InferRequestedOutput*>& outputs =
          std::vector<const InferRequestedOutput*>(),
      const Headers& headers = Headers(),
      const Parameters& query_params = Parameters(),
      const CompressionType request_compression_algorithm =
          CompressionType::NONE,
      const CompressionType response_compression_algorithm =
          CompressionType::NONE);
```

#### **Triton Client(Python)**

Tritoin C++ 클라이언트와 비슷하게 Python 환경에서도 동일한 압출 알고리즘 관련 파라미터request_compressionalgorithm, response_\__compression\_algorithm 값을_  [_http/\_\_init\_\_.py_](https://github.com/triton-inference-server/client/blob/main/src/python/library/tritonclient/http/\_\_init\_\_.py) _파일 안의 infer 와 async\_infer 함수에서 확인할 수 있습니다._

* infer 함수

```
def infer(self,
              model_name,
              inputs,
              model_version="",
              outputs=None,
              request_id="",
              sequence_id=0,
              sequence_start=False,
              sequence_end=False,
              priority=0,
              timeout=None,
              headers=None,
              query_params=None,
              request_compression_algorithm=None,
              response_compression_algorithm=None):
```

* async\_infer 함수

```
def async_infer(self,
                    model_name,
                    inputs,
                    model_version="",
                    outputs=None,
                    request_id="",
                    sequence_id=0,
                    sequence_start=False,
                    sequence_end=False,
                    priority=0,
                    timeout=None,
                    headers=None,
                    query_params=None,
                    request_compression_algorithm=None,
                    response_compression_algorithm=None):
```

#### SSL/TLS

[SSL/TLS](https://github.com/triton-inference-server/client/tree/main#ssltls) 옵션을 통해 Client 가 Triton Server 네트워크 구간 안전한 보안 채널을 통해 Infer Request/Response 메시지를 주고 받을 수 있습니다. 하지만 보통 상용 서비스 환경에서는 대용량 서비스 트래픽 관리를 위해 LoadBalancer 서비스 관문에 설정하고, 백엔드 클러스터를 쿠버네티스를 사용하는 경우 Ingress 를 통해 서비스 래픽을 관리 합니다.

특히 쿠버네티스 클러스터 환경에서 Ingress 를 활용하는 경우 클러스터 내 인증서 관리를 위한 Cert Manager 와 인증서 무료 발급이 가능한 Let's Encrypt 솔루션을 활용하여 인증서 발행/갱신 자동화 하고, 이를 Ingress Gateway 에 적용하여 Client 와 Ingress 네트워크 구간 E2E 보안이 가능합니다. 이후 Ingrss SSL Termination 을 통해 복호화된 Inference Request 메시지를 Triton Server 와 HTTP/REST or gRPC 평문 통신이 가능합니다.

따라서 상용 서비스 환경에서는  별도의 LoadBalacner 또는 Ingress 에 SSL/TLS 적용하고, 개발/테스트 환경에서는 Triton Server 에 SSL/TLS 옵션 설정을 테스트 가능합니다.



## gRPC Options

Triton은 서버-클라이언트 네트워크 트랜잭션을 구성하기 위한 다양한 GRPC 매개변수를 노출합니다. HTTP/REST 압축 방식과는 다르게 gRPC는 client-side/server-side 에서 압축을 위한 옵션을 설정해야 합니다. server-side 옵션의 사용 방법은 tritonserver --help의 출력을 통해 확인 가능합니다. 기본적으로는 HTTP/REST 압축 방식과 동일하게 함수 레벨에서 필요한 파리미터를 python/C++ 언어별로 지원하고 있습니다.

### Compression

#### server-side

Triton 은 server-side --grpc-infer-response-compression-level 옵션을 사용하여  Request/Reponse 메시지에 대한 실시간 압축을 지원합니다.&#x20;

#### client-side

[client-side gRPC 압축](https://github.com/triton-inference-server/client/tree/main#compression-1) 방법을 통해 gRPC 트랜잭션에 실시간 압축이 가능합니다.&#x20;

C++ 클라이언트의 경우 [grpc\_client.h](https://github.com/triton-inference-server/client/blob/main/src/c%2B%2B/library/grpc\_client.h)의 Infer, AsyncInfer 및 StartStream 함수에서 compression\_algorithm 매개변수를 참조하십시오. 기본적으로 매개변수는 GRPC\_COMPRESS\_NONE으로 설정됩니다.

* AsyncInfer

```
Error AsyncInfer(
      OnCompleteFn callback, const InferOptions& options,
      const std::vector<InferInput*>& inputs,
      const std::vector<const InferRequestedOutput*>& outputs =
          std::vector<const InferRequestedOutput*>(),
      const Headers& headers = Headers(),
      grpc_compression_algorithm compression_algorithm = GRPC_COMPRESS_NONE);
```

* StartStream

```
Error StartStream(
      OnCompleteFn callback, bool enable_stats = true,
      uint32_t stream_timeout = 0, const Headers& headers = Headers(),
      grpc_compression_algorithm compression_algorithm = GRPC_COMPRESS_NONE);
```

마찬가지로 Python 클라이언트의 경우 infer의 compression\_algorithm 매개변수, [grpc/**init**.py](https://github.com/triton-inference-server/client/blob/main/src/python/library/tritonclient/grpc/\_\_init\_\_.py)의 async\_infer 및 start\_stream 함수를 참조하세요.

* async\_infer

```
def async_infer(self,
                    model_name,
                    inputs,
                    callback,
                    model_version="",
                    outputs=None,
                    request_id="",
                    sequence_id=0,
                    sequence_start=False,
                    sequence_end=False,
                    priority=0,
                    timeout=None,
                    client_timeout=None,
                    headers=None,
                    compression_algorithm=None):
```

* start\_stream

```
def async_infer(self,
                    model_name,
                    inputs,
                    callback,
                    model_version="",
                    outputs=None,
                    request_id="",
                    sequence_id=0,
                    sequence_start=False,
                    sequence_end=False,
                    priority=0,
                    timeout=None,
                    client_timeout=None,
                    headers=None,
                    compression_algorithm=None):
```

### **GRPC KeepAlive**

keepalive ping은 전송을 통해 HTTP2 핑을 보내 채널이 현재 작동 중인지 확인하는 방법입니다. 주기적으로 발송됩니다**.** 그리고 특정 시간 초과 기간 내에 피어가 ping을 확인하지 않으면 전송이 끊어집니다.

Client/Server Side 의 keepalive ping 주요 핵심 채널 설정 값은 다음과 같습니다.

* `--grpc-keepalive-time`
* `--grpc-keepalive-timeout`
* `--grpc-keepalive-permit-without-calls`
* `--grpc-http2-max-pings-without-data`
* `--grpc-http2-min-recv-ping-interval-without-data`
* `--grpc-http2-max-ping-strikes`

위 설정 값을 client/server side 별로 적용 가능합니다.

| Channel Argument                                               | Client              | Server             |
| -------------------------------------------------------------- | ------------------- | ------------------ |
| GRPC\_ARG\_KEEPALIVE\_TIME\_MS                                 | INT\_MAX (disabled) | 7200000 (2 hours)  |
| GRPC\_ARG\_KEEPALIVE\_TIMEOUT\_MS                              | 20000 (20 seconds)  | 20000 (20 seconds) |
| GRPC\_ARG\_KEEPALIVE\_PERMIT\_WITHOUT\_CALLS                   | 0 (false)           | 0 (false)          |
| GRPC\_ARG\_HTTP2\_MAX\_PINGS\_WITHOUT\_DATA                    | 2                   | 2                  |
| GRPC\_ARG\_HTTP2\_MIN\_RECV\_PING\_INTERVAL\_WITHOUT\_DATA\_MS | N/A                 | 300000 (5 minutes) |
| GRPC\_ARG\_HTTP2\_MAX\_PING\_STRIKES                           | N/A                 | 2                  |

**client/server-side**

* GRPC\__ARG\_KEEPALIVE_\__TIME\__MS: keepalive ping이 전송된 이후, 전송 주기(MS Miille Seconds)를 제어하기 위한 파라미터 입니다.
* GRPC\__ARG\_KEEPALIVE_\__TIMEOUT\__MS: keepalive ping 발신자가 ACK 메시지를 기다리는 시간(밀리초)을 제어합니다. 이 시간 내에 승인을 받지 못하면 연결을 클라이언트가 종료합니다.
* GRPC\__ARG\_HTTP2\_MAX\_PINGS\_WITHOUT\_DATA:_ 이 채널 인수는 보낼 데이터/헤더 프레임이 없을 때 보낼 수 있는 최대 핑 수를 제어합니다. 제한을 초과하면 gRPC Core는 계속 핑을 보내지 않습니다. 0으로 설정하면 이러한 제한 없이 핑을 보낼 수 있습니다. (이것은 A8-client-side-keepalive.md와 일치하지 않는 불행한 설정입니다. 이상적으로는 keepalive ping에 이러한 제한이 없어야 하며 앞으로 더 이상 사용하지 않을 계획입니다.)
* GRPC\__ARG\_HTTP2\_MAX\_PERMIT\_WITHOUT\_CALLS:_ 이 채널 인수를 1(0: false, 1: true)로 설정하면 진행 중인 호출이 없는 경우에도 keepalive ping을 보낼 수 있습니다.

**server-side**

server-side 에서 keepalive ping을 위해 적용이 필요한 인자 값입니다.

* GRPC\_ARG\_HTTP2\_MIN\_RECV\_PING\_INTERVAL\_WITHOUT\_DATA\_MS: 전송에 데이터/헤더 프레임이 전송되지 않는 경우 서버 측의 이 채널 인수는 gRPC Core가 연속적인 핑을 수신하는 사이에 예상하는 최소 시간(밀리초)을 제어합니다. 연속 ping 사이의 시간이 이 시간보다 짧으면 ping은 피어에서 잘못된 ping으로 간주됩니다. 이러한 핑은 '핑 스트라이크'로 간주됩니다. 클라이언트 측에서는 아무런 영향을 미치지 않습니다.
*   GRPC\_ARG\_HTTP2\_MAX\_PING\_STRIKES: 이 인수는 HTTP2 GOAWAY 프레임을 보내고 전송을 닫기 전에 서버가 허용할 잘못된 핑의 최대 수를 제어합니다. 0으로 설정하면 서버가 잘못된 핑을 얼마든지 수락할 수 있습니다.\


    \
    \
    \


