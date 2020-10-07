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
  --go_out=plugins=grpc:greeter-server/helloworld \
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
oc create configmap proto-descriptor --from-file=proto.pb=greeter-server/helloworld/helloworld.pb -n go
helm upgrade -i greeter-server helm/greeter-server -n go
```

```sh
oc get secret api-cert -n istio-system -o jsonpath='{.data.ca\.crt}' | base64 -d > /tmp/ca.crt

grpcui -protoset greeter-server/helloworld/helloworld.pb --cacert /tmp/ca.crt -service helloworld.Greeter api-istio-system.apps.cluster-946d.946d.sandbox1072.opentlc.com:443

grpcui -reflect-header 'service: helloworld.Greeter' --cacert /tmp/ca.crt -service helloworld.Greeter api-istio-system.apps.cluster-946d.946d.sandbox1072.opentlc.com:443

curl -v -X POST -H 'Content-Type: application/json' --http1.1 https://api-istio-system.apps.cluster-946d.946d.sandbox1072.opentlc.com/v1/greeter?name=trevor --cacert /tmp/ca.crt

curl -v -X POST -H 'Content-Type: application/json' -d '{"name":"Tobias Fünke"}' --http1.1 https://api-istio-system.apps.cluster-946d.946d.sandbox1072.opentlc.com/v1/greeter --cacert /tmp/ca.crt

istioctl proxy-config listeners $(oc get pod -l app=greeter-server -n go -o jsonpath='{.items[0].metadata.name}') -n go  -o json | less

istioctl pc log $(oc get pod -l app=greeter-server -n go -o jsonpath='{.items[0].metadata.name}') -n go --level debug
```
