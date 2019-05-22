---
title: "istio graph && metric"
date: 2019-05-13T12:47:06+08:00
---


## istio-metrics
The description of istio metrics

### Project Success Rate
* Expresstion: 
 `sum(rate(istio_requests_total{reporter="destination", destination_service_namespace=~"$namespace", response_code!~"5.*"}[1m])) / sum(rate(istio_requests_total{reporter="destination", destination_service_namespace=~"$namespace"}[1m]))`
* Description: the rate of requests that response code is non-5xx

### Project 4xx Request Count
* Expresstion: `sum(irate(istio_requests_total{reporter="destination", destination_service_namespace=~"$namespace", response_code=~"4.*"}[1m]))`
* Description: the rate of requests that response code is 4xx

### Project 5xx Request Count 
* Expresstion: `sum(irate(istio_requests_total{reporter="destination", destination_service_namespace=~"$namespace", response_code=~"5.*"}[1m]))`
* Description: the rate of requests that response code is 5xx

### 4xx Requset Count by Service
* Expresstion: `topk(10, sum(irate(istio_requests_total{reporter="destination", destination_service_namespace=~"$namespace", response_code=~"4.*"}[1m])) by (destination_service_name, destination_service_namespace))`
* Description: the top-10 services that response code is 4xx

### 5xx Request Count by Service
* Expresstion: `topk(10, sum(irate(istio_requests_total{reporter="destination", destination_service_namespace=~"$namespace",  response_code=~"5.*"}[1m])) by (destination_service_name, destination_service_namespace))`
* Description: the top-10 services that response code is 5xx

### Project Request Volume
* Expresstion: `round(sum(irate(istio_requests_total{reporter="destination", destination_service_namespace=~"$namespace"}[1m])), 0.001) `
* Description: the request volume of one project

### Request Volume by Service
* Expresstion: `topk(10, round(sum(irate(istio_requests_total{reporter="destination", destination_service_namespace=~"$namespace"}[1m])) by (destination_service_name, destination_service_namespace) , 0.001))`
* Description: the top-10 request volume of one project

### P99 Request Duration by Service
* Expresstion: `topk(10, histogram_quantile(0.99, sum(irate(istio_request_duration_seconds_bucket{reporter="destination", destination_service_namespace=~"$namespace"}[1m])) by (le, destination_service_name, destination_service_namespace)))`
* Description: the top-10 services that the request duration is in the quantile of 0.99

### P90 Request Duration by Service
* Expresstion: `topk(10, histogram_quantile(0.90, sum(irate(istio_request_duration_seconds_bucket{reporter="destination", destination_service_namespace=~"$namespace"}[1m])) by (le, destination_service_name, destination_service_namespace)))`
* Description: the top-10 services that the request duration is in the quantile of 0.90

### P50 Request Duration by Service
* Expresstion: `topk(10, histogram_quantile(0.50, sum(irate(istio_request_duration_seconds_bucket{reporter="destination", destination_service_namespace=~"$namespace"}[1m])) by (le, destination_service_name, destination_service_namespace)))`
* Description: the top-10 services that the request duration is in the quantile of 0.50


## The test steps

### Enable istio

* cluster -> Tools --> Istio --> `Enable Istio`

### Deploy applications

* Deploy bookinfo application as the [link](https://istio.io/docs/examples/bookinfo/#if-you-are-running-on-kubernetes).
* Deploy a application named `httpbin` in the default namespace to produce some 4xx or 5xx data, the yaml is:

```
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
    service: httpbin
spec:
  ports:
  - port: 80
    name: http
  selector:
    app: httpbin
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: httpbin
  labels:
    app: httpbin
    version: v1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      containers:
      - name: httpbin
        image: kennethreitz/httpbin:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
---
```

If you want to produce some 5xx data, then you can go into the container of `ratings-v1 ` workload to exec the following command:

```bash
curl httpbin/status/500
```
As the same as the above, you can use the following command to produce some 4xx data:

```bash
curl httpbin/status/400
```

## istio-graph

The istio graphs display the traffic direction of all services/workloads in one namespace. The `Green` line shows that the request is successful and the `Red` line indicates that the request is aborted.
