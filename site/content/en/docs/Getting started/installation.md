---
title: "Install Kustomize"
linkTitle: "Install Kustomize"
date: 2022-02-27
weight: 10
description: >
  Kustomize can be installed in a variety of ways
---

## -- FROM -- Binaries

* can be found | [releases page](https://github.com/kubernetes-sigs/kustomize/releases)

* steps
  * `curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash`

## Homebrew / MacPorts

* FOR [Homebrew](https://brew.sh) users
  * `brew install kustomize`

* FOR [MacPorts](https://www.macports.org) users
  * `sudo port install kustomize`

## Chocolatey

* [Choco package](https://chocolatey.org/packages/kustomize)

* steps
  * `choco install kustomize`

## Docker Images

* requirements
  * Kustomize v3.8.7+

* hosted | [Google Container Registry (GCR)](https://us.gcr.io/k8s-artifacts-prod/kustomize/kustomize)

* steps

    ```bash
    docker pull registry.k8s.io/kustomize/kustomize:{{< example-version >}}
    docker run registry.k8s.io/kustomize/kustomize:{{< example-version >}} version
    ```

## Go Source code

* == this repo

* requirements
  * install [Go](https://golang.org)

### -- from -- source / WITHOUT cloning the repo

```bash
go install sigs.k8s.io/kustomize/kustomize/{{< example-major-version >}}
```

### -- from -- local source

```bash
# Clone the repo
git clone git@github.com:kubernetes-sigs/kustomize.git

# Get into the repo root
cd kustomize

# if you do NOT want to build | head -> checkout a particular tag 
git checkout kustomize/{{< example-version >}}

# requirements: GOBIN OR GOPATH/bin is | your PATH
# Build the binary
(cd kustomize; go install .)

# Run it
kustomize version
```
