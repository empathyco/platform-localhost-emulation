# Localhost Stack

## Requirements

* Docker 


# K8s emulators

## [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

- Oldest option from K8s emulators
- Can specify kubernetes version 
- Runs as Docker or some VirtualBox 
- There are a bunch of [addons](https://minikube.sigs.k8s.io/docs/tasks/addons/) available. For instance, cert-manager, nginx-controller...
- Runs images from localhost

[![asciicast](https://asciinema.org/a/IlTAXF1DeLLNPhAJA9KFMUT7B.svg)](https://asciinema.org/a/IlTAXF1DeLLNPhAJA9KFMUT7B?t=0:21)


### Deploy a cluster
```sh 
brew install minikube
minikube start --driver=docker --memory 8196 --cpus=4 --kubernetes-version=v1.18.0
```

### Delete a cluster 
```sh
minikube delete
```

### Loading a local image in your cluster

This is an example of pulling and running in the minikube cluster a `nginx` image:

```sh
minikube start --driver=docker --memory 8196 --cpus=4 --kubernetes-version=v1.18.0
eval $(minikube docker-env)
docker ps
docker pull nginx
docker tag nginx test-nginx:1.0
kubectl run --image=test-nginx:1.0 nginx-app --port=80 --image-pull-policy=Never
kubectl port-forward po/nginx-app 8080:80
```
- Specify imagePullPolicy: IfNotPresent or imagePullPolicy: Never on your container(s).


## [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

- K8s cluster into Docker containers 
- Faster startup than Minikube
- Better documentation than Minikube
- Runs images from localhost (previous load image as K3d)
- Cluster setup as code


[![asciicast](https://asciinema.org/a/wJPD0vykaeK9YY9gZTHrOEWsn.svg)](https://asciinema.org/a/wJPD0vykaeK9YY9gZTHrOEWsn?t=1:08)

### Deploy a cluster

```sh 
brew install kind
kind create cluster --config=kind/cluster-conf.yaml
```

### Delete a cluster 
```sh
kind delete cluster
```

### Loading a local image in your cluster
This is an example of pulling and running in the Kind cluster a `nginx` image:
```sh
docker pull nginx:latest 
docker tag nginx:latest localhost-nginx:1.0
kind load localhost-nginx:1.0
kubectl run --image=localhost-nginx:1.0 localhost-nginx --port=80 --image-pull-policy=Never
kubectl get po
kubectl logs -f 
```
- Specify imagePullPolicy: IfNotPresent or imagePullPolicy: Never on your container(s).

### Ingress controller 

[![asciicast](https://asciinema.org/a/hOGrrufd4LyClt3kEOFah1biW.svg)](https://asciinema.org/a/hOGrrufd4LyClt3kEOFah1biW)

## [K3d](https://k3d.io/)

- Lightweight k3s image from Docker
- Runs on Docker
- Runs faster than anyone
- More than enough to gain knowledge in k8s 
- Runs images from localhost (previous load image as Kind)
- [https://github.com/iwilltry42/k3d-demo](https://github.com/iwilltry42/k3d-demo)


[![asciicast](https://asciinema.org/a/5ScyNfNirMGOY7a9eFoZUZztB.svg)](https://asciinema.org/a/5ScyNfNirMGOY7a9eFoZUZztB?t=0:55)

### Deploy a cluster

```sh 
brew install k3d
k3d cluster create mycluster
```

### Delete a cluster 

```sh
k3d cluster delete mycluster
```

### Loading a local image in your cluster

This is an example of pulling and running in the K3d cluster a `nginx` image:
```sh
docker pull nginx:latest 
docker tag nginx:latest localhost-nginx:1.0
k3d image import localhost-nginx:1.0 -c mycluster
kubectl run --image=localhost-nginx:1.0 localhost-nginx --port=80 --image-pull-policy=Never
```
- Specify imagePullPolicy: IfNotPresent or imagePullPolicy: Never on your container(s).





# Localstack


[![asciicast](https://asciinema.org/a/qwPP5j0I7woDOgjIMAdJO4B0I.svg)](https://asciinema.org/a/qwPP5j0I7woDOgjIMAdJO4B0I?t=0:33)

For those interested in try their services before going to AWS, they can deploy localstack in Minikube and create their AWS resources in localhost

The steps to deploy localstack in your Minikube are the following:

K8s yaml definition details
```yaml
image: localstack/localstack
```

To be able to access from k8s:
```
kubectl expose pod localstack --type=ClusterIP --name=localstack
```
If you want to access localstack from your localhost:
```
kubectl port-forward po/localstack 4566:4566 
```
Now, you can access your localstack pod. For instance, you can create a bucket:
* From localhost:
```sh
aws --endpoint-url=http://localhost:4566 s3api list-buckets
aws --endpoint-url=http://localhost:4566 s3api create-bucket --bucket test-localstack
aws --endpoint-url=http://localhost:4566 s3 cp test.sh s3://test-localstack
aws --endpoint-url=http://localhost:4566 s3 ls s3://test-localstack
```
* From k8s: 
```sh
aws --endpoint-url=http://localstack:4566 s3api list-buckets
aws --endpoint-url=http://localstack:4566 s3api create-bucket --bucket test-localstack
aws --endpoint-url=http://localstack:4566 s3 cp test.sh s3://test-localstack
aws --endpoint-url=http://localstack:4566 s3 ls s3://test-localstack
```

For those supporters of Helm, there are a Helm chart in this repo to deploy with the serverless services (iam, lambda, dynamodb, apigateway, s3, sns). More settings: https://github.com/localstack/localstack#configurations

```sh
cd localstack
helm install localstack . -f values.yaml
```


Happy Localstacking! 

Official Localstack documentation: https://github.com/localstack/localstack

## Localstack Samples

### Installation

```sh
helm install localstack -f ./../values.yaml
```

## SQS sample

Golang code to test receive message in a Localstack SQS 

[![asciicast](https://asciinema.org/a/7ENgU8pXiopZgpulKWirWk3Zs.svg)](https://asciinema.org/a/7ENgU8pXiopZgpulKWirWk3Zs)

```sh
aws --endpoint-url=http://localstack.localhost sqs list-queues

aws --endpoint-url=http://localstack.localhost sqs create-queue --queue-name MyQueue

aws --endpoint-url=http://localstack.localhost sqs send-message --queue-url http://localstack.localhost/000000000000/MyQueue --message-body "Information about the largest city in Any Region."

go run main.go -q="MyQueue"
```


# GCP Emulator 

GCP offers emulator for the following GCP services:
* BigTable
* Datastore
* Firestore
* PubSub
* Spanner

### GCP PubSub Emulator
 
GCP offers PubSub emulator. A Helm chart has been setup to deploy it. 

[![asciicast](https://asciinema.org/a/K896u54QFB6IDMWzrTzL0HA6C.svg)](https://asciinema.org/a/K896u54QFB6IDMWzrTzL0HA6C?t=0:35)


K8s yaml definition details
```yaml
image: google/cloud-sdk:latest
command: ["gcloud", "beta", "emulators", "pubsub", "start", "--host-port=0.0.0.0:8085", "--project=test"]
```


```sh
helm install gcp-emulator . -f values.yaml
cd gcp-emulator/samples/pubsub
go test -v .
```