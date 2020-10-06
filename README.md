# grpc in service mesh

## Compile proto

<https://github.com/grpc/grpc-go#if-you-are-not-using-go-modules>

```sh
protoc \
  -I proto \
  -I proto/google/api \
  -I /usr/local/include \
  --descriptor_set_out=greeter-server/helloworld/helloworld.pb \
  --include_imports \
  --go_out=plugins=grpc:greeter-server/helloworld\
  --go_opt=paths=source_relative \
  proto/*.proto
```

## Setup

```sh
oc new-project istio-system
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

```sh
grpcui -proto helloworld/helloworld.proto --cacert /tmp/ca.crt -service helloworld.Greeter api-istio-system.apps.cluster-c718.c718.sandbox761.opentlc.com:443

curl -k -d '{"name": "asd"}' -H 'Content-Type: application/json' https://api-istio-system.apps.cluster-c718.c718.sandbox761.opentlc.com/v1/helloworld.Greeter/SayHello --cacert /tmp/ca.crt

istioctl proxy-config listeners $(oc get pod -l app=example -n go -o jsonpath='{.items[0].metadata.name}') -n go  -o json | less

istioctl pc log $(oc get pod -l app=example -n go -o jsonpath='{.items[0].metadata.name}') -n go --level debug

```
