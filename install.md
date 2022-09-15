---
description: >-
  ObjectBox Golang DB is easy to install. Just follow this steps and enjoy the
  ease of use and speed of the ObjectBox NoSQL database in your next Golang
  application.
---

# Installation

We're trying to make the installation experience smooth for everyone. In case you're getting stuck or are finding some steps hard to follow, please reach out to us, e.g. by creating a [GitHub issue](https://github.com/objectbox/objectbox-go/issues) or through our [Contact form](https://objectbox.io/contact/). Thanks!

## Linux/macOS

This section describe the installation on Linux or macOS, if you're using Windows, please skip to the [Installation on Windows](install.md#windows) section.&#x20;

The main prerequisite to using ObjectBox in Go is the ObjectBox binary library (.so, .dylib depending on your platform) which actually implements the database functionality.&#x20;

We are using CGO which requires you to have a C/C++ compiler, such as `gcc` or `clang`,  installed. You can try executing one of following commands in terminal to check if it's already available and working: `$CC --version` or `gcc --version` or `clang --version`. If any of the commands works fine (no need for all of them to work), you should be good to go. Otherwise, please `gcc` or `clang` according to the  instructions for your system (e.g. `sudo apt install gcc` on Ubuntu).

{% hint style="warning" %}
There's currently a known issue on some ARM platforms, see [Raspberry Pi 3 & 4 fallback](install.md#raspberry-pi-3-and-4-fallback).
{% endhint %}

### Quick installation

The fastest way to install is by using our installation script. Execute the following command in your project directory. If that doesn't work for you, you can skip to the manual installation bellow.

{% tabs %}
{% tab title="ObjectBox" %}
```bash
bash <(curl -s https://raw.githubusercontent.com/objectbox/objectbox-go/main/install.sh)
```
{% endtab %}

{% tab title="ObjectBox Sync" %}
```
bash <(curl -s https://raw.githubusercontent.com/objectbox/objectbox-go/main/install.sh) --sync
```
{% endtab %}
{% endtabs %}

{% content-ref url="getting-started.md" %}
[getting-started.md](getting-started.md)
{% endcontent-ref %}

### Manual installation

**C binary library**\
****You can run the following`download.sh` script (press Y to install the library to a system-wide folder when it asks you). you can remove the temporary "objectboxlib" directory created by this step afterwards.

```bash
mkdir objectboxlib && cd objectboxlib
bash <(curl -s https://raw.githubusercontent.com/objectbox/objectbox-c/main/download.sh) 0.13.0

// Or for ObjectBox Sync:
bash <(curl -s https://raw.githubusercontent.com/objectbox/objectbox-c/main/download.sh) --sync 0.13.0
```

#### Go package dependency&#x20;

{% tabs %}
{% tab title="Go modules based project" %}
If your project is using a `go.mod` file to keep track of the dependencies, you don't need to install anything else and you can just start using ObjectBox by importing it in your source code:

```go
import "github.com/objectbox/objectbox-go/objectbox"
```
{% endtab %}

{% tab title=" Legacy projects without a go.mod file" %}
In case you're not using Go modules, you can install ObjectBox using following commands:

```bash
go get -u github.com/objectbox/objectbox-go/...
go get -u github.com/google/flatbuffers/go
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
A package dependency is set up automatically if you've used the "Quick installation" method using the `install.sh` script
{% endhint %}

{% content-ref url="getting-started.md" %}
[getting-started.md](getting-started.md)
{% endcontent-ref %}

### Raspberry Pi 3 & 4 fallback

You may encounter an issue on some ARM platforms (seen this on Raspberry Pi 3 & 4) with the default native library installed by the script. Also, the issue seems to only occur when running natively, not inside docker.

You can check your installation to see if you encounter crashes (SIGBUS/SIGSEGV) by executing `go test github.com/objectbox/objectbox-go/...`&#x20;

As a workaround, you can install an ARMv6 version of the native library (instead of the default ARMv7 the script picks) using the same script as in the manual installation, just changing the arguments:

```bash
./download.sh 0.13.0 Linux armv6
```

## Windows

The main prerequisite to using ObjectBox in Go is the ObjectBox binary DLL which actually implements the database functionality. We are using CGO which requires you to have MinGW `gcc` in PATH, e. g. [http://tdm-gcc.tdragon.net/](http://tdm-gcc.tdragon.net/) - in case you don't have MinGW installed yet, please do so first.

### **Quick installation**

You can use the following PowerShell script to help you with installation of the library to the right folders:

* download the script: (right click and "Save link as", the location doesn't matter) [install.ps1](https://raw.githubusercontent.com/objectbox/objectbox-go/main/install.ps1)
* run the script - either double click or right click and "Run in PowerShell" - depends on your settings.
* the script will guide you through the installation steps.

### **Manual installation**

In case you couldn't install fully using the script (e.g. it didn't find your MinGW installation), you can try to finish the installation manually

1. In order to compile your program, you need to copy the downloaded`download/objectbox.dll` to the MinGW library directory, e. g. `C:\TDM-GCC-64\lib`
2. To run the program, you either need to have the `objectbox.dll` library in the same folder as the compiled program, have its location (e. g. `C:\TDM-GCC-64\lib`in `Path` environment variable), or copy it to the system library directory `c:\Windows\System32`

### Go package dependency&#x20;

{% tabs %}
{% tab title="Go modules based project" %}
If your project is using a `go.mod` file to keep track of the dependencies, you don't need to install anything else and you can just start using ObjectBox by importing it in your source code:

```go
import "github.com/objectbox/objectbox-go/objectbox"
```
{% endtab %}
{% endtabs %}

{% content-ref url="getting-started.md" %}
[getting-started.md](getting-started.md)
{% endcontent-ref %}

### Distributing

Don't forget to include the `objectbox.dll` with your program when distributing/packaging for installer. Having it in the same directory as your program binary should be enough.
