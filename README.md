# Demo: Crossplane ImageConfig

## Pre-Requisites

Create a Kubernetes cluster, e.g. with `kind`:

```sh
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane --set 'args={--enable-dependency-version-upgrades,--enable-signature-verification}'
```

## Pre

### Create secrets to pull private packages

```sh
kubectl -n crossplane-system create secret docker-registry upbound-platform-packages --docker-server=xpkg.upbound.io --docker-username=${REGISTRY_USR_PLAT} --docker-password=${REGISTRY_PW_PLAT}
```

### Install private package with private dependencies

```sh
kubectl apply -f pre/configuration-0.5.0.yaml
```

#### Things don't work

```sh
kubectl get configuration
```

> Even though we have a `packagePullSecret` it is not being used on the
dependencies...

## Post

### Install an ImageConfig

```sh
kubectl apply -f post/imageconfig.yaml
```

#### Yay it works

```sh
kubectl get configuration
```

### But wait there's more

Let's do image verification

#### Create secrets to pull other private packages

```sh
kubectl -n crossplane-system create secret docker-registry upbound-packages --docker-server=xpkg.upbound.io --docker-username=${REGISTRY_USR_LTS} --docker-password=${REGISTRY_PW_LTS}
```

#### Install an ImageConfig with verification

```sh
kubectl apply -f post/imageconfig-verification.yaml
```

#### Install a private provider

```sh
kubectl apply -f post/provider-s3.yaml
```

#### Check the image verification

```sh
kubectl get providerrevision -l pkg.crossplane.io/package=provider-aws-s3 -ojson | jq '.items[].status.conditions'
```

```yaml
- lastTransitionTime: "2024-11-07T04:17:52Z"
  message: Signature verification succeeded using ImageConfig named "upbound-packages"
  reason: SignatureVerificationSucceeded
  status: "True"
  type: Verified
```

### But wait there's more

Let's upgrade dependencies

```sh
kubectl apply -f post/configuration-0.6.0.yaml
```

```sh
kubectl get configuration
```

## Cleanup

```sh
kubectl delete configuration,provider,imageconfig --all && kubectl delete secrets upbound-platform-packages upbound-packages -n crossplane-system
```
