# GCP PubSub emulator sample

Sample to show how to create a topic and subscription using PubSub emulator as well as receive messages

```sh
helm install gcp-emulator -f values.yaml
kubectl port-forward svc/gcp-emulator 8080:8085
go test -v .
```
