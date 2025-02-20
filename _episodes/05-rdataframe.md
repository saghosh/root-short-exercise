---
title: "Using ROOT with Python"
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

## Basic histogramming, fitting and plotting

The following script uses basic features from ROOT, which are used commonly in day-to-day work with ROOT. You can investigate the typical workflow to create histograms with `TH1F`, fit a function to the data with `TF1` and produce an accurate visualization with `TCanvas` and others. Below, you can see the output of the fit to the data with the measured parameters.

```python
import ROOT
import numpy as np

# Make global style changes
ROOT.gStyle.SetOptStat(0) # Disable the statistics box
ROOT.gStyle.SetTextFont(42)

# Create a canvas
c = ROOT.TCanvas('c', 'my canvas', 800, 600)

# Create a histogram with some dummy data and draw it
data = np.random.randn(1000).astype(np.float32)
h = ROOT.TH1F('h', ';Gaussian process; N_{Events}', 30, -3, 3)
for x in data: h.Fill(x)
h.Draw('E')

# Fit a Gaussian function to the data
f = ROOT.TF1('f', '[0] * exp(-0.5 * ((x - [1]) / [2])**2)')
f.SetParameters(100, 0, 1)
h.Fit(f)

# Let's add some CMS style headline
label = ROOT.TLatex()
label.SetNDC(True)
label.SetTextSize(0.040)
label.DrawLatex(0.10, 0.92, '#bf{CMS Dummy Data}')
label.DrawLatex(0.58, 0.92, '#sqrt{s} = 13 TeV, L_{int} = 100 fb^{-1}')

# Save as png file and show interactively
c.SaveAs('dummy_data.png')
c.Draw()
```

```
 FCN=30.2937 FROM MIGRAD    STATUS=CONVERGED      67 CALLS          68 TOTAL
                     EDM=1.34686e-08    STRATEGY= 1      ERROR MATRIX ACCURATE
  EXT PARAMETER                                   STEP         FIRST
  NO.   NAME      VALUE            ERROR          SIZE      DERIVATIVE
   1  p0           8.09397e+01   3.19887e+00   7.10307e-03  -3.40988e-05
   2  p1          -3.46483e-03   3.10501e-02   8.47265e-05  -2.30742e-03
   3  p2           9.56532e-01   2.24141e-02   4.97399e-05   2.58872e-03
Info in <TCanvas::Print>: file dummy_data.png has been created
```

> ## Try it by yourself!
> Run the example code by yourself! In case the execution ends without displaying the plot on screen, you can run the script in interpreted mode with `python -i your_script.py`. That will keep the process alive after the plot is displayed.
{: .challenge}

![](../fig/dummy_data.png)



