---
description: >-
  https://www.notion.so/skvps/VPS-GRPC-GATEWAY-for-RESTAPI-fd9bb6bf350c4a4ea09d4a7ce248de06
---

# RESTful API 지원을 위한 gRPC Gateway 구축



### Introduce

gRPC 는 prootbuf 이용하여 RESTful HTTP 통신 방식에 비해 빠른 통신이 가능합니다. 하지만 gRPC 프로토콜은 브라우저와 같은 다양한 클라이언트 환경에서 지원하지 않는 단점이 존재합니다. 이를 위해 Google 에서 [gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway) 오프소스 제공을 통해 RESTful API 를 gRPC 프로토콜로 변환하는 방법을 제공하고 있습니다. 이를 이용하여 VPS gRPC 서비스 환경에서 RESTful API 제공을 위한 [VPS-GRPC-GATEWAY](https://github.com/sysmoon/vps-grpc-gateway) 를 개발하여 적용합니다.

#### gRPC 소개

gRPC 는 protobuf 를 인터페이스 정의 언어(IDL, Interface Definition Language) 를 사용하여 클라이언트와 서버 간 통신하기 위한 메시지를 스키마 형태로 정의하여 통신하는 형태입니다. gRPC 에서는 클라이언트 애플리케이션이 마치 로컬 함수를 호출 하듯이 원격 서버에 있는 함수를 RPC(Remote Procedure Call) 형태로 호출할 수 있습니다. 이를 통해 분산된 애플리케이션과 서비스를 쉽게 만들 수 있습니다. RESTFull API 의 경우 API Docs 명세서를 클라이언트가 알아야 서버와의 통신이 가능한 반면, gRPC의 경우 proto 파일에 정의된 인터페이스 구조를 통해 클라이언트와 서버가 각각 해당 인터페이스를 구현하고, 클라이언트의 RPC 원격 호출을 다룰 수 있는 gRPC Server 를 운영합니다. 클라이언트 사이드에서는 gRPC Server 에서 구현된 동일한 메소드를 제공하는 stub 를 통해 서버와 통신합니다.

아래 그림과 같이 클라이언트와 서버가 Protocol Buffer 통신을 위해 .proto 파일에 통신을 위한 인터페이스를 설계하고, 이후 각각 개발하고자 하는 언어로 protoc 컴파일하면 해당 언어에서 개발 가능한 gRPC Server/Stub 코드가 자동으로 생성됩니다. 각 클라이언트는 해당 gRPC Stub 를 이용하여 Proto Request (RPC Remote Procedure Call) 하고, 이에 대한 응답을 Proto Reponse 로 받는 구조입니다.

![](http://thenewstack.io/wp-content/uploads/2016/09/gRPC-1.png)

**gRPC 장점**

1. 빠른 전송 (데이터 직렬화)

protocol buffer 는 구조체 형태로 사용자가 정의한 구조화된 데이터를 직렬화(Serialize) 하여 바이터리 형태로 압축 전송하기 때문에 네트워크 전송시 빠르게 전송 가능합니다. ([JSON 직렬화 대비 성능 8대 향상](https://docs.microsoft.com/ko-kr/dotnet/architecture/cloud-native/grpc))

1. 통신 방법의 명확성 (IDL)

gRPC 통신을 위해서는 protocol buffer 에 인터페이스 정의 언어 (IDL) 이용하여 어떻게 데이터를 주고 받을 것인지 정의합니다. 따라서 RESTFul API 처럼 OpenAPI Doc(ex: Swagger) 개발자가 이해하여 코드를 개발할 필요 없이, protoc 컴파일에서 자동으로 code generates 된 gRPC Server/Stub 코드를 이용하여 개발 생산성을 높일 수 있습니다.

1. HTTP/2 프로토콜 지원

gRPC는 HTTP/2 프로토콜을 지원합니다. HTTP/1.1 지원하지만 HTTP/2 에서 제공하는 HTTP Pipeline 기능을 통해 클라이언트와 서버간 생성된 TCP Connection 안에서 Proto Request 메시지를 sync or async 모드로 병렬 전송이 가능합니다. 따라서 기존 HTTP/1.1 기반의 RESTFul API 통신처럼 매번 요청시 발생하는 TCP Connection 생성에 발생하는 네트워크 자원의 비효율성을 줄일 수 있습니다.

1. 마이크로서비스 최적화

마이크로서비스는 독립적인 배포하여 서비스가 가능한 단위로 특히 쿠버네티스 클러스터 안에 Pod 형태로 배포되어 서비스들 간에 내부 통신이 많은 경우 효율적인 데이터 명세화를 통한 빠른 통신이 필수적입니다. 위 gRPC 의 장점은 마이크로서비스에 최적화 되어있습니다. proto buffer 통신하기 위한 방법을 명세화하고, 각 담당자가 해당 서비스에 최적화된 프로그래밍 언어로 protoc 컴파일 하여 개발하고, 서비스 간 gRPC 통신이 가능합니다. 이를 통해 개발자들 간 통신 방법에 대한 명확성을 통해 원하는 서비스를 쿠버네티스 클러스터 환경에서 빠르게 개발/운영 가능합니다.

![](https://miro.medium.com/max/627/0\*qnJGeU-x16alxlc9.png)

**gRPC 단점**

* 브라우저 통신미지원

브라우저와 gRPC 서버 간 통신은 아직 지원하지 않습니다. 이를 위해 Google 에서 gRPC-Gateway 오픈소스를 통해 RESTFul API 요청을 받아 protobuf 형식으로 데이터를 자동 변환하여 백엔드 gRPC Server 와 통신이 가능한 gRPC Reverse Proxy 기능을 제공하고 있습니다. 이 방법 외에 브라우저에서 직적접으로 gRPC 통신을 가능하도록 [gRPC-WEB](https://github.com/grpc/grpc-web) 과 gRPC-Proxy 를 이용하여 구현 가능하지만, 여전히 추가적인 아키텍처 설계가 필요하므로 [고려해야 할 항목](https://velog.io/@kyusung/grpc-web-example)이 증가합니다.

### Protocol Buffer 사용법

기본적으로 gRPC는 상호 통신을 위해 [Protocol Buffer](https://developers.google.com/protocol-buffers/docs/overview) 를 사용합니다. Protocol Buffer 는 스키마 형태의 구조화된 데이터를 Serializing 하여 바이너리 형태로 네트워크 전송을 최적화 하기 위한 메커니즘을 제공합니다.

Protocol Buffer 를 사용하기 위해서는 .proto 파일에 클라이언트와 서버가 통신하기 위한 meesage, service 를 아래 스키마 형태로 정의합니다.

```protobuf
message Person {
	string name = 1;
	int32 id = 2;
	bool has_ponycopter = 3;
}
```

이후, protoc 컴파일러를 이용하여 원하는 프로그래밍 언어(C++, Java, Go, Python, JS..) 로 데이터 접근 가능한 클래스 파일들 생성할 수 있습니다. 이 클래스 파일들은 데이터에 대한 접근을 위해 필요한 기본적인 GetXXX(), SetXXX() 함수들을 제공하고, 전체 데이터 구조를 raw bytes 바이너리 형태로 직렬화/파싱 하기 위한 메소드도 제공합니다. 예들 들면 위 person.proto 파일을 Go 언어로 선택하여 컴파일하면 Go 언어에서 사용 가능한 person.pb.go 확장자의 파일이 생성되고, 이 파일에서 Person 이라는 클래스 제공을 통해 데이터를 제어할 수 있고, 데이터를 직렬화하여 클라이언트와 서버 간 통신이 가능합니다.

.proto 파일에는 통신하기 위한 message 외에 RPC 원격 메소드 호출을 위한 service를 정의할 수 있습니다. 아래와 같이 클라이언트, 서버 통신하기 위한 메세지(HelloRequest, HelloReply) 를 정의하고, 이를 gRPC Server 에서 서비스(Greeter) 하기 위한 RPC 함수(SayHello) 정의합니다.

```protobuf
// The gretter service definition
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) return (HelloReply) {}
}

// The request message containing the user's name
message HelloRequest {
  string name = 1;
}

// The response message contraining the greetings
message HelloReply {
  string message = 1;
}
```

### Installation

#### Go 개발환경 구축

**Go 설치**

1. **Go Download**

go 공식홈페이지를 통해 [Download Go for Mac](https://go.dev/doc/install) 버전을 다운로드 받아 설치합니다.

![](../../.gitbook/assets/go\_download.png)

1. Go 버전 체크

```bash
go version                                                                                                                                                                              [11:10:17]
go version go1.17.3 darwin/amd64
```

1. Go 설치환경 체크

* GOROOT (Go Root 디렉토리)

GOROOT golang SDK 가 설치되는 위치 입니다.

```bash
go env GOROOT
/usr/local/go

tree -L 1 -d /usr/local/go                                                                                                                                                                    [11:40:55]
go
├── api
├── bin
├── doc
├── lib
├── misc
├── pkg
├── src
└── test
```

* GOPATH

go 는 외부 라이브러리 모듈을 쉽게 가져와서 사용할 수 있으며, 이때 3rd party 라이브러리를 GOPATH 에 설치 관리합니다. Go 프로젝트마다 참조하는 외부 라이브러리가 다르기 때문에 프로젝트 마다 GOPATH 를 적절히 설정하여 관리가 필요합니다. Go를 설치하면 기본적으로 $HOME/go 위치를 사용합니다.

```go
go env GOPATH
/Users/sysmoon/go
```

* Go 실행파일

```bash
which go
/usr/local/go/bin/go
```

* GOBIN (Go 플러그인 및 모듈 바이너리 파일)

```bash
go env GOBIN
/Users/sysmoon/go/bin
```

1. Build & Run 테스트

* hello.go 파일 생성

```go
package main

import "fmt"

func main() {
	fmt.Printf("hello, world\\n")
}
```

* hello.go 빌드

```go
# output 바이너리 파일을 -o 옵션을 활용하여 hello 이름으로 지정
go build -o hello hello.go
```

* hello 실행

```go
./hello
hello, world
```

**VSCode Go 개발환경**

blabla

### Architecture

#### gRPC-Gateway

gRPC-Gateway 는 protobuf 컴파일러(protoc)의 플러그인 입니다. gRPC는 클라이언트와 서버 간 통신을 위한 메시지를 proto 파일 (profile-service.proto) 에 스키마 형태로 정의하고, proto 파일을 gRPC-Gateway 플러그인을 이용하여 gRPC Reverse Proxy 코드를 자동으로 생성(genereates proxy) 합니다. Reverse Proxy 는 RSETful HTTP API 메시지를 수신하여 protobuf 메시지를 자동 변환하여 백엔드 gRPC Server (Your gRPC Service) 에 요청/응답 받는 역할을 수행합니다. 백엔드 gRPC Server 는 protoc 컴파일러를 통해 stub 코드를 자동으로 생성(generates stub)하여 백엔드 비지니스 로직 개발이 가능합니다.

아래 그림은 API Client 가 HTTP PUT 메소드를 이용하여 Body 메시지에 JSON 형식의 메시지를 Reverse Proxy 로 요청하고, 이후 JSON 을 protobuf 자동 변환하여 백엔드 gRPC Server 와 통신하는 과정을 보여주고 있습니다.



![](https://grpc-ecosystem.github.io/grpc-gateway/assets/images/architecture\_introduction\_diagram.svg)

###
