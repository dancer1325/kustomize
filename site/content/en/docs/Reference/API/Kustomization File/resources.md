---
title: "resources"
linkTitle: "resources"
type: docs
weight: 20
description: >
    Resources to include.
---

* == list(paths)
  * paths
    * requirements
      * EACH path ==    >=1 k8s manifest
        * if there are >1 -> separated by `---`
    * ALLOWED paths
      * path -- to a -- file, OR
      * path (or URL) -- to -- another kustomization directory
    * are read & processed -- through -- depth-first order
      * == PREVIOUSLY to pass to NEXT path, end read & process that item + item's children 

*  File paths should be specified _relative_ to the directory holding the
kustomization file containing the `resources` field.

Directory specification can be relative, absolute, or part of a URL
* URL specification
  * follow the [hashicorp URL format](https://github.com/hashicorp/go-getter#url-format) 
*  The directory must contain a `kustomization.yaml` file.

