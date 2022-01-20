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

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b25d291-f606-46a0-ab2f-02bf366e3a43/Untitled.png)

**gRPC 장점**

1. 빠른 전송 (데이터 직렬화)

protocol buffer 는 구조체 형태로 사용자가 정의한 구조화된 데이터를 직렬화(Serialize) 하여 바이터리 형태로 압축 전송하기 때문에 네트워크 전송시 빠르게 전송 가능합니다. ([JSON 직렬화 대비 성능 8대 향상](https://docs.microsoft.com/ko-kr/dotnet/architecture/cloud-native/grpc))

1. 통신 방법의 명확성 (IDL)

gRPC 통신을 위해서는 protocol buffer 에 인터페이스 정의 언어 (IDL) 이용하여 어떻게 데이터를 주고 받을 것인지 정의합니다. 따라서 RESTFul API 처럼 OpenAPI Doc(ex: Swagger) 개발자가 이해하여 코드를 개발할 필요 없이, protoc 컴파일에서 자동으로 code generates 된 gRPC Server/Stub 코드를 이용하여 개발 생산성을 높일 수 있습니다.

1. HTTP/2 프로토콜 지원

gRPC는 HTTP/2 프로토콜을 지원합니다. HTTP/1.1 지원하지만 HTTP/2 에서 제공하는 HTTP Pipeline 기능을 통해 클라이언트와 서버간 생성된 TCP Connection 안에서 Proto Request 메시지를 sync or async 모드로 병렬 전송이 가능합니다. 따라서 기존 HTTP/1.1 기반의 RESTFul API 통신처럼 매번 요청시 발생하는 TCP Connection 생성에 발생하는 네트워크 자원의 비효율성을 줄일 수 있습니다.

1. 마이크로서비스 최적화

마이크로서비스는 독립적인 배포하여 서비스가 가능한 단위로 특히 쿠버네티스 클러스터 안에 Pod 형태로 배포되어 서비스들 간에 내부 통신이 많은 경우 효율적인 데이터 명세화를 통한 빠른 통신이 필수적입니다. 위 gRPC 의 장점은 마이크로서비스에 최적화 되어있습니다. proto buffer 통신하기 위한 방법을 명세화하고, 각 담당자가 해당 서비스에 최적화된 프로그래밍 언어로 protoc 컴파일 하여 개발하고, 서비스 간 gRPC 통신이 가능합니다. 이를 통해 개발자들 간 통신 방법에 대한 명확성을 통해 원하는 서비스를 쿠버네티스 클러스터 환경에서 빠르게 개발/운영 가능합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d2bd4e06-970f-46c0-a67a-c6d42b7bddaf/Untitled.png)

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

go 공식홈페이지를 통해 Download Go for Mac 버전을 다운로드 받아 설치합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/280a9f35-83fb-49fa-879a-6628bf7dc604/Untitled.png)

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

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e814620a-714a-49af-9db9-6cd57233f2da/Untitled.png)

### gRPC Server (Your gRPC service)구현

**Go 모듈 초기화**

먼저 go mod 명령을 통해 gRPC-Gateway 모듈을 생성합니다.

```protobuf
go mod init github.com/sysmoon/vps-grpc-gateway
```

**grpc 모듈 설치**

