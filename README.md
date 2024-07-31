# Crossplane Compositions

Use crossplane compositions to create a custom CRD that hides platform complexities from Dev teams.


## Setup

- Install k3d as [seen here](https://github.com/invalidred/argo-workflow#setup)

- Install crossplane

```bash
helm repo add crossplane-stable https://charts.crossplane.io/stable

helm repo update

helm install crossplane \
--namespace crossplane-system \
--create-namespace crossplane-stable/crossplane
```

- Install crossplane kubernetes provider

```bash
kubectl apply -f crossplane-k8s-provider.yaml

SA=$(kubectl -n crossplane-system get sa -o name | grep provider-kubernetes | sed -e 's|serviceaccount\/|crossplane-system:|g')

kubectl create clusterrolebinding provider-kubernetes-admin-binding --clusterrole cluster-admin --serviceaccount="${SA}"

kubectl apply -f crossplane-k8s-provider-config.yaml
```


## General k8s Deployment

See [my-server-native-k8s](./my-server-native-k8s.yaml) file. You will notice approximately **70+ lines of configuration**

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  <truncated>

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-server
  namespace: eng
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-server
  template:
    metadata:
      <truncated>
    spec:
      containers:
        <truncated>
```

You can create the above in k3s cluster by running

```bash
k apply -f my-server-native-k8s.yaml

k get all -n eng
```

## Using Crossplane Compositions

See [my-server-crossplane-xrd](./my-server-crossplane-xrd.yaml). You will the same complexity as above with only **20 lines** of _yaml_.

Notice you will see a custom CRD of `BackendServer` that we can customize to our needs.


```bash
apiVersion: my-domain.org/v1alpha1
kind: BackendServer
metadata:
  name: abdul-server-noice
spec:
  compositionSelector:
    matchLabels:
      lang: nodejs
  name: abdul-server-v1
  namespace: eng
  image: mendhak/http-https-echo
  version: "30"
  replicas: 3
  resources:
    cpu: 100m
    memory: 100Mi
  port: 8080
  config:
    foo: "bar-server"
    bar: "pig-server-abdul"
```

Also notice the `compositionSelector.matchLabels.lang = nodejs`. We can dynamically use a different composition based on another language if needed in a generic way.


## Crossplane composition internals

A Crosspane composition is made up of 2 parts

### CompositeResourceDefintion

This is akin to a CRD (Custom Resource Definition). It uses OpenAPI specification to define the [`BackendServer` CRD](./crossplane-composition-backend-server/api.yaml).

We define a spec that requires
- `image`
- `version`
- `name`
- `replicas`
- `port`
- `resources`

Using defaults we coud require even lesser properites.


```bash
kubectl apply -f crossplane-composition-backend-server
```


### Composition

This contains all the configuration needed to create k8s resources and other crossplane managed resources like s3 buckets, ecr repositories etc..

```bash
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: backend-server-nodejs
  labels:
    lang: nodejs
spec:
  compositeTypeRef:
    apiVersion: my-domain.org/v1alpha1
    kind: XBackendServer
  resources:
    - name: BackenServerConfigMap
    <truncated>
    - name: BackendServerDeployment
    <truncated>
```

_To view all compositions registred by crossplane_
```bash
kubectl get compositions


# output

NAME                        XR-KIND          XR-APIVERSION            AGE
backend-server-config-map   XBackendServer   my-domain.org/v1alpha1   27m
backend-server-nodejs       XBackendServer   my-domain.org/v1alpha1   25m
backend-server-python       XBackendServer   my-domain.org/v1alpha1   25m
```

_To view al composite resources_

```bash
kubectl get composite

# output

NAME                       SYNCED   READY   COMPOSITION             AGE
abdul-server-noice-s2ptd   True     True    backend-server-nodejs   26m
```

## Learn more

We've covered most of the concepts below. There are some nuances, so do read the articles below. However I found the documentation to be a tad hard to follow due to the number concepts we need to keep in our brain at any given time.

- [Compositions](https://docs.crossplane.io/latest/concepts/compositions/)
- [Composite resource definitions](https://docs.crossplane.io/latest/concepts/composite-resource-definitions/)
- [Composite resource](https://docs.crossplane.io/latest/concepts/composite-resources/)
- [Claims](https://docs.crossplane.io/latest/concepts/claims/)
