# 클라우드 저장소



### **Model Repository Locations**

Triton은 여러 파일 시스템을 통해 모델 저장소로 활용할 수 있습니다.

* Local File System
* Google Cloud Storage
* Azure Blob Storage
* AWS S3

****

**Local File System**

모델 저장소로 테스트하기 가장 쉬운 방법은 로컬 파일 시스템 절대경로 지정을 통해 모델 저장소로 사용하는 방법입니다.

```
$ tritonserver --model-repository=/path/to/model/repository ...
```

Quick Start 예제에서의 같이 샘플 모델 저장소 경로명을 Local File System 경로명으로 이용하고, 도커 컨테이너 실행시 볼륨 마운트하여 Trtion 서버 실행이 가능합니다.

```
docker run --gpus=1 --rm -p8000:8000 -p8001:8001 -p8002:8002 -v/home/vpsdev/server/docs/examples/model_repository:/models nvcr.io/nvidia/tritonserver:22.02-py3 tritonserver --model-repository=/models
```



Local File System 뿐만 아니라 Public Cloud (GCP, AWS, Azure) 에서 제공하는 Storage 서비스를 이용하여 모델 저장소로 활용 가능합니다.



**Google Cloud Storage**

google cloud storage 에 있는 모델 repository는 gs:// 를 앞에 작성해야 한다.

```
$ tritonserver --model-repository=gs://bucket/path/to/model/repository ...
```

**S3**

For a model repository residing in Amazon S3, the path must be prefixed with s3://.

```
$ tritonserver --model-repository=s3://bucket/path/to/model/repository ...
```

For a local or private instance of S3, the prefix s3:// must be followed by the host and port (separated by a semicolon) and subsequently the bucket path.

```
$ tritonserver --model-repository=s3://host:port/bucket/path/to/model/repository ...
```

By default, Triton uses HTTP to communicate with your instance of S3. If your instance of S3 supports HTTPS and you wish for Triton to use the HTTPS protocol to communicate with it, you can specify the same in the model repository path by prefixing the host name with https://.

```
$ tritonserver --model-repository=s3://https://host:port/bucket/path/to/model/repository ...
```

When using S3, the credentials and default region can be passed by using either the [aws config](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) command or via the respective [environment variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html). If the environment variables are set they will take a higher priority and will be used by Triton instead of the credentials set using the aws config command.

#### Azure

Azure Blob Storage 를 모델 저장소로 활용하기 위해 Storage Account 를 azcli 를 통해 생성하고, Triton 샘플 모델 저장소를 저장합니다. 이를 위해 다음과 같은 사전 준비사항이 필요합니다.

* azcopy 설치

```
brew install azcopy
```

* Storage Account SAS 토큰 생성

```
az storage container generate-sas --account-name $YOUR_STORAGE_ACCOUNT --name $YOUR_CONTAINER_NAME --permissions acdlrw --expiry '2022-03-20' --auth-mode login --as-user
```

* azcopy 도구를 이용하여 모델 복사

azcopy 명령어를 이용하여 모델 파일을 Azure Blob Storage 에 저장합니다. 이때 사용되는 Storage Account,  Container, SAS 토큰을 이용하여 URL 경로명을 아래와 같이 복사합니다.

```
azcopy copy {YOUR_SRC_PATH} https://{YOUR_STORAGE_ACCOUNT}.blob.core.windows.net/{YOUR_CONTAINER}\$YOUR_SAS_TOKEN --recursive
```



Azure Blob Storage 를 모델 저장소로 활용하기 위해 모델 저장소 경로명은 반드시 as:// 형식으로 고정 사용해야 합니다.

```
$ tritonserver --model-repository=as://account_name/container_name/path/to/model/repository ...
```

Azure Blob Storage 를 사용하여 Triton 을 실행할 경우 반드시 Azure Storage Account 접근을 위한 리소스 이름과 보안키가 필요합니다. Azure Portal 사이트를 통해 확인하거나 아래 스크립트와 같이 azcli 를 통해 확인 가능합니다. 이후 해당 값을 Triton 서버 도커 컨테이너 실행시 환경 변수 `AZURE_STORAGE_ACCOUNT, AZURE_STORAGE_KEY 입력값으로 활용합니다.`

```
#!/bin/bash

# AZ Storage Account 접속 정보
export AZURE_STORAGE_ACCOUNT="YourStorageAccountName"
export AZURE_STORAGE_KEY=$(az storage account keys list -n $AZURE_STORAGE_ACCOUNT --query "[0].value" | tr -d '"')

# 접속 정보 확인 
echo $AZURE_STORAGE_ACCOUNT
echo $AZURE_STORAGE_KEY

# AZ Storage Account 를 모델 저장소로 활용하여 Triton Server 실행
docker run -e AZURE_STORAGE_ACCOUNT=$AZURE_STORAGE_ACCOUNT -e AZURE_STORAGE_KEY=$AZURE_STORAGE_KEY --gpus=1 --rm -p8000:8000 -p8001:8001 -p8002:8002 nvcr.io/nvidia/tritonserver:22.02-py3 tritonserver --model-repository=as://vlammodel/testmodelrepo/
```

이후 Triton 서버가 정상적으로 실행되어 모델 저장소에 있는 각 모듈이 정상적으로 로딩된 것을 확인할 수 있습니다.

```
I0317 07:53:38.713928 1 server.cc:592] 
+----------------------+---------+--------+
| Model                | Version | Status |
+----------------------+---------+--------+
| densenet_onnx        | 1       | READY  |
| inception_graphdef   | 1       | READY  |
| simple               | 1       | READY  |
| simple_dyna_sequence | 1       | READY  |
| simple_identity      | 1       | READY  |
| simple_int8          | 1       | READY  |
| simple_sequence      | 1       | READY  |
| simple_string        | 1       | READY  |
+----------------------+---------+--------+

I0317 07:53:38.732299 1 metrics.cc:623] Collecting metrics for GPU 0: Tesla T4
I0317 07:53:38.732658 1 tritonserver.cc:1932] 
+----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Option                           | Value                                                                                                                                                                                        |
+----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| server_id                        | triton                                                                                                                                                                                       |
| server_version                   | 2.19.0                                                                                                                                                                                       |
| server_extensions                | classification sequence model_repository model_repository(unload_dependents) schedule_policy model_configuration system_shared_memory cuda_shared_memory binary_tensor_data statistics trace |
| model_repository_path[0]         | as://vlammodel/testmodelrepo/                                                                                                                                                                |
| model_control_mode               | MODE_NONE                                                                                                                                                                                    |
| strict_model_config              | 1                                                                                                                                                                                            |
| rate_limit                       | OFF                                                                                                                                                                                          |
| pinned_memory_pool_byte_size     | 268435456                                                                                                                                                                                    |
| cuda_memory_pool_byte_size{0}    | 67108864                                                                                                                                                                                     |
| response_cache_byte_size         | 0                                                                                                                                                                                            |
| min_supported_compute_capability | 6.0                                                                                                                                                                                          |
| strict_readiness                 | 1                                                                                                                                                                                            |
| exit_timeout                     | 30                                                                                                                                                                                           |
+----------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

I0317 07:53:38.787592 1 grpc_server.cc:4375] Started GRPCInferenceService at 0.0.0.0:8001
I0317 07:53:38.787793 1 http_server.cc:3075] Started HTTPService at 0.0.0.0:8000
I0317 07:53:38.832298 1 http_server.cc:178] Started Metrics Service at 0.0.0.0:8002
```
