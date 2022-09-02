# Lab 2 - Deploy the sample application BookInfo

To play with Istio and demonstrate some of it's capabilities, you will deploy the example BookInfo application, which is included the Istio package.

## What is the BookInfo Application?

This application is a polyglot composition of microservices are written in different languages and sample BookInfo application displays information about a book, similar to a single catalog entry of an online book store. Displayed on the page is a description of the book, book details (ISBN, number of pages, and so on), and a few book reviews.

The end-to-end architecture of the application is shown in the figure.

<a href="img/bookinfo-off-mesh.png">
<img src="img/bookinfo-off-mesh.png" width="50%" align="center" />
</a>

_Figure: BookInfo deployed off the mesh_

It’s worth noting that these services have no dependencies on Istio, but make an interesting service mesh example, particularly because of the multitude of services, languages and versions for the reviews service.

As shown in the figure below, proxies are sidecarred to each of the application containers.

<a href="img/bookinfo-on-mesh.png">
<img src="img/bookinfo-on-mesh.png" width="50%" align="center" />
</a>

_Figure: BookInfo deployed on the mesh_

Sidecars proxy can be either manually or automatically injected into the pods. Automatic sidecar injection requires that your Kubernetes api-server supports `admissionregistration.k8s.io/v1` or `admissionregistration.k8s.io/v1beta1` or `admissionregistration.k8s.io/v1beta2` APIs. Verify whether your Kubernetes deployment supports these APIs by executing:

```sh
kubectl api-versions | grep admissionregistration
```

If your environment **does NOT** supports either of these two APIs, then you may use [manual sidecar injection](./appendix-manual-injection.md) to deploy the sample app.

As part of Istio deployment in [Lab 1](../lab-1/README.md), you have deployed the sidecar injector.

### <a name="auto"></a> Deploying Sample App with Automatic sidecar injection

Istio, deployed as part of this workshop, will also deploy the sidecar injector. Let us now verify sidecar injector deployment.

```sh
kubectl -n istio-system get configmaps istio-sidecar-injector
```

Output:

```sh
NAME                     DATA   AGE
istio-sidecar-injector   2      9h
```
#### Enable automatic sidecar injection
Using Meshery, navigate to the Istio management page. 
1. Enter `default` in the namespace field. 
2. Then click on `+` under `Apply Service Mesh Configuration` and click on `Automatic Sidecar injection`
<a href="img/sidecar-injection.png">
<img src="img/sidecar-injection.png" width="50%" align="center" />
</a>


NamespaceSelector decides whether to run the webhook on an object based on whether the namespace for that object matches the [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors).

```sh
kubectl get namespace -L istio-injection
```

Output:

```sh
NAME           STATUS    AGE       ISTIO-INJECTION
default        Active    1h        enabled
istio-system   Active    1h        disabled
kube-public    Active    1h
kube-system    Active    1h
```



Using Meshery, navigate to the Istio management page.

1. Enter `default` in the `Namespace` field.
1. Click the (+) icon on the `Sample Application` card and select `BookInfo Application` from the list.

This will do 2 things:

1. Deploys all the BookInfo services in the `default` namespace.
1. Deploys the virtual service and gateway needed to expose the BookInfo's productpage application in the `default` namespace.

<small>Manual step for can be found [here](#appendix)</small>

### <a name="verify"></a> Verify Bookinfo deployment

1. Verify that the deployments are all in a state of AVAILABLE before continuing.

   ```sh
   watch kubectl get deployment
   ```

2. Choose a service, for instance `productpage`, and view it's container configuration:

```sh
kubectl get po

kubectl describe pod productpage-v1-.....
```

3. Examine details of the services:

   ```sh
   kubectl describe svc productpage
   ```

Next, you will expose the BookInfo application to be accessed external from the cluster.

<h2>
  <a href="../lab-3/README.md">
  <img src="../img/go.svg" width="32" height="32" align="left" />
  Continue to Lab 3</a>: Access BookInfo via Istio Ingress Gateway
</h2>

<br />
<hr />

Alternative, manual installation steps are provided for reference below. No need to execute these if you have performed the steps above.

<hr />

## <a name="appendix"></a> Appendix - Alternative Manual Steps

### Label namespace for injection

Label the default namespace with istio-injection=enabled

```sh
kubectl label namespace default istio-injection=enabled
```

```sh
kubectl get namespace -L istio-injection
```

Output:

```sh
NAME           STATUS    AGE       ISTIO-INJECTION
default        Active    1h        enabled
istio-system   Active    1h        disabled
kube-public    Active    1h
kube-system    Active    1h
```

### Deploy BookInfo

Applying this yaml file included in the Istio package you collected in https://github.com/layer5io/istio-service-mesh-workshop/tree/master/lab-1#1 will deploy the BookInfo app in you cluster.

```sh
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

### Deploy Gateway and Virtual Service for BookInfo app

```sh
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```
