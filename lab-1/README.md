# Lab 1 - Setup Istio

Now that we have a Kubernetes cluster and Meshery, we are ready to download and deploy Istio resources.

## Steps

- [1. Install Istio](#1)
- [2. Verify install](#2)
- [3. Confirm add-ons](#3)

Optional (manual install of Istio):

- [1. Download Istio resources](#1.1)
- [2. Setup `istioctl`](#1.2)

## <a name="1"></a> 1 - Install Istio

Using Meshery, select Istio from the `Management` menu.

<a href="img/istio-adapter.png">
<img src="img/istio-adapter.png" width="50%" align="center" />
</a>

In the Istio management page:

1. Type `istio-system` into the namespace field.
2. Click the (+) icon on the `Install` card and click on `Istio Service Mesh` to install latest version of Istio.

   <a href="img/install-istio1.png">
   <img src="img/install-istio1.png" width="50%" align="center" />
   </a>

3. Click the `Deploy` button on the confirmation modal.

   <a href="img/install-istio2.png">
   <img src="img/install-istio2.png" width="50%" align="center" />
   </a>

<h2>
  <a href="../lab-2/README.md">
  <img src="../img/go.svg" width="32" height="32" align="left" />
  Continue to Lab 2</a>: Deploy the sample application BookInfo
</h2>

<br />
<hr />

Alternative, manual installation steps are provided for reference below. No need to execute these if you have performed the steps above.

<hr />

## <a name="appendix"></a> Appendix - Alternative Manual Install

### <a name="1.1"></a> 1.1 - Download Istio

You will download and deploy the latest Istio resources on your Kubernetes cluster.

**_Note to Docker Desktop users:_** please ensure your Docker VM has atleast 4GiB of Memory, which is required for all services to run.

On your local machine:

```sh
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.7.3 sh -
```

### <a name="1.2"></a> 1.2 - Setting up istioctl

On a \*nix system, you can setup `istioctl` by doing the following:

```sh
brew install istioctl
```

Alternatively, change into the Istio package directory and add the `istioctl` client to your PATH environment variable.

```sh
cd istio-*
export PATH=$PWD/bin:$PATH
```

Verify `istioctl` is available:

```sh
istioctl version
```

Check if the cluster is ready for installation:

```sh
istioctl verify-install
```

### 1.3 Install Istio:

To install Istio with a `demo` profile, execute the below command.

```sh
istioctl install --set profile=demo
```

Alternatively, with Envoy logging enabled:

```sh
istioctl install --set profile=demo --set meshConfig.accessLogFile=/dev/stdout
```

## <a name="2"></a> 1.4 - Verify install

In the Istio management page:

1. Click the (+) icon on the `Validate Service Mesh Configuration` card.
1. Select `Verify Installation` to verify the installation of Istio.

#### Alternatively:

Istio is deployed in a separate Kubernetes namespace `istio-system`. To check if Istio is deployed, and also, to see all the pieces that are deployed, execute the following:

```sh
kubectl get all -n istio-system
```