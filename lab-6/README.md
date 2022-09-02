# Lab 6 - Fault Injection

In this lab we will learn how to test the resiliency of an application by injecting systematic faults.

<!-- Before we start let us reset the route rules: -->

<!-- ```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml 
``` -->
Before we start, we will need to reset the virtual services.

Using Meshery, navigate to the Istio management page:
Undeploy `alltraffic-to-v3.design.yaml` that you had imported in the previous lab from design configurator

<!-- 
```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
``` -->

## 6.1 Inject a route rule to create a fault using HTTP delay

To start, we will inject a 7s delay for accessing the ratings service for a user `jason`. reviews v2 service has a 10s hard-coded connection timeout for its calls to the ratings service configured globally.

<!-- ```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
``` -->

Using Meshery, navigate to the designs page under configuration and import the below design. Make sure Istio adapter is running.
**Patternfile**:
```yaml
name: FaultForJason
services:
  vs:
    settings:
      hosts:
      - ratings
      http:
      - fault:
          delay:
            fixedDelay: 7s
            percentage:
              value: 100
        match:
        - headers:
            end-user:
              exact: jason
        route:
        - destination:
            host: ratings
            subset: v1
      - route:
        - destination:
            host: ratings
            subset: v1
    type: VirtualService.Istio
    name: vs
    namespace: default

```

<small>Manual step for can be found [here](#appendix)</small>

This will update the existing virtual service definition for ratings to inject a delay for user `jason` to access the ratings V1.

In a few, we should be able to verify the virtual service by using the command below:
```sh
kubectl get virtualservice ratings -o yaml
```

Now we login to `/productpage` as user `jason` and observe that the page loads but because of the induced delay between services the reviews section will show :

<pre>
        <b>Error fetching product reviews!</b>

Sorry, product reviews are currently unavailable for this book.
</pre>

If you logout or login as a different user, the page should load normally without any errors.

## 6.2 Inject a route rule to create a fault using HTTP abort

In this section, , we will introduce an HTTP abort to the ratings microservices for user `jason`.

<!-- Now apply the change to the cluster:
```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
``` -->


Using Meshery, navigate to the designs page under configuration and import the below design. Make sure Istio adapter is running.
**Patternfile**:
```yaml
name: FaultForJason
services:
  vs:
    settings:
      hosts:
      - ratings
      http:
      - fault:
          delay:
            fixedDelay: 7s
            percentage:
              value: 100
        match:
        - headers:
            end-user:
              exact: jason
        route:
        - destination:
            host: ratings
            subset: v1
      - route:
        - destination:
            host: ratings
            subset: v1
    type: VirtualService.Istio
    name: vs
    namespace: default
```

This will update the existing virtual service definition for ratings to inject a HTTP abort for user `jason` to access the ratings V1.

In a few, we should be able to verify the virtual service by using the command below:

```sh
kubectl get virtualservice ratings -o yaml
```


Now we login to `/productpage` as user `jason` and observe that the page loads without any new delays but because of the induced fault between services the reviews section will show:

 `Ratings service is currently unavailable`.

### 6.3 Verify fault injection
Verify the fault injection by logging out (or logging in as a different user), the page should load normally without any errors.

## [Continue to Lab 7 - Circuit Breaking](../lab-7/README.md)

<hr />
Alternative, manual installation steps below. No need to execute, if you have performed the steps above.
<hr />

## <a name="appendix"></a> Appendix

### Route all traffic to version V1 of all services

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml 
```

### Route all traffic to version V2 of reviews for user Jason

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

### Inject 7s delay for ratings service

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
```

### Inject HTTP abort for ratings service
```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
```


