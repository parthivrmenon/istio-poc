# Istio POC

## Pre-requisites
- [Install](https://minikube.sigs.k8s.io/docs/start/) `minikube` 
- [Install](https://helm.sh/docs/intro/install/) Helm


## Setup
- [Prepare](#prepare-minikube-cluster) `minikube` cluster
- [Install](#install-istio) Istio
- [Install](#install-istioctl) Istioctl
- Setup [whoami](#deploy-the-whoami-app) app
- Enable [Istio-Injection](#enable-isitio-injection)
- [Deploy](#deploy-the-whoami-app) `whoami` application
- [Expose](#expose-whoami-app) whoami app via Gateway
- [Verify](#verify-configuration-via-istioctl) configuration
- [Test](#test-access) access to `whoami`


## Prepare Minikube cluster
```bash
# Step 1: Verify minikube is installed
minikube status

# (Optional) Step 2: Purge existing minikube configuration
minikube delete

# Step 3: Start a new minikube
minikube start

# Step 4: Create NS
kubectl create namespace istio-system
kubectl create namespace istio-ingress
kubectl create namespace whoami
```

## Install Istio
```bash
# Add Helm Istio repo
helm repo add istio https://istio-release.storage.googleapis.com/charts

# Pull latest charts
helm repo update

# Install istio components onto the istio-system NS
helm install istio-base istio/base -n istio-system --set defaultRevision=default

helm install istiod istio/istiod -n istio-system --wait

helm install istio-ingressgateway istio/gateway -n istio-ingress

# Verify installation
helm status istio-base -n istio-system
helm status istiod -n istio-system
helm status istio-ingressgateway  -n istio-ingress

```

## Install istioctl
```bash
curl -sL https://istio.io/downloadIstioctl | sh -
export PATH=$HOME/.istioctl/bin:$PATH
```

## Enable isitio injection
```
kubectl label namespace default istio-injection=enabled
kubectl label namespace whoami istio-injection=enabled
```

## Deploy the [whoami](https://github.com/parthivrmenon/whoami) app
```bash
kubectl apply -f whoami/deployment.yaml  -f whoami/service.yaml
```

## Expose `whoami` app
```bash
kubectl apply -f traffic_management/whoami_gateway.yaml -f traffic_management/whoami_vs.yaml -f other/telemetry.yaml

```

## Apply authn & authz policies
```bash
# authz
kubectl apply -f authz/allow_public.yaml -f authz/allow_root.yaml -f authz/deny_private.yaml

# view authz policies
# istioctl experimental authz check <pod.namespace>

# authn
kubectl apply -f authn/jwt.yaml

```


## Verify configuration via Istioctl
```bash
istioctl analyze
istioctl analyze -n whoami
```

## Test access
```bash
# run on different terminal
minikube tunnel

curl http://127.0.0.1/whoami

# Tail Envoy access logs
kubectl -n whoami logs -l app=k8s-whoami -c istio-proxy -f


```

