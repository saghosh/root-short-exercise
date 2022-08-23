---
title: "ROOT scripts"
teaching: 10
exercises: 10
questions:
- "How to use ROOT scripts?"
objectives:
- "Learn about using C++ scripts with ROOT"
keypoints:
- "ROOT provides many features from histogramming, fitting and plotting to investigating data interactively in C++ and Python but scrpts can help with advanced work"
---

> This section is dedicated to the introduction to using ROOT scripts or macros.
{: .testimonial}

### ROOT scripts

A unique feature of ROOT is the possibility to use C++ scripts, also called "ROOT macros". A ROOT script contains valid C++ code and uses as entrypoint a function with the same name as the script. Let's take as example the file `myScript.C` with the following content.

```cpp
void myScript() {
    auto file = TFile::Open("https://root.cern/files/tmva_class_example.root");
    for (auto key : *file->GetListOfKeys()) {
        const auto name = key->GetName();
        const auto entries = file->Get<TTree>(name)->GetEntries();
        std::cout << name << " : " << entries << std::endl;
    }
}
```

Scripts can be processed by passing them as argument to the `root` executable:

```bash
$ root myScript.C

root [0]
Processing myScript.C...
TreeS : 6000
TreeB : 6000
```

The advantage of such scripts is the simple interaction with C++ libraries (such as ROOT) and running your code at C++ speed with the convenience of a script.

## Compiled C++

You can improve the runtime of your programs if you compile them upfront. Therefore, ROOT tries to make the compilation of ROOT macros as convenient as possible!

### ACLiC

ROOT provides a mechanism called ACLiC to compile the script in a shared library and call the compiled code from interactive C++, all automatically!

The only change required to our script is that we need to include all required headers:

```cpp
#include "TFile.h"
#include "TTree.h"
#include <iostream>

void myScript() {
    // The body of the myScript function goes here
}
```

Now, let's compile and run the script again. Note the `+` after the script name!

```bash
$ root myScript.C+

root [0]
Processing myScript.C+...
Info in <TUnixSystem::ACLiC>: creating shared library /path/to/myScript_C.so
TreeS : 6000
TreeB : 6000
```

ACLiC has many more features, for example compiling your program with debug symbols using `+g`. You can find the documentation [here](https://root.cern/manual/first_steps_with_root/#compiling-a-root-macro-with-aclic).

### C++ compilers

Of course, the C++ code can also just be compiled with C++ compilers such as `g++` or `clang++` with the advantage that you have full control of all compiler settings, most notable the optimization flags such as `-O3`!

To do so, we have to add the `main` function to the script, which is the default entrypoint for C(++) programs.

```cpp
#include "TFile.h"
#include "TTree.h"
#include <iostream>

void myScript() {
    // The body of the myScript function goes here
}

int main() {
    myScript();
    return 0;
}
```

Now, you can use the following command with your C++ compiler of choice to compile the script into an executable.

```bash
$ g++ -O3 -o myScript myScript.C $(root-config --cflags --libs)
$ ./myScript
TreeS : 6000
TreeB : 6000
```

Computationally heavy programs and long running analyses may benefit greatly from the optimized compilation with `-O3` and can save you hours of computing time!

{% include links.md %}
