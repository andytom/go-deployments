# Go Deployments

---

``` bash
go build
```

**What Now?** <!-- .element: class="fragment" -->
- The binary is now sitting on you laptop/CI server. How does anyone else get it? <!-- .element: class="fragment" -->
- What about users who use a different OS or Architecture? <!-- .element: class="fragment" -->

---

## Disclaimer

This is how we do it at iZettle, this works for us but might not work for you.

---

## What we deploy

**Services**

- Applications in our growing fleet of micro-services <!-- .element: class="fragment" -->
- All of them are Web Applications but planning to add Background Workers soon <!-- .element: class="fragment" -->

**Tools** <!-- .element: class="fragment" -->

- Command line applications for use by developers and our CI system <!-- .element: class="fragment" -->
- Support \*nix systems <!-- .element: class="fragment" -->

---

## Tools

* Cross Compile for all platforms
* We use [mitchellh/gox](https://github.com/mitchellh/gox) to run this in parallel
* Upload all the compiled artifacts to github releases page
* We use [tcnksm/ghr](https://github.com/tcnksm/ghr) which also does it in parallel
* We also upload an install script to help with installation

+++

**build.sh**

``` bash
gox -osarch="linux/amd64 darwin/amd64"\
    -output="bin/{{.Dir}}_{{.OS}}_{{.Arch}}"

ghr -u example -r awesome_app "${TAG}" bin/
```

---

## Services

* Compile a static binary for Linux
* Build a Docker image and push to the Registry
* Update the Kubernetes config file which triggers the deploy to the Kubernetes Cluster

+++

**DockerFile**

```docker
FROM alpine:3.5

MAINTAINER user@example.com

COPY awesome_app /usr/bin/awesome_app

CMD ["awesome_app"]
```

**build.sh**

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o awesome_app

docker build -t ${DOCKER_TAG} .
```

---

## Tips and Gotchas

**Not always static**
* Static isn't always static. If you compile on Linux for Linux it will link to libc.
* Use `CGO_ENABLED=0` to force static compilation

---

## Tips and Gotchas

**Package your services**
* There is more to the service than just the binary
* Use native packages (rpm, deb, etc) or docker to package the whole service

+++

## Building Packages

You can use [jordansissel/fpm](https://github.com/jordansissel/fpm) to make
this really simple.

```bash
fpm -s dir -t deb -n awesome_app \
    --config-files /etc/awesome_app/awesome.conf \
    -v ${TAG} \
    awesome_app=/usr/bin \
    awesome.conf=/etc/awesome_app
```

---

## Tips and Gotchas


**Script your builds**

* Building and deploying the app takes multiple steps with a lot of arguments for each step
* Write some simple build scripts (`shell`, `make`, etc) to make you release process a single script

---

# Questions?
