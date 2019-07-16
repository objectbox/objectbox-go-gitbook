# Installation

### Quick installation

The fastest way to install is by using our installation script. Execute the following command in your project directory. If that doesn't work for you, you can skip to the manual installation bellow.

```bash
bash <(curl -s https://raw.githubusercontent.com/objectbox/objectbox-go/master/install.sh)
```

### Dependencies

**C binary library**  
The only prerequisite to using ObjectBox in Go is the ObjectBox binary library \(.so, .dylib, .dll depending on your platform\) which actually implements the database functionality.   
You can run the following`download.sh` script \(press Y to install the library to a system-wide folder when it asks you\). you can remove the temporary "objectboxlib" directory created by this step afterwards.

```bash
mkdir objectboxlib && cd objectboxlib
bash <(curl -s https://raw.githubusercontent.com/objectbox/objectbox-c/master/download.sh) 0.6.0
```

### Setup

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

At this point you can start working with ObjectBox in your project. Have a look at this example and follow to the Getting started section:

{% code-tabs %}
{% code-tabs-item title="main.go" %}
```go
package main

import (
        "fmt"
        "github.com/objectbox/objectbox-go/objectbox"
)

func main() {
        fmt.Println(objectbox.VersionInfo())
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% page-ref page="getting-started.md" %}

#### ObjectBox Library on Windows

There are some additional steps necessary for windows. 

1. We are using `cgo` which requires you to have MinGW gcc in PATH, e. g. [http://tdm-gcc.tdragon.net/](http://tdm-gcc.tdragon.net/)
2. In order to compile your program, you need to copy the downloaded`objectbox.dll` to the MinGW library directory, e. g. `C:\TDM-GCC-64\lib`
3. To run the program, you either need to have the `objectbox.dll` library in the same folder as the compiled program, or copy it to the system library directory `c:\Windows\System32`
4. Don't forget to include the objectbox.dll with your program when distributing/packaging for installer. Having it in the same directory as your program binary should be enough.



