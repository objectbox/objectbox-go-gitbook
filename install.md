# Installation

To get started with ObjectBox you can get the repository code as usual with `go get` and install the few prerequisites - flatbuffers, ObjectBox pre-compiled library and code generator generator.

```bash
go get github.com/objectbox/objectbox-go/...
go get github.com/google/flatbuffers/go

mkdir objectboxlib && cd objectboxlib
curl https://raw.githubusercontent.com/objectbox/objectbox-c/master/download.sh > download.sh
bash download.sh # press y to install to /usr/local/lib

go install github.com/objectbox/objectbox-go/cmd/objectbox-gogen/
go test github.com/objectbox/objectbox-go/...
```

At this point you should see the GO tests passing and you can start working with ObjectBox in your project:

{% page-ref page="getting-started.md" %}

In case you're interested or are having trouble with the installation, the following sections provide more details on the procedure.

### ObjectBox Library

The main prerequisite to using the Go APIs is the ObjectBox binary library \(.so, .dylib, .dll depending on your  platform\) which actually implements the database functionality. In the [ObjectBox C repository,](https://github.com/objectbox/objectbox-c) you should find a `download.sh` script you can run. 

```bash
curl https://raw.githubusercontent.com/objectbox/objectbox-c/master/download.sh > download.sh
bash download.sh
```

Follow the instructions and type `Y` when it asks you if it should install the library.

Note, if `curl` is absent on your system you can use wget instead:

```bash
wget https://raw.githubusercontent.com/objectbox/objectbox-c/master/download.sh
```

{% hint style="info" %}
See [https://github.com/objectbox/objectbox-c\#usage-and-installation](https://github.com/objectbox/objectbox-c#usage-and-installation) in case you are interested in additional installation instructions for the library.
{% endhint %}

#### ObjectBox Library on Windows

There are some additional steps necessary for windows. 

1. We are using `cgo` which requires you to have MinGW gcc in PATH, e. g. [http://tdm-gcc.tdragon.net/](http://tdm-gcc.tdragon.net/)
2. In order to compile your program, you need to copy the downloaded`objectbox.dll` to the MinGW library directory, e. g. `C:\TDM-GCC-64\lib`
3. To run the program, you either need to have the `objectbox.dll` library in the same folder as the compiled program, or copy it to the system library directory `c:\Windows\System32`

### Bindings generator

ObjectBox can generate Go bindings \(structs & functions\) for your data entities. While you certainly could do this manually, it would be quite tedious and that's why we provide a bindings generator. This is a command line program you can compile and install just by running 

```text
go install github.com/objectbox/objectbox-go/cmd/objectbox-gogen/
```

With the generator installed, you can just write a special comment `//go:generate objectbox-gogen` to the `.go` file where your entity/entities structs are defined and execute `go generate` on that folder or alternatively you can run the following command on your entire project \(execute it in your project root\):

```text
go generate ./...
```