이후 생성된 go.mod 에 다른 모듈을 import 가능하고, 해당 모듈을 프로젝트 내에서 사용 가능합니다. go.mod 파일이 생성되었으니 go get -u <모듈> 명령어를 통해 [grpc](https://github.com/grpc/grpc-go) 모듈을 import 합니다.

```protobuf
go get -u google.golang.org/grpc
```

이후 go.mod 파일을 확인해보면 grpc 모듈에 필요한 디펜던시 모듈들을 포함하여 필요한 모듈이 import 된 것을 확인할 수 있습니다.

```protobuf
cat go.mod                                                                                                                                                                [16:00:46]

require (
	github.com/golang/protobuf v1.5.2 // indirect
	golang.org/x/net v0.0.0-20211209124913-491a49abca63 // indirect
	golang.org/x/sys v0.0.0-20211210111614-af8b64212486 // indirect
	golang.org/x/text v0.3.7 // indirect
	google.golang.org/genproto v0.0.0-20211208223120-3a66f561d7aa // indirect
	google.golang.org/grpc v1.42.0 // indirect
	google.golang.org/protobuf v1.27.1 // indirect
)
```

**Protocol Buffer 정의**

proto file:[https://github.com/sysmoon/vps-grpc-gateway/tree/main/schema](https://github.com/sysmoon/vps-grpc-gateway/tree/main/schema)

**Protocol Buffer 빌드**

해당 schema 폴더에 정의된 proto 파일을 기준으로 gRPC 서비스를 위해 protoc 로 컴파일 합니다.

[build-proto.sh](https://github.com/sysmoon/vps-grpc-gateway/blob/main/build-proto.sh)

```bash
#!/bin/bash

# grpc-server (/w SensorData)
rm -f schema/SensorData/*.go
protoc -I./ \\
	-I./schema \\
    schema/vgeodb_common.proto \\
    schema/SensorData/*.proto \\
    --go_out=. --go_opt paths=source_relative

# grpc-server (/w VPResult)
rm -f schema/VPResult/*.go    
protoc -I./ \\
    schema/VPResult/vpresult.proto \\
    schema/VPResult/prdb_meta.proto \\
    --go_out . --go_opt paths=source_relative \\
    --go-grpc_out . --go-grpc_opt paths=source_relative

# grpc-gateway
protoc -I . \\
	    --grpc-gateway_out . \\
	    --grpc-gateway_opt logtostderr=true \\
	    --grpc-gateway_opt paths=source_relative \\
        schema/VPResult/vpresult_grpc_gw.proto

# swagger
# protoc -I . --openapiv2_out ./gen/openapiv2 \\
#     --openapiv2_opt logtostderr=true \\
#     schema/VPResult/vpresult.proto
```

**gRPC Server 개발** ([grpc-server/main.go](https://github.com/sysmoon/vps-grpc-gateway/blob/main/vpgw/grpc-server/main.go))

```go
package main

import (
	"context"
	fmt "fmt"
	"log"
	"net"

	sensorpb "github.com/sysmoon/vps-grpc-gateway/schema/SensorData"
	vpresultpb "github.com/sysmoon/vps-grpc-gateway/schema/VPResult"
	"google.golang.org/grpc"
)

const portNumber = "9001"

type locationServiceServer struct {
	vpresultpb.LocationServiceServer
}

func (s *locationServiceServer) GetLocationPose(ctx context.Context, req *sensorpb.VPData) (*vpresultpb.VPServiceT, error) {
	println("get request")
	// header := req.StVPHeader
	fmt.Printf("%+v\\n", req)

	stVPResult := &vpresultpb.MetaT{
		U64KeyFrameIndex: 111,
		U64Timestamp:     1234,
	}

	return &vpresultpb.VPServiceT{
		StVPResult:             stVPResult,
		StVPResultTrackingMode: nil,
	}, nil
}

func main() {
	fmt.Println("vps tet")
	lis, err := net.Listen("tcp", ":"+portNumber)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	grpcServer := grpc.NewServer()
	vpresultpb.RegisterLocationServiceServer(grpcServer, &locationServiceServer{})

	log.Printf("start VPGW server on %s port", portNumber)
	if err := grpcServer.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %s", err)
	}
}
```

**Go Build**

[build-golang.sh](http://build-golang.sh)

```bash
# vps-grpc-server
go build -o bin/vps-grpc-server vpgw/grpc-server/main.go
```

**Go Run**

```bash
bin/vps-grpc-server
vps tet
2021/12/13 16:31:15 start VPGW server on 9001 port
```

### gRPC-Gateway (gRPC Reverse Proxy) 구현

gRPC 서비스는 쿠버네티스 클러스터에서 MSA 관점에서 통신하기에는 효율적이지만, Web 브라우저에서 gRPC 를 지원하지 않고, Mobile OS(Android, iOS) 환경에서는 별도의 gRPC Client 개발이 필요하다. 이를 위해 Google 에서 제공하는 gRPC-Gateway 를 통해 Client 사이드에서 RESTFul API 요청에 대해 gRPC Reverse Proxy 에서 protobuf 변환하여 백엔드 gRPC Server 와 연동하는 방법을 제공합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/26ca733f-3c11-4863-b9b5-3a01151ff8fd/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/401ae3ea-9b57-487d-8505-fc4a2ea981a0/Untitled.png)

#### gRPC-Gateway 컴파일러 설치

사전에 protoc 컴파일러가 설치되어 있다는 가정하에 gRPC Reverse Proxy 서버 코드 생성을 위해 grpc-gateway 컴파일러용 plugin 설치가 필요합니다.

```bash
go install \\
    github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway \\
    github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2 \\
    google.golang.org/protobuf/cmd/protoc-gen-go \\
    google.golang.org/grpc/cmd/protoc-gen-go-grpc
```

**gRPC-Gateway Plugins**

* protoc-gen-grpc-gateway

proto 파일 기반으로 gRPC Reverse Proxy 코드 생성하는 모듈

* protoc-gen-openapiv2

openapi 도구를 통해 proto 파일에 명시된 통신 인터페이스 규격을 openapi 표준 문서로 생성 (ex: swagger)

* protoc-gen-go

protoc Go 컴파일러

* protoc-gen-go-grpc

protoc Go gRPC 서버 코드 생성을 위한 컴파일러

위 플러그인 바이너리 파일들은 $GOPATH/bin 폴더에서 확인할 수 있습니다.

```bash
ls $GOPATH/bin                                                                                                                                          [20:45:25]

-rwxr-xr-x  1 sysmoon  staff    14M Nov 25 14:15 dlv
-rwxr-xr-x  1 sysmoon  staff    14M Nov 25 14:16 dlv-dap
-rwxr-xr-x  1 sysmoon  staff   3.0M Nov 25 14:15 go-outline
-rwxr-xr-x  1 sysmoon  staff   3.1M Nov 25 14:15 gomodifytags
-rwxr-xr-x  1 sysmoon  staff   4.1M Nov 25 14:15 gopkgs
-rwxr-xr-x  1 sysmoon  staff   6.3M Nov 25 14:15 goplay
-rwxr-xr-x  1 sysmoon  staff    23M Nov 25 14:16 gopls
-rwxr-xr-x  1 sysmoon  staff   9.1M Nov 25 14:15 gotests
-rwxr-xr-x  1 sysmoon  staff   5.3M Nov 25 14:15 impl
**-rwxr-xr-x  1 sysmoon  staff   7.9M Dec 13 13:16 protoc-gen-go
-rwxr-xr-x  1 sysmoon  staff   7.7M Dec 13 13:16 protoc-gen-go-grpc
-rwxr-xr-x  1 sysmoon  staff   9.6M Dec 13 13:16 protoc-gen-grpc-gateway
-rwxr-xr-x  1 sysmoon  staff    10M Dec 13 13:16 protoc-gen-openapiv2**
-rwxr-xr-x  1 sysmoon  staff   7.8M Dec  9 14:15 protoc-gen-swagger
-rwxr-xr-x  1 sysmoon  staff    11M Nov 25 14:16 staticcheck
```

#### HTTP Endpoint 추가

protocol buffer 서비스에 HTTP 접근이 가능하도록 Endpoint 를 추가해야 합니다.

VPS 서비스를 위한 RPC 가 정의된 [vpresult\_grpc\_gw.proto](https://github.com/sysmoon/vps-grpc-gateway/blob/main/schema/VPResult/vpresult\_grpc\_gw.proto) 파일에 google/api/annotaions.proto 파일을 import 합니다. 이 3rd-Party proto 파일을 import 하기 위해서 annotaions.proto, http.proto 파일을 프로젝트내 repository 로 가져와야 합니다. 이때 가장 중요한 것은 프로젝트의 레파지토리의 Root 경로에 google/api 폴더 경로에 저장해야 합니다.

annotations.proto 파일을 추가하면 rpc 내에 google.api.http option 을 사용할 수 있습니다. 이 옵션이 바로 RESTFul API 즉 HTTP 메시지를 protobuf 맵핑해주는 HTTP Endpoint 를 제공하는 기능입니다. 옵션 안에서는 HTTP Method (GET, POST, PUT, DELETE) 를 정의하고, 사용할 url pattern 과 수신할 body 메시지 즉 protobuf 스키마와 맵핑될 body 필드 영역을 지정하면 됩니다.

VPS RESTful API 의 경우 HTTP POST 방식으로 url 은 /skvps/locdata 형식이고, body 메시지 전체에 vpresult.proto 스키마에 맵핑되는 JSON 메시지 전체를 수신하도록 아래와 같이 설정합니다.

```protobuf
syntax = "proto3";

import "google/protobuf/wrappers.proto";

// import "prdb_meta.proto";
// import "vp_tracking.proto";
// import "vp_msg.proto";
import "schema/VPResult/prdb_meta.proto";
import "schema/SensorData/vp_tracking.proto";
import "schema/SensorData/vp_msg.proto";

**import "google/api/annotations.proto";**

package vpresult;

option java_package = "vp.service";
option go_package = "github.com/sysmoon/vps-grpc-gateway/schema/VPResult";

service LocationService {
	rpc getLocationPose (vpdata.VPData) returns (VPService_t) {
		**option (google.api.http) = {
            post: "/skvps/locdata",
						body: "*"
        };**
	}

	rpc getLocationInfo (vpdata.VPData) returns (.google.protobuf.FloatValue) {} // For later, multi rpc test
}

message VPService_t {
	prdb.meta.Meta_t stVPResult = 1;

	// Newly added members for Tracking Mode
	vp.tracking.MetaTrackingMode_t stVPResultTrackingMode = 2;
}
```

#### gRPC-Gateway 서비스 컴파일

HTTP Endpoint 접근이 가능한 proto 파일 (vpresult\_grpc\_gw.proto)을 정의했고, 이 파일을 **protoc-gen-grpc-gateway** 플러그인으로 컴파일 하면 Generates Proxy 기능을 통해 gRPC Reverse Proxy 코드가 생성됩니다.

[**build-proto.sh**](https://github.com/sysmoon/vps-grpc-gateway/blob/main/build-proto.sh)

```protobuf
# grpc-gateway
protoc -I . \\
	    --grpc-gateway_out . \\
	    --grpc-gateway_opt logtostderr=true \\
	    --grpc-gateway_opt paths=source_relative \\
        schema/VPResult/vpresult_grpc_gw.proto
```

위와 같이 [build-proto.sh](http://build-proto.sh) 스크립트 안에 grpc-gateway 섹션을 복사해서 컴파일 하면 schema/VPResult/vpresult\_grpc\_gw.pb.gw.go 파일이 생성된 것을 확인할 수 있고, 이 파일이 Genereated Proxy 기능을 통해 생성된 Go 파일입니다. 이 파일을 golang 을 활용하ㅇ Reverse Proxy 를 위한 비지니스 로직을 개바할 수 있습니다.

#### gRPC Reverse Proxy 구현

```protobuf
**package main

import (
	"context"
	"log"
	"net/http"

	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
	"google.golang.org/grpc"

	vpresultpb "github.com/sysmoon/vps-grpc-gateway/schema/VPResult"
)

const (
	portNumber           = "9000"
	gRPCServerPortNumber = "50051"
)

func main() {
	ctx := context.Background()
	mux := runtime.NewServeMux()
	options := []grpc.DialOption{
		grpc.WithInsecure(),
	}

	if err := vpresultpb.RegisterLocationServiceHandlerFromEndpoint(
		ctx,
		mux,
		"vpgw.vps.svc.cluster.local:"+gRPCServerPortNumber,
		options,
	); err != nil {
		log.Fatalf("failed to register gRPC gateway: %v", err)
	}

	log.Printf("start HTTP server on %s port", portNumber)
	if err := http.ListenAndServe(":"+portNumber, mux); err != nil {
		log.Fatalf("failed to serve: %s", err)
	}
}**
```

**Context**

가장 먼저 context 패키지에 있는 ctx 를 선언합니다. context 는 gRPC Reverse Proxy 와 gRPC Server 를 이어주는 맥락이라고 이해하면 편합니다. 이 값을 **RegisterLocationServiceHandlerFromEndpoint** 함수에 파라미터로 입력함으로써 gRPC Server 에서 context Done 신호가 오면 하나의 Request Call 에 대한 맥락을 즉 Connection 을 끊습니다.

**Mux**

Mux (Multiplexer) 는 수신한 HTTP Request 메시지를 백엔드 gRPC Server 로 전송/제어하기 위한 미들웨어 입니다. 이를 통해 특정 HTTP Request 메시지에 대해서만 필터링 하여 차단하거나, Rewrite, Redirect 등 메시지에 대한 제어가 가능합니다.

**Options**

gRPC Reverse Proxy 와 gRPC Server 가 연결을 맺을때 추가할 수 있는 다양한 옵션을 정의할 수 있습니다. 여기서는 특별한 옵션 없이 보안채널 없이 평문으로 통신하기 위해 grpc.DialOption() 함수 안에 **grpc.WithInsecure()** 옵션만 추가했습니다.

**gRPC Server 연동**

이제 어떤 gRPC Server 와 연동할 것인지에 대한 endpoint 정보만 입력하면 됩니다. gRPC Reverse Proxy 에서 사용할 port 번호는 **portNumber = "9000"** 정의하였고, 연동할 gRPC Server 접속 정보는 현재 AKS에 배포되어 있는 vpgw.vps.svc.cluster.local:50051 (VPGW) endpoint 로 정의했습니다. 띠라서 쿠버네티스 클러스터 환경에서 서비스 플로우를 정리하면 아래와 같은 순서로 RESTful API 요청 메시지가 전송됩니다.

ALB (Azure Load Balancer)

→ Istio Ingress Gateway([vpgw.vpslab.co.kr:443](http://vpgw.vpslab.co.kr:443))

→ gRPC Reverse Proxy(vps-grpc-gateway.vps.svc.cluster.local:9000)

→ VPGW(vpgw.vps.svc.cluster.local:50051)

#### gRPC Reverse Proxy 빌드

gRPC Reverse Proxy 를 구현한 Go 파일을 빌드하여 VPS-GRPC-GATEWAY 서버를 실행할 수 있습니다.

[**build-golang.sh**](https://github.com/sysmoon/vps-grpc-gateway/blob/main/build-golang.sh)

```protobuf
go build -o bin/vps-grpc-gateway vpgw/grpc-gateway/main.go
```

### Deploy

Azure VPS 플랫폼에 VPS-GRPC-GATEWAY 를 Pod 형태로 배포하고, 백엔드 VPGW ↔ SA ↔ PE 각 Micro Service 구간별 gRPC 통신을 통해 VPS 서비스가 가능한 시나리오를 고려하여 배포합니다.

#### Containerization

[**Dockerfile**](https://github.com/sysmoon/vps-grpc-gateway/blob/main/Dockerfile)

```protobuf
FROM golang:1.17.3-stretch AS Builder
LABEL AUTHOR Danel Moon (sysmoon@gmail.com)

RUN mkdir -p /app
WORKDIR /app 
COPY go.mod .
COPY go.sum .
RUN go mod tidy
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o bin/vps-grpc-gateway vpgw/grpc-gateway/main.go

EXPOSE 9000

ENTRYPOINT [ "/app/bin/vps-grpc-gateway" ]
```

[**Docker Build**](https://github.com/sysmoon/vps-grpc-gateway/blob/main/build-docker.sh)

```protobuf
#!/bin/bash

docker build --tag vpsregistry.azurecr.io/vps-grpc-gateway:1.0.0 -f Dockerfile .
```

#### Kubernetes Deploy

[Deployment](https://www.notion.so/apiVersion-apps-v1-kind-Deployment-metadata-name-vps-grpc-gateway-spec-selector-matchL-c05da8803aaa4659b74965bb412c6771)

```protobuf
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vps-grpc-gateway
spec:
  selector:
    matchLabels:
      app: vps-grpc-gateway
  template:
    metadata:
      labels:
        app: vps-grpc-gateway
    spec:
      containers:
      - name: vps-grpc-gateway
        image: vpsregistry.azurecr.io/vps-grpc-gateway:1.0.0
        resources:
          limits:
            memory: "256Mi"
            cpu: "3000m"
        ports:
        - containerPort: 9000
```

[Service](https://github.com/sysmoon/vps-grpc-gateway/blob/main/manifests/vps-grpc-gateway-svc.yaml)

```protobuf
apiVersion: v1
kind: Service
metadata:
  name: vps-grpc-gateway
spec:
  selector:
    app: vps-grpc-gateway
  ports:
  - port: 9000
    targetPort: 9000
```

**Istio Request Routing**

```protobuf
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: vpgw2-vs
  namespace: vps
spec:
  gateways:
  - vpgw-gateway
  hosts:
  - vpgw.vpslab.co.kr
  http:
  - match:
    - uri:
        exact: /skvps/locdata
    route:
    - destination:
        host: vps-grpc-gateway.vps.svc.cluster.local
        port:
          number: 9000
  - route:
    - destination:
        host: vpgw.vps.svc.cluster.local
        port:
          number: 50051
```

### Test

#### **RESTful API**

[**restapi-test.sh**](https://github.com/sysmoon/vps-grpc-gateway/blob/main/unittest/restapi-test.sh)

```protobuf
#!/bin/bash

# curl -XPOST -H "Content-Type: Application/json" -d @sample.json  <https://vpgw.vpslab.co.kr/skvps/locdata> | jq '.'

JSON_FILE_PATH=$1
HOST=$2
PORT=$3

if [ $# -eq 4 ]
  then
    printf "No arguments supplied"
    printf "usage: ./restapi-test.sh <json file> <host> <port> \\n"
    printf "example: ./restapi-test.sh sample.json localhost 9000 \\n"
fi

# json
printf "**********\\n"
printf "restapi test\\n"
printf "**********\\n"

curl -XPOST -H "Content-Type: Application/json" -d @${JSON_FILE_PATH}  <https://$>{HOST}:${PORT}/skvps/locdata | jq '.'

printf "**********\\n"
printf "grpcurl test\\n"
printf "**********\\n"
# grpcurl
grpcurl -vv  -user-agent 'gputype/t4' -d @ ${HOST}:${PORT} vpresult.LocationService/getLocationPose < ${JSON_FILE_PATH}
```

```protobuf
./restapi-test.sh sample.json vpgw.vpslab.co.kr 443
```

#### grpcurl

**grpcurl (/w user-agent)**

```protobuf
# grpcurl with user-agent
grpcurl -vv  -user-agent 'gputype/t4' -d @ vpgw.vpslab.co.kr:443 vpresult.LocationService/getLocationPose < sample.json
```

### Ref

* [https://github.com/sysmoon/vps-grpc-gateway](https://github.com/sysmoon/vps-grpc-gateway)
* [install python (/w pyenv)](https://www.liquidweb.com/kb/how-to-install-pyenv-on-ubuntu-18-04/)
* [grpc for python quick start](https://grpc.io/docs/languages/python/quickstart/)
* [grpc reverse proxy exmaple](https://medium.com/nirman-tech-blog/nginx-as-reverse-proxy-with-grpc-820d35642bff)
* [nginx api gateway for grpc](https://www.nginx.com/blog/deploying-nginx-plus-as-an-api-gateway-part-3-publishing-grpc-services/)
* [HTTP/JSON 을 gRPC로 트랜스코딩](https://cloud.google.com/endpoints/docs/grpc/transcoding)
* [gRPC Gateway for RestAPI](https://devjin-blog.com/golang-grpc-server-3/)
* [예제로 배우는 Go 프로그래밍](http://golang.site/go/article/2-Go-%EC%84%A4%EC%B9%98%EC%99%80-Go-%ED%8E%B8%EC%A7%91%EA%B8%B0-%EC%86%8C%EA%B0%9C)
* [Go언어 시작하기 - Golang Korean Community](https://golangkorea.github.io/post/go-start/getting-start/)
* [go mod를 이용한 패키지 관리 방법](https://lejewk.github.io/go-mod/)
* [golang - 설치와 GoPath](https://jacking75.github.io/go\_install/)
* [(번역) Go Modules 사용하기](https://johngrib.github.io/wiki/golang-mod/)
* [Context 이해하기](https://devjin-blog.com/golang-context/)
