# Go

Go is a statically typed, compiled high-level programming language designed at Google by Robert Griesemer, Rob Pike, and Ken Thompson. It is syntactically similar to C, but with memory safety, garbage collection, structural typing, and CSP-style concurrency. It is often referred to as Golang because of its former domain name, golang.org, but its proper name is Go.

wikipedia.org/wiki/Go_(programming_language)

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/05/Go_Logo_Blue.svg/1920px-Go_Logo_Blue.svg.png" alt="go logo" width="60%" height="auto">

## How to use this Makejail

### Basic usage

Create a `Makejail` in your Go app project.

```
INCLUDE options/network.makejail
INCLUDE gh+AppJail-makejails/go

WORKDIR /app
COPY app/

RUN %{GO_EXECUTABLE} build hello.go

STAGE cmd

WORKDIR /app
RUN ./hello
```

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
INCLUDE options/network.makejail
INCLUDE gh+AppJail-makejails/go

WORKDIR /app
COPY app/

RUN %{GO_EXECUTABLE} build hello.go
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

### Build Arguments

* `GO_VERSION` (default: empty): If empty `lang/go` will be installed instead of a specific version. Valid versions are: `118`, `119`, `120`.
