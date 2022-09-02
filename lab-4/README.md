# Lab 4 - Observability

## <a name="1"></a>4.1 Install Telemetry Add-ons

Using Meshery, install Istio telemetry add-ons. In the Istio management page:

1. Toggle each of the following add-ons:
   1. [Prometheus](https://prometheus.io/)
   1. [Grafana](https://grafana.com/)
   1. [Jaeger](https://www.jaegertracing.io/)

<a href="img/istio-addons.png">
<img src="img/istio-addons.png" width="50%" align="center" />
</a>

You will use Prometheus and Grafana for collecting and viewing metrics and [Jaeger](https://www.jaegertracing.io/) collecting and viewing distributed traces. Expose each add-on external to the cluster. Each the service network typs are set to "LoadBalancer".

**Question: Why can't you expose these add-on components through Istio Ingress Gateway?**

### <a name="2"></a>4.2 Service Mesh Performance and Telemetry

Many of the labs require load to be placed on the sample apps. Let's generate HTTP traffic against the BookInfo application, so we can see interesting telemetry. 

Verify access through the Ingress Gateway:

```sh
kubectl get service istio-ingressgateway -n istio-system
```

Once we have the port, we can append the IP of one of the nodes to get the host. 

The URL to run a load test against will be `http://<IP/hostname of any of the nodes in the cluster>:<ingress port>/productpage`

__Please note:__ If you are using Docker Desktop, please use the IP address of your host. You can leave the port blank. For example: `http://1.2.3.4/productpage`

Use the computed URL above in Meshery, in the browser, to run a load test and see the results.

#### 2.1 Connect Grafana (optionally, Prometheus) to Meshery.

On the [Settings page](http://localhost:9081/settings#metrics):

1. Navigate to the `Metrics` tab.
1. Enter Grafana's URL:port number and submit.

#### 2.2 Use Meshery to generate load and analyze performance.

On the [Performance page](http://localhost:9081/performance):

1. give this load test a memorable name
1. enter the URL to the BookInfo productpage
1. select `Istio` in the `Service Mesh` dropdown
1. enter a valid number for `Concurrent requests`
1. enter a valid number for `Queries per second`
1. enter a valid `Duration` (a number followed by `s` for seconds (OR) `m` for minutes (OR) `h` for hour)
1. use the host IP address in the request Tab and in the advanced options, type in the header as `Host:<app-name>`

Click on `Run Test`. A performance test will run and statistical analysis performed. Examine the results of the test and behavior of the service mesh.

Next, you will begin controlling requests to BookInfo using traffic management features.

<h2>
  <a href="../lab-5/README.md">
  <img src="../img/go.svg" width="32" height="32" align="left" />
  Continue to Lab 5</a>: Traffic Management
</h2>

<br />
<hr />

Alternative, manual installation steps are provided for reference below. No need to execute these if you have performed the steps above.

<hr />

## <a name="appendix"></a> Appendix - Alternative Manual Steps

### 4.1 Install Add-ons:

**Prometheus**

```sh
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/prometheus.yaml

```

**Grafana**

```sh
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/grafana.yaml

```

**Jaeger**

```sh
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.7/samples/addons/jaeger.yaml

```

### Exposing services

Istio add-on services are deployed by default as `ClusterIP` type services. We can expose the services outside the cluster by either changing the Kubernetes service type to `NodePort` or `LoadBalancer` or by port-forwarding or by configuring Kubernetes Ingress.

**Option 1: Expose services with NodePort**
To expose them using NodePort service type, we can edit the services and change the service type from `ClusterIP` to `NodePort`

**Option 2: Expose services with port-forwarding**
Port-forwarding runs in the foreground. We have appeneded `&` to the end of the above 2 commands to run them in the background. If you donot want this behavior, please remove the `&` from the end.

## 4.2 Prometheus

You will need to expose the Prometheus service on a port either of the two following methods:

**Option 1: Expose services with NodePort**

```sh
kubectl -n istio-system edit svc prometheus
```

To find the assigned ports for Prometheus:

```sh
kubectl -n istio-system get svc prometheus
```

**Option 2: Expose Prometheus service with port-forwarding:**
\*\*
Expose Prometheus service with port-forwarding:

```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') \
  9090:9090 &
```

Browse to `http://<ip>:<port>` and in the `Expression` input box enter: `istio_request_bytes_count`. Click the Execute button.

![](img/Prometheus.png)

## 4.3 Grafana

You will need to expose the Grafana service on a port either of the two following methods:

```sh
kubectl -n istio-system edit svc grafana
```

Once this is done the services will be assigned dedicated ports on the hosts.

To find the assigned ports for Grafana:

```sh
kubectl -n istio-system get svc grafana
```

**Expose Grafana service with port-forwarding:**

```sh
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana \
  -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
```

![](img/Grafana_Istio_Dashboard.png)

<!--
## 4.4 Kiali

**Option 1: Expose services with NodePort**

```sh
kubectl -n istio-system edit svc kiali
```

To find the assigned ports for Servicegraph:
```sh
kubectl -n istio-system get svc kiali
```

**Option 2: Expose Kiali service with port-forwarding:**

```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') \
  20001:20001 &
```
Update the URI to `/kiali` and you will be presented with a login screen. Please use `admin` for both user name and password. After you login, you can navigate to the different sections using the menu on the left.

![](https://istio.io/docs/tasks/telemetry/kiali/kiali-graph.png)

## 4.5 - Distributed Tracing
-->

## 4.4 - Distributed Tracing

The sample Bookinfo application is configured to collect trace spans using Zipkin or Jaeger. Although Istio proxies are able to automatically send spans, it needs help from the application to tie together the entire trace. To do this applications need to propagate the appropriate HTTP headers so that when the proxies send span information to Zipkin or Jaeger, the spans can be correlated correctly into a single trace.

To do this the application collects and propagates the following headers from the incoming request to any outgoing requests:

- `x-request-id`
- `x-b3-traceid`
- `x-b3-spanid`
- `x-b3-parentspanid`
- `x-b3-sampled`
- `x-b3-flags`
- `x-ot-span-context`

<a href="img/jaeger.png">
<img src="img/jaeger.png" width="50%" align="center" />
</a>

### Exposing services

Istio add-on services are deployed by default as `ClusterIP` type services. We can expose the services outside the cluster by either changing the Kubernetes service type to `NodePort` or `LoadBalancer` or by port-forwarding or by configuring Kubernetes Ingress. In this lab, we will briefly demonstrate the `NodePort` and port-forwarding ways of exposing services.

#### Option 1: Expose services with NodePort

To expose them using NodePort service type, we can edit the services and change the service type from `ClusterIP` to `NodePort`

For Jaeger, either of `tracing` or `jaeger-query` can be exposed.

```sh
kubectl -n istio-system edit svc tracing
```

Once this is done the services will be assigned dedicated ports on the hosts.

To find the assigned ports for Jaeger:

```sh
kubectl -n istio-system get svc tracing
```

#### Option 2: Expose services with port-forwarding

To port-forward Jaeger:

```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') \
  16686:16686 &
```

<!-- ### 4.5.1 View Traces -->

### 4.4.1 View Traces

Let us find the port Jaeger is exposed on by running the following command:

```sh
kubectl -n istio-system get svc tracing
```

You can click on the link at the top of the page which maps to the right port and it will open Jaeger UI in a new tab.

