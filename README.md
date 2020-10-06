# grpc in service mesh

<https://github.com/grpc/grpc-go#if-you-are-not-using-go-modules>

```sh
protoc \
  -I proto \
  -I proto/google/api \
  -I /usr/local/include \
  --descriptor_set_out=github.com/trevorbox/grpc/examples/helloworld/helloworld/helloworld.pb \
  --include_imports \
  --go_out=plugins=grpc:. \
  proto/*.proto


protoc --go_out=plugins=grpc:. *.proto

protoc -I google/api -I . -I $GOPATH/src -I $GOPATH/src/google.golang.org/protobuf/proto --go_out=plugins=grpc:. *.proto

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
