# Localstack Sample SQS 

Requirements:
* Localstack running in K8s or localhost

## Running localhost 

```sh 
go run main.go -q="MyQueue"
```

## Build and Deploy K8s 

```sh

docker build . -t sqs:1.0

kind load docker-image sqs:1.0

aws --endpoint-url=http://localstack.localhost sqs list-queues

aws --endpoint-url=http://localstack.localhost sqs create-queue --queue-name MyQueue

aws --endpoint-url=http://localstack.localhost sqs list-queues

kubectl run --image=sqs:1.0 sqs-localstack --env="LOCALSTACK_URL=http://localstack:4566"  --image-pull-policy=Never

kubectl logs -f sqs-localstack

aws --endpoint-url=http://localstack.localhost sqs send-message --queue-url http://localstack.localhost/000000000000/MyQueue --message-body "Test SQS Localstack"

aws --endpoint-url=http://localstack.localhost sqs delete-queue --queue-url http://localstack.localhost/000000000000/MyQueue

```
