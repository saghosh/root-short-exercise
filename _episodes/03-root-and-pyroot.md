---
title: "ROOT and using ROOT prompt"
teaching: 10
exercises: 5
questions:
- "How to use a ROOT file and the ROOT prompt ?"
objectives:
- "Open a ROOT file"
- "Use ROOT prompt"
keypoints:
- "The choice of interactive C++, compiled C++ or Python is based on the use case!"
- "Usage of C++ code, compiled with optimization flags, may save you hours of computing time!"
- "PyROOT lets you use C++ from Python but offers many more advanced features to speed up your analysis in Python. Details about the dynamic Python bindings provided by PyROOT can be found on [https://root.cern/manual/python](https://root.cern/manual/python/)."
---
{: .testimonial}

## Interactive C++

One of the main features of ROOT is the possibility to use C++ interactively thanks to the C++ interpreter [Cling](https://root.cern/cling/). Cling lets you use C++ just like Python either from the prompt or in scripts.

### The ROOT prompt

By just typing `root` in the terminal you will enter the ROOT prompt. Like the Python prompt, the ROOT prompt is well suited to fast investigations.

```bash
$ root
root [0] 1+1
(int) 2
```

If you pass a file as argument to `root`, the file will be opened when entering the prompt and put in the variable `_file0`. ROOT typically comes with support for reading files remotely via HTTP (and [XRootD](https://xrootd.slac.stanford.edu/)), which we will use for the following example:

> ## No support for remote files?
> Although unlikely, your ROOT build may not be configured to support remote file access. In this case, you can just download the file with `curl -O https://root.cern/files/tmva_class_example.root` and point to your local file. No other changes required!
{: .solution}

```bash
$ root tmva_class_example.root

root [0]
Attaching file mva_class_example.root as _file0...
(TFile *) 0x555f82beca10

root [1] _file0->ls() // Show content of the file, all objects are accessible via the prompt!
TWebFile**              https://root.cern/files/tmva_class_example.root
 TWebFile*              https://root.cern/files/tmva_class_example.root
  KEY: TTree    TreeS;1 TreeS
  KEY: TTree    TreeB;1 TreeB

root [2] TreeS->GetEntries() // Number of events in the dataset

root [3] TreeS->Print() // Show dataset structure
******************************************************************************
*Tree    :TreeS     : TreeS                                                  *
*Entries :     6000 : Total =           98896 bytes  File  Size =      89768 *
*        :          : Tree compression factor =   1.00                       *
******************************************************************************
*Br    0 :var1      : var1/F                                                 *
*Entries :     6000 : Total  Size=      24641 bytes  One basket in memory    *
*Baskets :        0 : Basket Size=      32000 bytes  Compression=   1.00     *
*............................................................................*
*Br    1 :var2      : var2/F                                                 *
*Entries :     6000 : Total  Size=      24641 bytes  One basket in memory    *
*Baskets :        0 : Basket Size=      32000 bytes  Compression=   1.00     *
*............................................................................*
*Br    2 :var3      : var3/F                                                 *
*Entries :     6000 : Total  Size=      24641 bytes  One basket in memory    *
*Baskets :        0 : Basket Size=      32000 bytes  Compression=   1.00     *
*............................................................................*
*Br    3 :var4      : var4/F                                                 *
*Entries :     6000 : Total  Size=      24641 bytes  One basket in memory    *
*Baskets :        0 : Basket Size=      32000 bytes  Compression=   1.00     *
*............................................................................*

root [4] TreeS->Draw("var1") // Draw a histogram of the variable var1
```

<kbd>
<img src="../fig/trees_draw.png">
</kbd>

{% include links.md %}

## Investigating data in ROOT files

You have already seen the usage of `TTree::Draw` in the previous section. Such quick investigations of data in ROOT files are typical usecases which most analysts encounter on a daily basis. In the following you can learn about different ways to approach this task!

### Manually plotting with `TTree::Draw`

For quick studies on the raw data in a `TTree` on the command line, you can use `TTree::Draw` to make simple visualizations:

```bash
$ root https://root.cern/files/tmva_class_example.root

root [0]
Attaching file https://root.cern/files/tmva_class_example.root as _file0...
(TFile *) 0x558d7b54aa50
root [1] TreeS->Draw("var1") // just draw var1
Info in <TCanvas::MakeDefCanvas>:  created default TCanvas with name c1
root [2] TreeS->Draw("var1", "var2 > var1", "SAME") // draw var1 with the selection var2 > var1
(long long) 3222
```

<kbd>
<img src="../fig/trees_draw_2.png">
</kbd>

### The `TBrowser`

More convenient is using ROOT's tool for browsing ROOT files, the `TBrowser`. You can spawn the GUI directly from the ROOT prompt as shown below.

```bash
$ root https://root.cern/files/tmva_class_example.root

root [0]
Attaching file https://root.cern/files/tmva_class_example.root as _file0...
(TFile *) 0x557892a0ef10
root [1] TBrowser b
(TBrowser &) Name: Browser Title: ROOT Object Browser
```

### The `rootbrowse` executable

For convenience, ROOT provides the executable `rootbrowse`, which lets you open a `TBrowser` directly from the command line and display the files given as arguments!

```bash
$ rootbrowse https://root.cern/files/tmva_class_example.root
```

<kbd>
<img src="../fig/root_browser.png">
</kbd>

### Other ROOT executables

There are many small helpers shipped with ROOT, which let you operate on data quickly from the command line and solve typical day-to-day tasks with ROOT files.

**List of ROOT executables**
- `rootbrowse`: Open a ROOT file and a TBrowser
- `rootls`: List file content, tree branches, objects’ stats
- `rootcp`: Copy objects within a file or between files
- `rootdrawtree`: Simple analyses from the command line
- `rooteventselector`: Select branches, events, compression algorithms and extract slimmer trees
- `rootmkdir`: Creates a directory in a TFile
- `rootmv`: Move objects between files
- `rootprint`: Print objects in plots on files
- `rootrm`: Remove objects from files

```bash
$ rootls https://root.cern/files/tmva_class_example.root
TreeB  TreeS
```

```bash
$ rootls -l https://root.cern/files/tmva_class_example.root
TTree  Jan 19 14:25 2009 TreeB  "TreeB"
TTree  Jan 19 14:25 2009 TreeS  "TreeS"
```

```bash
$ rootls -t https://root.cern/files/tmva_class_example.root
TTree  Jan 19 14:25 2009 TreeB  "TreeB"
  var1    "var1/F"    0
  var2    "var2/F"    0
  var3    "var3/F"    0
  var4    "var4/F"    0
  weight  "weight/F"  0
  Cluster INCLUSIVE ranges:
   - # 0: [0, 5998]
   - # 1: [5999, 5999]
  The total number of clusters is 2
TTree  Jan 19 14:25 2009 TreeS  "TreeS"
  var1  "var1/F"  0
  var2  "var2/F"  0
  var3  "var3/F"  0
  var4  "var4/F"  0
  Cluster INCLUSIVE ranges:
   - # 0: [0, 5998]
   - # 1: [5999, 5999]
  The total number of clusters is 2
```


> ## Try it by yourself!
> Feel free to investigate the tools presented here!
{: .challenge}
