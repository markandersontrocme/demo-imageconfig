# Demo: Crossplane ImageConfig

## Pre

### Connect to a control plane

#### Upbound Cloud

```sh
up ctx kubecon-loves-xp/upbound-gcp-us-central-1/default/hacky-mchack-face
```

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

#### Patch Crossplane service account

```sh
kubectl edit serviceaccount crossplane -n crossplane-system
```

```yaml
imagePullSecrets:
  - name: package-pull-secret
  - name: upbound-platform-packages
```

#### Yay it works

```sh
kubectl get configuration
```

> If we upgrade the Crossplane release we will need to re-patch the service
account. We can pass the secret name as a value to the Crossplane Helm chart
if we wanted to.

### Upgrade our configuration

```sh
kubectl apply -f pre/configuration-0.6.0.yaml
```

```yaml
message: 'cannot resolve package dependencies: incompatible dependencies: existing
package xpkg.upbound.io/upbound-platform/platform-composite-ubc-gke@v2.5.0 is
incompatible with constraint v2.6.0'
```

> We would need to manually edit the configuration to get the new version, but
enough of this, let's move to how things are now.

## Post

### Connect to a control plane

```sh
up ctx kubecon-loves-xp/upbound-gcp-us-central-1/default/fancy-pants
```

### Create secrets to pull private packages

```sh
kubectl -n crossplane-system create secret docker-registry upbound-platform-packages --docker-server=xpkg.upbound.io --docker-username=${REGISTRY_USR_PLAT} --docker-password=${REGISTRY_PW_PLAT}
```

### Install an ImageConfig

```sh
kubectl apply -f post/imageconfig.yaml
```

### Install private package with private dependencies

```sh
kubectl apply -f post/configuration-0.5.0.yaml
```

#### Yay it works

```sh
kubectl get configuration
```

### But wait there's more

Let's do image verification

#### Create secrets to pull other private packages

```sh
kubectl -n crossplane-system create secret docker-registry upbound-lts-packages --docker-server=xpkg.upbound.io --docker-username=${REGISTRY_USR_LTS} --docker-password=${REGISTRY_PW_LTS}
```

#### Install an ImageConfig with verification

```sh
kubectl apply -f post/imageconfig-verification.yaml
```

#### Install a private provider

```sh
kubectl apply -f post/provider-lts.yaml
```

#### Check the image verification

```sh
kubectl get providerrevision
```

```yaml
- lastTransitionTime: "2024-11-07T04:17:52Z"
  message: Signature verification succeeded using ImageConfig named "upbound-lts-packages"
  reason: SignatureVerificationSucceeded
  status: "True"
  type: Verified
```

### But wait there's more

Let's upgrade dependencies

```sh
kubectl apply -f post/configuration-0.6.0.yaml
```
