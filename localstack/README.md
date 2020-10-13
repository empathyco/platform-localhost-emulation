# Localstack

<!-- vim-markdown-toc GFM -->

  - [Dependencies](#dependencies)
  - [Values](#values)
  - [Install chart](#install-chart)
  - [Monitoring/Dashboards](#monitoringdashboards)
  - [Alerting](#alerting)
  - [Service account](#service-account)

<!-- vim-markdown-toc -->

## Dependencies


## Values

In the file `values.yaml` there are set the most important parameter of the charts. 

The other file that should be changed is the [Chart.yaml](Chart.yaml), this file contains the description and the version that we will use in our "application"

```yaml
...
# this is the image version that we will use
appVersion: 1.0.0
...
```

## Install chart 

```sh
# this cmd will install/upgrade the chart 
helm upgrade --install localstack --namespace localstack_NAMESPACE -f values.yaml .
```

## Monitoring/Dashboards

To add a grafana dash just copy the file to [dashboards/](dashboards/)

## Alerting

To add alerts you only need to add the [prometheus alerts](https://prometheus.io/docs/alerting/overview/), just add a file with the name of the alerting group and the alert like the example below

```yaml
rules:
  - alert: "Empathy-{{ include "localstack.fullname" .}}-mem-usage"
    expr: jvm_memory_bytes_used{namespace="{{ $.Release.Namespace }}", service="{{ include "localstack.fullname" . }}", area="heap"}/jvm_memory_bytes_max{namespace="{{ $.Release.Namespace }}", area="heap"} > 80
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: "Service - {{`{{$labels.service}}`}} user too much mem"
      description: "Alarm - {{`{{$labels.service}}`}} reports that is using more than 80% of mem ($value) Please check it"
```

Please be aware that you need to change the 
```text
{{$labels.service}} for {{`{{$labels.service}}`}} if you want to user variables
```

## Service account

This service needs a service account to access [pubsub](https://cloud.google.com/pubsub/docs)

To create the service account

```sh
# create a new service account with a descriptive name
# just know that a service account name must be between 6 and 30 characters (inclusive), must begin with
# a lowercase letter, and consist of lowercase alphanumeric characters that can be separated by hyphens.

# add the appropriate roles to your service account
# for more info on roles, check https://cloud.google.com/iam/docs/understanding-roles
# for our app, we only need the Pub/Sub Publisher and Subscriber roles

# setting these up just to improve readability of the following commands
PROJECT_ID={{YOUR_PROJECT}}
SERVICE_ACCOUNT_NAME=localstack
TOPIC_NAME=localstack
$ gcloud iam service-accounts create ${SERVICE_ACCOUNT_NAME}
# add the Pub/Sub Publisher role
$ gcloud projects add-iam-policy-binding ${PROJECT_ID}\
    --member "serviceAccount:${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"\
    --role "roles/pubsub.publisher"

# add the Pub/Sub Subscriber role
$ gcloud projects add-iam-policy-binding ${PROJECT_ID}\
    --member "serviceAccount:${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"\
    --role "roles/pubsub.subscriber"

# download a key file containing your credentials
$ gcloud iam service-accounts keys create ${SERVICE_ACCOUNT_NAME}.json\
    --iam-account ${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com

# move the service account to the proper folder
$ mv ${SERVICE_ACCOUNT_NAME}.json serviceaccount/key.json

# Create topic for the service
gcloud pubsub topics create ${TOPIC_NAME}
```
