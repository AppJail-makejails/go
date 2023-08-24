# Go

Go is a statically typed, compiled high-level programming language designed at Google by Robert Griesemer, Rob Pike, and Ken Thompson. It is syntactically similar to C, but with memory safety, garbage collection, structural typing, and CSP-style concurrency. It is often referred to as Golang because of its former domain name, golang.org, but its proper name is Go.

wikipedia.org/wiki/Go_(programming\_language)

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/05/Go_Logo_Blue.svg/1920px-Go_Logo_Blue.svg.png" alt="go logo" width="60%" height="auto">

## How to use this Makejail

### Basic usage

Create a `Makejail` in your Go app project.

```
OPTION start
OPTION overwrite

INCLUDE options/network.makejail

FROM go:13.2

WORKDIR /app
COPY app/

RUN go build hello.go

STAGE cmd

WORKDIR /app
RUN ./hello
```

**Note**: Remember that you can use `INCLUDE gh+AppJail-makejails/go` instead of `FROM go:...` to use arguments, but when using a specific version of go, you need to change the entry point when building a golang application.

Where `options/network.makejail` are the options that suit your environment, for example:

```
ARG network
ARG interface=goapp

OPTION virtualnet=${network}:${interface} default
OPTION nat
```

Open a shell and run `appjail makejail`:

```sh
appjail makejail -j goapp -- --network development
```

To run the application we can use `appjail run`:

```console
# appjail run goapp
Hello, world!
```

### Using Makejail builders

If we get the size of the previous jail

```console
# appjail stop goapp
...
# appjail cmd local goapp du -sh
471M    .
```

we have a very large jail to run a simple binary. For compiled programming languages we could use Makejail builders to reduce the size of the jail.

**Makejail**:

```
OPTION start
OPTION overwrite

EXEC --name goapp-builder --file build.makejail --arg network=development --arg interface=goappb

WORKDIR /app
COPY --jail goapp-builder /app/hello
DESTROY --force goapp-builder

STAGE cmd

WORKDIR /app
RUN ./hello
```

**build.makejail**:

```
OPTION start
OPTION overwrite

INCLUDE options/network.makejail

FROM go:13.2

WORKDIR /app
COPY app/

RUN go build hello.go
```

For simplicity, `Makejail` does not use more options than necessary, but you can use as many as you want without affecting `build.makejail`.

Open a shell and run `appjail makejail`:

```sh
appjail makejail -j goapp
```

Now our jail with the application we want to run has a very reduced size.

```console
# appjail stop goapp
...
# appjail cmd local goapp du -sh
 14M    .
```

Much of the size overhead if for jail, but for big applications this is not harmful.

### Arguments

* `go_tag` (default: `13.2`): see [#tags](#tags).

## How to use the Image

Make any changes you want to your image.

```
INCLUDE options/network.makejail
INCLUDE gh+AppJail-makejails/go --file build.makejail
```

Build the jail:

```sh
appjail makejail -j go
```

Remove unportable or unnecessary files and directories and export the jail:

```sh
appjail stop go
appjail cmd local go sh -c "rm -f var/log/*"
appjail cmd local go sh -c "rm -f var/cache/pkg/*"
appjail cmd local go sh -c "rm -f var/run/*"
appjail cmd local go vi etc/rc.conf
appjail image export go
```

### Arguments

* `go_version` (optional): Valid versions are `121`, `120` and `119`. If empty (default), the meta-port `lang/go` is used.

## Tags

| Tag        | Arch    | Version           | Type   | `go_version` |
| ---------- | ------- | ----------------- | ------ | ------------ |
| `13.2`     | `amd64` | `13.2-RELEASE-p2` | `thin` |       -      |
| `13.2-121` | `amd64` | `13.2-RELEASE-p2` | `thin` |      121     |
| `13.2-120` | `amd64` | `13.2-RELEASE-p2` | `thin` |      120     |
| `13.2-119` | `amd64` | `13.2-RELEASE-p2` | `thin` |      119     |