A guide to such advanced features of PyROOT can be found in the official manual at [https://root.cern/manual/python](https://root.cern/manual/python/). Feel free to investigate!

## Interoperability with NumPy arrays

There are many reasons, for example machine learning applications, to want to export your data in Python to NumPy arrays. This is easily possible with ROOT and is part of RDataFrame. The code snippets below show you how to do this conversion and how to move the data to typical tools in the Python ecosystem, e.g., `numpy` and `pandas`.

> ## numpy and pandas
> Have you installed `numpy` and `pandas` or are you on a system which has them available? Normally, you can just run `pip install --user numpy pandas` to install missing packages! Another option is searching in your system package manager, they are typically available on all platforms.
{: .callout}

### Convert data in ROOT files to numpy arrays

The conversion feature is attached to the class RDataFrame. We will not introduce you here to this way to process data with ROOT because the following section is dedicated to RDataFrame. For now, just keep in mind that you call `AsNumpy`! The data is returned as a dictionary of one-dimensional numpy arrays.

```python
# Read out the data as a dictionary of numpy arrays
import ROOT
df = ROOT.RDataFrame('TreeS', 'https://root.cern/files/tmva_class_example.root')
columns = ['var1', 'var2', 'var3', 'var4']
data = df.AsNumpy(columns)
print('var1: {}'.format(data['var1']))
```

```
var1: [-1.1436108   2.1434433  -0.44391322 ...  0.37746507 -2.072639 -0.09141494]
```

### Move the data to numpy or pandas

The data can be passed naturally to any method in the Python ecosystem which processes numpy arrays. Below is an example that computes the mean of each column.

```python
# Apply numpy methods
import numpy as np
print('Means: {}'.format([np.mean(data[c]).item() for c in columns]))
```

```
Means: [0.18244409561157227, 0.28425973653793335, 0.3789360225200653, 0.7712161540985107]
```

Another interesting usecase is moving the dataset directly to a pandas dataframe. You can use the output of `AsNumpy` directly as input to its constructor.

```python
# Convert to a pandas dataframe
import pandas
pdf = pandas.DataFrame(data)
print(pdf)
```

```
          var1      var2      var3      var4
0    -1.143611 -0.822373 -0.495426 -0.629427
1     2.143443 -0.018923  0.267030  1.267493
2    -0.443913  0.486827  0.139535  0.611483
3     0.281100 -0.347094 -0.240525  0.347208
4     0.604006  0.151232  0.964091  1.227711
...        ...       ...       ...       ...
5995 -0.040650 -0.154212 -0.097715  0.440331
5996  0.099931 -1.183759  0.034616  0.644502
5997  0.377465 -0.030945  1.166082  0.728614
5998 -2.072639 -0.635586 -0.747371 -1.285679
5999 -0.091415  0.221271  0.569032  1.386130

[6000 rows x 4 columns]
```

> ## Try it by yourself!
> The statements are very short, you can just copy paste them into the Python prompt. Feel free to investigate what you can do with `AsNumpy`! Further information can be found [here](https://root.cern/doc/master/df026__AsNumpyArrays_8py.html).
{: .challenge}

## ROOT in Jupyter notebooks

ROOT provides a deep integration with Jupyter notebooks. You can start a Jupyter notebook server including ROOT features with the following command:

```bash
root --notebook
```

Alternatively, you can go to [https://swan.cern.ch](https://swan.cern.ch), which provides Jupyter notebooks integrated with CERN's cloud storage as a web service. Note that you may have to visit [https://cernbox.cern.ch](https://cernbox.cern.ch) first at least once with your user account to create your CERNBox space!

### Python kernel

Jupyter is often use to edit Python code interactively. By creating a new notebook with a Python kernel, you will see something similar to the screenshot below and you can work interactively with Python in the browser!

<kbd>
<img src="../fig/jupyter_python.png">
</kbd>

### C++ kernel

ROOT provides a Jupyter C++ kernel, which behaves similarly to the Python kernel but for C++! Similar to the ROOT prompt, you can work interactively with C++ in the notebook. Just select the C++ kernel in the drop-down menu!

<kbd>
<img src="../fig/jupyter_cpp.png">
</kbd>

### JSROOT

Another feature of ROOT is the `%jsroot on` magic, which enables ROOT's JavaScript integration! This allows you to interact with the visualization such as you are used to it from the interactive graphics in the Python prompt.

Because it's JavaScript, we can also embed these plots easily in any website. You can find an interactive version of the plot from the top of this section at the bottom of the page. For example, you can zoom in, add grid lines or get detailed information about the data points, right here!

<kbd>
<img src="../fig/jupyter_jsroot.png">
</kbd>

<script src="https://root.cern/js/5.8.1/scripts/JSRootCore.min.js" type="text/javascript"></script>
<script type='text/javascript'>
 var filename = "../fig/gauss.root";
 JSROOT.OpenFile(filename, function(file) {
    file.ReadObject("c;1", function(obj) {
       JSROOT.draw("drawing", obj, "");
    });
 });
</script>
<div id="drawing" style="width: 800px; height:600px"></div>

> ## Try it by yourself!
> Either run Jupyter locally via `root --notebook` or go to [https://swan.cern.ch](https://swan.cern.ch) to try ROOT in a Jupyter notebook!
{: .challenge}

## More useful features

ROOT is made for HEP analysis and contains many other features that are useful in typical tasks, for example:

* [TEfficiency](https://root.cern.ch/doc/master/classTEfficiency.html), to handle histograms representing efficiencies and their uncertainties
* [THStack](https://root.cern.ch/doc/master/classTHStack.html), to stack histograms
* [TRatioPlot](https://root.cern.ch/doc/master/classTRatioPlot.html), to create ratio plots with the histograms on the top and the ratio with the correct uncertainties on the bottom
* And many more!

> ## Try using ROOT with interactive C++, compiled C++ and Python!
> Make yourself familiar with the different ways you can run an analysis with ROOT!


{% include links.md %}

