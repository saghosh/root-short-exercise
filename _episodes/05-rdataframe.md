---
title: "Using ROOT with Python"
teaching: 10
exercises: 15
questions:
- "How can I use ROOT with Python?"
objectives:
- "Learn about the basics of ROOT with Python"
keypoints:
- "ROOT can be used with python"
- "PyROOT lets you use C++ from Python but offers many more advanced features to speed up your analysis in Python. Details about the dynamic Python bindings provided by PyROOT can be found on [https://root.cern/manual/python](https://root.cern/manual/python/)."
---
## Python

ROOT provides the Python bindings called PyROOT. PyROOT is not just ROOT from Python, but a full-featured interface to call C++ libraries in a pythonic way. This lets you import the ROOT module from Python and makes all features dynamically available. Let's rewrite the C++ example from above and put the code in the file `myScript.py`!

```python
import ROOT

rfile = ROOT.TFile.Open('https://root.cern/files/tmva_class_example.root')
for key in rfile.GetListOfKeys():
    name = key.GetName()
    entries = rfile.Get(name).GetEntries()
    print('{} : {}'.format(name, entries))
```

Calling the Python script works as expected:

```bash
$ python myScript.py
TreeS : 6000
TreeB : 6000
```

But PyROOT can do much more for you than simply providing access to C++ libraries from Python. You can also inject efficient C++ code into your Python program to speed up potentially slow parts of your program!

```python
import ROOT

ROOT.gInterpreter.Declare('''
int my_heavy_computation(std::string x) {
    // heavy computation goes here
    return x.length();
}
''')

# Functions and object made available via the interpreter are accessible from
# the ROOT module
y = ROOT.my_heavy_computation("the ultimate answer to life and everything")
print(y) # Guess the result!
```

A guide to such advanced features of PyROOT can be found in the official manual at [https://root.cern/manual/python](https://root.cern/manual/python/). Feel free to investigate!

> ## Try using ROOT with interactive C++, compiled C++ and Python!
> Make yourself familiar with the different ways you can run an analysis with ROOT!


{% include links.md %}

