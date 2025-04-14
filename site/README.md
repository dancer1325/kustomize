# kustomize website: the alpha of the alpha

* legacy website 

## Overview
* -- based on -- [docsy exmaple website](https://example.docsy.dev/)

## How to run locally?

* requisites
  - [npm](https://www.npmjs.com/)
  - [Go](https://go.dev/)
  - [Hugo (Extended version)](https://gohugo.io/)
  - container runtime, like [Docker](https://www.docker.com/)
* ways
  * -- via -- [Hugo (Extended version)](https://gohugo.io/)
  * | container runtime
    * recommended one
* steps
  * pull | submodule & OTHER development dependencies
    * |Windows
        ```powershell
        # fetch submodule dependencies
        git submodule update --init --recursive --depth 1
        ```
    * | Linux / Unix
        ```bash
        # fetch submodule dependencies
        make module-init
        ```
   * follow
     * [-- via -- container](#---via----container) or
     * [-- via -- Hugo](#---via----hugo)

### -- via -- container

```bash
# You can set $CONTAINER_ENGINE to the name of any Docker-like container tool
# Build the image
make container-image

# Run the container
make container-serve
```

* TODO: If you see errors, it probably means that the hugo container did not have enough computing resources available. 
To solve it, increase the amount of allowed CPU and memory usage for Docker on your machine ([MacOS](https://docs.docker.com/desktop/settings/mac/) and [Windows](https://docs.docker.com/desktop/settings/windows/)).

Open up your browser to <http://localhost:1313> to view the website. As you make changes to the source files, Hugo updates the website and forces a browser refresh.

If you need to update the `HUGO_VERSION`, edit it in the [`netlify.toml`](netlify.toml#L7) file and then run `make tools` to update the pinned version of Hugo in [hack/go.mod](/hack/go.mod#L9).

### -- via -- Hugo

Make sure to install the Hugo extended version specified by the `HUGO_VERSION` environment variable in the [`netlify.toml`](netlify.toml#L7) file.

To install dependencies, deploy and test the site locally, run:

- For macOS and Linux
  ```bash
  npm ci
  make serve
  ```
- For Windows (PowerShell)
  ```powershell
  npm ci
  hugo.exe server --buildFuture --environment development
  ```

This will start the local Hugo server on port 1313. Open up your browser to <http://localhost:1313> to view the website. As you make changes to the source files, Hugo updates the website and forces a browser refresh.

## Troubleshooting

### error: failed to transform resource: TOCSS: failed to transform "scss/main.scss" (text/x-scss): this feature is not available in your current Hugo version

Hugo is shipped in two set of binaries for technical reasons. The current website runs based on the **Hugo Extended** version only. In the [release page](https://github.com/gohugoio/hugo/releases) look for archives with `extended` in the name. To confirm, run `hugo version` and look for the word `extended`.

### Troubleshooting macOS for too many open files

If you run `make serve` on macOS and receive the following error:

```bash
ERROR 2020/08/01 19:09:18 Error: listen tcp 127.0.0.1:1313: socket: too many open files
make: *** [serve] Error 1
```

Try checking the current limit for open files:

`launchctl limit maxfiles`

Then run the following commands (adapted from <https://gist.github.com/tombigel/d503800a282fcadbee14b537735d202c>):

```shell
#!/bin/sh

# These are the original gist links, linking to my gists now.
# curl -O https://gist.githubusercontent.com/a2ikm/761c2ab02b7b3935679e55af5d81786a/raw/ab644cb92f216c019a2f032bbf25e258b01d87f9/limit.maxfiles.plist
# curl -O https://gist.githubusercontent.com/a2ikm/761c2ab02b7b3935679e55af5d81786a/raw/ab644cb92f216c019a2f032bbf25e258b01d87f9/limit.maxproc.plist

curl -O https://gist.githubusercontent.com/tombigel/d503800a282fcadbee14b537735d202c/raw/ed73cacf82906fdde59976a0c8248cce8b44f906/limit.maxfiles.plist
curl -O https://gist.githubusercontent.com/tombigel/d503800a282fcadbee14b537735d202c/raw/ed73cacf82906fdde59976a0c8248cce8b44f906/limit.maxproc.plist

sudo mv limit.maxfiles.plist /Library/LaunchDaemons
sudo mv limit.maxproc.plist /Library/LaunchDaemons

sudo chown root:wheel /Library/LaunchDaemons/limit.maxfiles.plist
sudo chown root:wheel /Library/LaunchDaemons/limit.maxproc.plist

sudo launchctl load -w /Library/LaunchDaemons/limit.maxfiles.plist
```

This works for Catalina as well as Mojave macOS.
