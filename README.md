# grpc in service mesh

## How to compile greeter-server proto

<https://github.com/grpc/grpc-go#if-you-are-not-using-go-modules>

```sh
protoc \
  -I greeter-server/proto \
  -I greeter-server/proto/google/api \
  -I /usr/local/include \
  --descriptor_set_out=greeter-server/helloworld/helloworld.pb \
  --include_imports \
  --go_out=plugins=grpc:greeter-server/helloworld\
  --go_opt=paths=source_relative \
  greeter-server/proto/*.proto
```

## How to run greeter-server locally

```sh
go get github.com/trevorbox/service-mesh-grpc/greeter-server/helloworld
go run greeter-server/main.go
```

## Setup

```sh
oc new-project istio-system
oc new-project go
```

## Deploy cert-manager (skip if already present in the cluster)

```shell
oc new-project cert-manager
oc apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.0/cert-manager.yaml
```

## Install control plane

```sh
helm upgrade -i control-plane helm/control-plane -n istio-system
```

## Install greeter-server

```sh
helm upgrade -i greeter-server helm/greeter-server -n go
```

```sh
grpcui -proto helloworld/helloworld.proto --cacert /tmp/ca.crt -service helloworld.Greeter api-istio-system.apps.cluster-c718.c718.sandbox761.opentlc.com:443

curl -k -d '{"name": "asd"}' -H 'Content-Type: application/json' https://api-istio-system.apps.cluster-c718.c718.sandbox761.opentlc.com/v1/helloworld.Greeter/SayHello --cacert /tmp/ca.crt

istioctl proxy-config listeners $(oc get pod -l app=example -n go -o jsonpath='{.items[0].metadata.name}') -n go  -o json | less

istioctl pc log $(oc get pod -l app=example -n go -o jsonpath='{.items[0].metadata.name}') -n go --level debug

```
