# vault-benchmark

- `vault-benchmark` 는 Vault 인증 방법과 비밀 엔진의 성능을 테스트하기 위해 고안된 도구입니다.
- [Release Download](https://releases.hashicorp.com/vault-benchmark)

## Vault 구성 설정

- `vault_addr`: 볼트 주소, VAULT_ADDR 재정의
- `cluster_json`: cluster.json 파일의 경로
- `vault_token`: 볼트 토큰, VAULT_TOKEN 재정의
- `audit_path`: 볼트 클러스터 생성 시, Audit 로그 파일 경로
- `ca_pem_file`: HTTPS로 외부 볼트를 사용하는 경우, 해당 CA 파일의 경로를 PEM 형식으로 지정합니다.

이외의 주요 옵션은 다음과 같습니다:

- `workers`:  가상 사용자 (작업자 수)
- `duration`:  벤치마크 시간
- `rps`: 초당 요청 수
- `report_mode` : 리포팅 모드 - terse, verbose, json
- `pprof_interval` : 볼트 디버그 프로페서 프로파일링 수집 간격
- `input_results` : 테스트를 실행하는 대신 이전 테스트 실행에서 JSON 파일을 읽음
- `annotate` :  콤마로 구분된 name=value 페어로 bench_running 프로메테우스 매트릭에 포함됨
- `debug` : 테스트를 실행하기 전에 각 벤치마크 대상을 실행하고 요청/응답 정보를 출력

볼트 벤치마크는 '작업자'라는 가상 사용자를 생성하여 지속적으로가상 사용자를 생성합니다.  
생성할 요청은 테스트 옵션에 의해 제어되며, 합이 100이어야 합니다.
또한 해당 테스트 지정 섹션에서 찾을 수 있는 테스트별 옵션도 있습니다.

## Build Project

Golang 과 Git 을 설치합니다.
Go 모듈을 사용하면 외부 모듈을 로컬로 다운로드 받게 되는데 이때 git 프로젝트를 프록시를 통해 다운로드 받게 됩니다.
그래서 git 을 미리 설치해 두어야 합니다.

```bash
$ wget https://go.dev/dl/go1.18.2.linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz
$ vi ~/.profile
...
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
$ source ~/.profile
$ mkdir ~/go

$ sudo apt-get update
$ sudo apt-get install git
$ git version

$ cd ~/go
$ git clone https://github.com/daeung0921/vault-benchmark.git
$ cd vault-benchmark
$ go build
$ sudo mv vault-benchmark /usr/bin
```


## Vault cluster

벤치마크_볼트에는 `vault_addr` 및 `vault_token` 인자 또는 환경 변수 `VAULT_ADDR` 및 `VAULT_TOKEN`을 설정해야 합니다.
Vault 클러스터에는 미리 설정된 마운트가  필요하지 않습니다. 
테스트 준비의 일환으로 무작위로 설정된 마운트 포인트를 무작위로 설정합니다.  
따라서 설정한 볼트 토큰에 충분한 권한이 있어야 합니다. 

# Examples

``` 
$ cat > config.hcl << EOF
vault_addr = "https://vault.idtplateer.com:8200"
vault_token = "hvs.0uD38NBCctHe36BzeVFxGqBJ"
ca_pem_file = "/home/ec2-user/vault_ca.pem"
duration = "30s"
cleanup = true
debug = true
report_mode = "terse"
random_mounts = true
nvalid = "option"
workers = 10

test "kvv2_read" "static_secret_writes" {
  weight = 75
  config {
    numkvs = 100
    kvsize = 10
  }
}

test "kvv2_write" "static_secret_writes" {
  weight = 25
  config {
    numkvs = 100
    kvsize = 10
  }
}
EOF

$ vault-benchmark run -config=config.hcl
```

# Test Cases

## Auth Methods

- [Approle Configurations](/docs/tests/auth-approle.md)
- [Certificate Configurations](/docs/tests/auth-certificate.md)
- [LDAP Configurations](/docs/tests/auth-ldap.md)
- [Userpass Configurations](/docs/tests/auth-userpass.md)

## Secret Engines

- [CassandraDB Configurations](/docs/tests/secret-cassandra.md)
- [Consul Configurations](/docs/tests/secret-consul.md)
- [Couchbase Configurations](/docs/tests/secret-couchbase.md)
- [Elasticsearch Configurations](/docs/tests/elasticsearch.md)
- [KV Configurations](/docs/tests/secret-kv.md)
- [LDAP Configurations](/docs/tests/secret-ldap.md)
- [MongoDB Configurations](/docs/tests/secret-mongo.md)
- [PKI Configurations](/docs/tests/secret-pki.md)
- [PostgreSQL Configurations](/docs/tests/secret-postgresql.md)
- [RabbitMQ Configurations](/docs/tests/secret-rabbit.md)
- [Redis Configurations](/docs/tests/secret-redis.md)
- [SSH Key Signing](/docs/tests/secret-ssh-sign.md)
- [Signed SSH Certificates Configurations](/docs/tests/secret-ssh-sign-ca.md)
- [Transit Configurations](/docs/tests/secret-transit.md)

## System Status

- [System Status Configurations](/docs/tests/system-status.md)

# Outputs

## Reports

테스트가 완료되면 stdout에 보고서가 생성됩니다.  보고서의 형식은 다음과 같은 형식을 가질 수 있습니다:

- `terse`: 테스트 유형당 한 줄의 메트릭 블록
- `verbose`: 테스트 유형당 한 줄의 메트릭 블록(여러 줄)
- `json`: 전체 메트릭을 포함하는 JSON 블록

## Troubleshooting

'100%'를 예상했을 때 '성공률: 0.00%'와 같은 예기치 않은 결과가 표시되는 경우,
디버그 플래그 `-debug=true`를 사용하여 잠재적인 문제에 대한 자세한 정보를 얻으세요.

## Profiling

'pprof_interval'은 'vault debug' 명령을 실행하여 pprof 데이터를 수집합니다.
는 'vault-debug-X'라는 폴더에 기록되며, 여기서 X는 타임스탬프입니다.
 
