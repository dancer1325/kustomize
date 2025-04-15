---
title: "Creating Your First Kustomization"
linkTitle: "Creating Your First Kustomization"
date: 2022-02-27
weight: 20
description: >
  A step by step tutorial for absolute kustomize beginners
---
[kustomization reference]: https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/

* [here](/examples/firstKustomization)

# simple case
* | [here](/examples/firstKustomization),
  * `kustomize build .`
    * generate a manifest / contains ALL resources

# Customize resources -- via -- `namePrefix` transformer
 
* | [here](/examples/firstKustomization/customizeResources),
  * `kustomize build .`
    * generate a manifest / 
      * contains ALL resources
      * add a prefix

# Create variants -- via -- overlays

* use case
  * deploy nginx manifests -- to -- 2 environments (`Staging` & `Production`) /
    * IDENTICAL == "variants" 
    * few minor changes

* directory structure
    ```
    kustomize-example
    ├── base
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   └── service.yaml
    └── overlays
        ├── production
        │   └── kustomization.yaml
        └── staging
            └── kustomization.yaml
    ```
* | [production](/examples/firstKustomization/variantsViaOverlays/overlays/production)
  * `kustomize build .`
* | [staging](/examples/firstKustomization/variantsViaOverlays/overlays/staging)
  * `kustomize build .`

# Customizing overlays

* goal

|Requirement|             Production             |           Staging           |
|-----------|------------------------------------|-----------------------------|
|Name       |env1-example-nginx-production       |env2-example-nginx-staging   |
|Namespace  |production                          |staging                      |
|Replicas   |3                                   |2                            |

* directory structure
    ```
    kustomize-example
    ├── base
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   └── service.yaml
    └── overlays
        ├── production
        │   └── kustomization.yaml
        └── staging
            └── kustomization.yaml
    ```

* | production
  * `kustomize build .`
* | staging
  * `kustomize build .`

# Further customizations

|Requirement|             Production             |           Staging           |
|-----------|------------------------------------|-----------------------------|
|Image      |nginx:1.20.2                        |nginx:latest                 |
|Label      |variant=var1                        |variant=var2                 |
|Env Var    |ENVIRONMENT=prod                    |ENVIRONMENT=stg              |
  
* `images`
  * allows
    * specifying the image tag

* `labels`
  * allows
    * setting the label

* `patches`
  * allows
    * modifying a resource (_Example:_ resource's environment variable)

* TODO:
One important thing to note here is that the name of the deployment used is the name that you are getting from the base and not the deployment name that has the prefix and suffix added.

Rebuilding the overlay shows that the environment variable has been added to your container:

```yaml
$ kustomize build overlays/production/

apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
    variant: var1
  name: env1-example-nginx-production
  namespace: production
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
    variant: var1
  name: env1-example-nginx-production
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        variant: var1
    spec:
      containers:
      - env:
        - name: ENVIRONMENT               ### Environment variable has been added here
          value: prod
        image: nginx:1.20.2
        name: nginx
        ports:
        - containerPort: 80
```

Looking at the output of `kustomize build` you can see that the additional requirements that were set have now been met.  
Below are the files as they should be at this point in your overlays and the `kustomize build` output:

_kustomize-example/overlays/production/kustomization.yaml_:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namePrefix: env1-
nameSuffix: -production

namespace: production

replicas:
- name: example-nginx
  count: 3

images:
- name: nginx
  newTag: 1.20.2

labels:
- pairs:
    variant: var1
  includeSelectors: false
  includeTemplates: true

resources:
- ../../base

patches:
- path: patch-env-vars.yaml
```

_kustomize-example/overlays/production/patch-env-vars.yaml_:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-nginx
spec:
  template:
    spec:
      containers:
      - name: nginx
        env:
        - name: ENVIRONMENT
          value: prod
```

_kustomize-example/overlays/staging/kustomization.yaml_:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namePrefix: env2-
nameSuffix: -staging

namespace: staging

replicas:
- name: example-nginx
  count: 2

images:
- name: nginx
  newTag: latest

labels:
- pairs:
    variant: var2
  includeSelectors: false
  includeTemplates: true

resources:
- ../../base

patches:
- path: patch-env-vars.yaml
```

_kustomize-example/overlays/staging/patch-env-vars.yaml_:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-nginx
spec:
  template:
    spec:
      containers:
      - name: nginx
        env:
        - name: ENVIRONMENT
          value: stg
```

_Production overlay build_:

```yaml
$ kustomize build overlays/production/

apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
    variant: var1
  name: env1-example-nginx-production
  namespace: production
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
    variant: var1
  name: env1-example-nginx-production
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        variant: var1
    spec:
      containers:
      - env:
        - name: ENVIRONMENT
          value: prod
        image: nginx:1.20.2
        name: nginx
        ports:
        - containerPort: 80

```

_Staging overlay build_:

```yaml
$ kustomize build overlays/staging/

apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
    variant: var2
  name: env2-example-nginx-staging
  namespace: staging
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
    variant: var2
  name: env2-example-nginx-staging
  namespace: staging
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        variant: var2
    spec:
      containers:
      - env:
        - name: ENVIRONMENT
          value: stg
        image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
```

# Next steps

Congratulations on making it to the end of this tutorial.  
As a summary for you, these are the customizations that were presented in this tutorial:

- Add a name prefix and a name suffix
- Set the namespace for your resources
- Set the number of replicas for your deployment
- Set the image to use
- Add a label to your resources
- Add an environment variable to a container by using a patch

If you are interested to learn more, the [kustomization reference] is your next step. 
You will see how you can use components to define base resources and add them to specific overlays where needed, use generators to create configMaps from files, and much more!
