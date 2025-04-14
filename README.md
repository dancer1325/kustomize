# kustomize

* `kustomize`
  * allows you,
    * customize YAML /
      * raw OR template-free
      * MULTIPLE purposes
      * untouchable original YAML
  * targets kubernetes /
    * understands & can patch [kubernetes style] API objects
  * ==
    * [`make`]
      * Reason: 🧠declarative | file🧠
    * [`sed`]
      * Reason:🧠emits edited text🧠

* -- sponsored by -- [sig-cli] ([KEP])

- [Installation instructions](https://github.com/dancer1325/kubernetes-sigs-cli-experimental/tree/master/site/content/en/installation/kustomize)
- [General documentation](https://github.com/dancer1325/kubernetes-sigs-cli-experimental/blob/master/site/content/en/guides/introduction/kustomize.md)
- [Examples](examples)
- [docs](site/content/en) 

## kubectl integration

* 💡== embedded | kubectl💡
    ```sh
    > kubectl version --client
    Client Version: v1.31.0
    Kustomize Version: v5.4.2
    ```
  * [kubectl v1.14][kubectl announcement]

| Kubectl version | Kustomize version |
| --------------- | ----------------- |
| < v1.14         | n/a               |
| v1.14-v1.20     | v2.0.3            |
| v1.21           | v4.0.5            |
| v1.22           | v4.2.0            |
| v1.23           | v4.4.1            |
| v1.24           | v4.5.4            |
| v1.25           | v4.5.7            |
| v1.26           | v4.5.7            |
| v1.27           | v5.0.1            |

[v2.0.3]: https://github.com/kubernetes-sigs/kustomize/releases/tag/v2.0.3
[#2506]: https://github.com/kubernetes-sigs/kustomize/issues/2506
[#1500]: https://github.com/kubernetes-sigs/kustomize/issues/1500
[kust-in-kubectl update]: https://github.com/kubernetes/kubernetes/blob/4d75a6238a6e330337526e0513e67d02b1940b63/CHANGELOG/CHANGELOG-1.21.md#kustomize-updates-in-kubectl

* how to use the kubectl integration?
  * see [kubernetes documentation]

## How to use?

### 1) Make a [kustomization] file

* | SOME directory / contain your YAML [resource] files (deployments, services, configmaps, etc.)
  * create a [kustomization] file
  * add a COMMON label

    ```
    base: kustomization + resources
    
    kustomization.yaml                                      deployment.yaml                                                 service.yaml
    +---------------------------------------------+         +-------------------------------------------------------+       +-----------------------------------+
    | apiVersion: kustomize.config.k8s.io/v1beta1 |         | apiVersion: apps/v1                                   |       | apiVersion: v1                    |
    | kind: Kustomization                         |         | kind: Deployment                                      |       | kind: Service                     |
    | labels:                                     |         | metadata:                                             |       | metadata:                         |
    | - includeSelectors: true                    |         |   name: myapp                                         |       |   name: myapp                     |
    |   pairs:                                    |         | spec:                                                 |       | spec:                             |
    |     app: myapp                              |         |   selector:                                           |       |   selector:                       |
    | resources:                                  |         |     matchLabels:                                      |       |     app: myapp                    |
    |   - deployment.yaml                         |         |       app: myapp                                      |       |   ports:                          |
    |   - service.yaml                            |         |   template:                                           |       |     - port: 6060                  |
    | configMapGenerator:                         |         |     metadata:                                         |       |       targetPort: 6060            |
    |   - name: myapp-map                         |         |       labels:                                         |       +-----------------------------------+
    |     literals:                               |         |         app: myapp                                    |
    |       - KEY=value                           |         |     spec:                                             |
    +---------------------------------------------+         |       containers:                                     |
                                                            |         - name: myapp                                 |
                                                            |           image: myapp                                |
                                                            |           resources:                                  |
                                                            |             limits:                                   |
                                                            |               memory: "128Mi"                         |
                                                            |               cpu: "500m"                             |
                                                            |           ports:                                      |
                                                            |             - containerPort: 6060                     |
                                                            +-------------------------------------------------------+
    ```

    > ```
    > ~/someApp
    > ├── deployment.yaml
    > ├── kustomization.yaml
    > └── service.yaml
    > ```

* `kustomize build ~/someApp`
  * generate customized YAML 
* `kustomize build ~/someApp | kubectl apply -f -`
  * YAML can be DIRECTLY [applied] | cluster

### 2) Create [variants] -- using -- [overlays]

* use cases
  * manage TRADITIONAL [variants] of a configuration -- via -- [overlays] / modify a COMMON [base]

```

overlay: kustomization + patches

kustomization.yaml                                      replica_count.yaml                      cpu_count.yaml
+-----------------------------------------------+       +-------------------------------+       +------------------------------------------+
| apiVersion: kustomize.config.k8s.io/v1beta1   |       | apiVersion: apps/v1           |       | apiVersion: apps/v1                      |
| kind: Kustomization                           |       | kind: Deployment              |       | kind: Deployment                         |
| labels:                                       |       | metadata:                     |       | metadata:                                |
|  - includeSelectors: true                     |       |   name: myapp                 |       |   name: myapp                            |
|    pairs:                                     |       | spec:                         |       | spec:                                    |
|      variant: prod                            |       |   replicas: 80                |       |  template:                               |
| resources:                                    |       +-------------------------------+       |     spec:                                |
|   - ../../base                                |                                               |       containers:                        |
| patches:                                      |                                               |         - name: myapp                    |
|   - path: replica_count.yaml                  |                                               |           resources:                     |
|   - path: cpu_count.yaml                      |                                               |             limits:                      |
+-----------------------------------------------+                                               |               memory: "128Mi"            |
                                                                                                |               cpu: "7000m"               |
                                                                                                +------------------------------------------+
```

> ```
> ~/someApp
> ├── base
> │   ├── deployment.yaml
> │   ├── kustomization.yaml
> │   └── service.yaml
> └── overlays
>     ├── development
>     │   ├── cpu_count.yaml
>     │   ├── kustomization.yaml
>     │   └── replica_count.yaml
>     └── production
>         ├── cpu_count.yaml
>         ├── kustomization.yaml
>         └── replica_count.yaml
> ```

* steps
  * place 
    * [step 1 files](#1-make-a-kustomization-file) | `someApp/base/` subdirectory called `base`
    * overlays | `someApp/overlays/`
* allows
  * split management
    * == "base/" owners != overlays/" owners

* uses
  * `kustomize build ~/someApp/overlays/production | kubectl apply -f -`
    * generate YAML / directly [applied] | cluster

[`make`]: https://www.gnu.org/software/make
[`sed`]: https://www.gnu.org/software/sed
[DAM]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#declarative-application-management
[KEP]: https://github.com/kubernetes/enhancements/blob/master/keps/sig-cli/2377-Kustomize/README.md
[Kubernetes Code of Conduct]: code-of-conduct.md
[applied]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#apply
[base]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#base
[declarative configuration]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#declarative-application-management
[kubectl announcement]: https://kubernetes.io/blog/2019/03/25/kubernetes-1-14-release-announcement
[kubernetes documentation]: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
[kubernetes style]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#kubernetes-style-object
[kustomization]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#kustomization
[overlay]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#overlay
[overlays]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#overlay
[release page]: https://github.com/kubernetes-sigs/kustomize/releases
[resource]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#resource
[resources]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#resource
[sig-cli]: https://github.com/kubernetes/community/blob/master/sig-cli/README.md
[variants]: https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#variant
