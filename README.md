# Go

Go is a statically typed, compiled high-level programming language designed at Google by Robert Griesemer, Rob Pike, and Ken Thompson. It is syntactically similar to C, but with memory safety, garbage collection, structural typing, and CSP-style concurrency. It is often referred to as Golang because of its former domain name, golang.org, but its proper name is Go.

wikipedia.org/wiki/Go_(programming_language)

<img src="https://camo.githubusercontent.com/6c8462aa983febb3f0906ce528a857e1f63ea5bd2aa870a53c9a171e4fe7d89a/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f7468756d622f302f30352f476f5f4c6f676f5f426c75652e7376672f3139323070782d476f5f4c6f676f5f426c75652e7376672e706e67" width="30%" height="auto" alt="Go logo">

## How to use this Makejail

### Start a Go instance in your app

The most straightforward way to use this image is to use a Go container as both the build and runtime environment. In your `Containerfile`, writing something along the lines of the following will compile and run your project (assuming it uses `go.mod` for dependency management):

```dockerfile
FROM ghcr.io/appjail-makejails/go

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN go build -v -o /usr/local/bin/app ./...

CMD ["app"]
```

You can then build and run the OCI image:

```console
$ buildah build --network=host -t my-golang-app .
$ appjail oci run \
    -o overwrite=force \
    -o ephemeral \
    -o alias \
    -o ip4_inherit \
    localhost/my-golang-app my-golang-app
```

### Compile your app inside the container

There may be occasions where it is not appropriate to run your app inside a container. To compile, but not run your app inside the container instance, you can write something like:

```console
$ appjail oci run \
    -o overwrite=force \
    -o ephemeral \
    -o alias \
    -o ip4_inherit \
    -o fstab="$PWD /myapp" \
    -w /myapp \
    localhost/my-golang-app my-golang-app \
    go build -v
$ ls ./myapp
./myapp
```

### OCI image and Makejail

This repository includes a small `Makejail` that ultimately uses the OCI image, in case you prefer to use `appjail-makejail(5)`.

```console
$ appjail makejail \
    -j golang \
    -f gh+AppJail-makejails/go \
    -o alias \
    -o ip4_inherit \
    -o ephemeral
...
$ appjail cmd jexec golang go version
go version go1.25.11 freebsd/amd64
```

### Arguments (stage: build)

* `go_from` (default: `ghcr.io/appjail-makejails/go`): Location of OCI image. See also [OCI Configuration](#oci-configuration).
* `go_tag` (default: `latest`): OCI image tag. See also [OCI Configuration](#oci-configuration).

## OCI Configuration

```yaml
build:
  variants:
    - tag: 15.1
      containerfile: Containerfile
      aliases: ["latest"]
      default: true
      args:
        FREEBSD_RELEASE: "15.1"
    - tag: 15.1-125
      containerfile: Containerfile
      args:
        FREEBSD_RELEASE: "15.1"
        GOVER: "125"
    - tag: 15.1-126
      containerfile: Containerfile
      args:
        FREEBSD_RELEASE: "15.1"
        GOVER: "126"
```

## Notes

1. `/go` is world-writable to allow flexibility in the user which runs the container (for example, in a container started with `--user 1000:1000`, running `go get github.com/example/...` into the default `$GOPATH` will succeed). While the `777` directory would be insecure on a regular host setup, there are not typically other processes or users inside the container, so this is equivalent to `700` for Podman usage, but allowing for `--user` flexibility.
2. The ideas present in the Docker image of Go are taken into account for users who are familiar with it.
