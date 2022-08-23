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
    auto file = TFile::Open("tmva_class_example.root");

    auto newtree = (TTree*)file->Get("TreeS");
    auto entries = newtree->GetEntries();
    std::cout << "Entried in TreeS : " << entries << std::endl;
}
```

Scripts can be processed by passing them as argument to the `root` executable:

```bash
$ root myScript.C

root [0]
Processing myScript.C...
Entried in TreeS  : 6000
```

There are other ways to run scripts, try them.

### Plotting from file ith script
```cpp
void myScript_plot() {

    TH1F *h_var1 = new TH1F("h_var1", "var1", 100, -4.0, 4.0);
    TFile *file = new TFile("/Users/sar/Desktop/TestROOT/tmva_class_example.root");
    TTree *tree1 = (TTree*)file->Get("TreeS");

    float var1;

    tree1->SetBranchAddress("var1", &var1);

    int nentries = (int)tree1->GetEntries();

    for(int i=0; i < nentries; i++){

        tree1->GetEntry(i);
        h_var1->Fill(var1);

    }

    TCanvas *c1 = new TCanvas();
    h_var1->Draw();
}

```


### Examples of creating histogam:

```cpp
void scriptHist(){
    auto cnt_r_h=new TH1F("count_rate",
                "Count Rate;N_{Counts};# occurencies",
                100, // Number of Bins
                -0.5, // Lower X Boundary
                15.5); // Upper X Boundary

    auto mean_count=3.6f;
    TRandom3 rndgen;
    // simulate the measurements
    for (int imeas=0;imeas<400;imeas++)
        cnt_r_h->Fill(rndgen.Poisson(mean_count));

    auto c= new TCanvas();
    cnt_r_h->Draw();

    auto c_norm= new TCanvas();
    cnt_r_h->DrawNormalized();

    // Print summary
    cout << "Moments of Distribution:\n"
         << " - Mean     = " << cnt_r_h->GetMean() << " +- "
                             << cnt_r_h->GetMeanError() << "\n"
         << " - Std Dev  = " << cnt_r_h->GetStdDev() << " +- "
                             << cnt_r_h->GetStdDevError() << "\n"
         << " - Skewness = " << cnt_r_h->GetSkewness() << "\n"
         << " - Kurtosis = " << cnt_r_h->GetKurtosis() << "\n";
}
```
## Using graphs

```cpp

// Builds a graph with errors, displays it and saves it as
// image. First, include some header files
// (not necessary for Cling)

#include "TCanvas.h"
#include "TROOT.h"
#include "TGraphErrors.h"
#include "TF1.h"
#include "TLegend.h"
#include "TArrow.h"
#include "TLatex.h"

void macro1(){
    // The values and the errors on the Y axis
    const int n_points=10;
    double x_vals[n_points]=
            {1,2,3,4,5,6,7,8,9,10};
    double y_vals[n_points]=
            {6,12,14,20,22,24,35,45,44,53};
    double y_errs[n_points]=
            {5,5,4.7,4.5,4.2,5.1,2.9,4.1,4.8,5.43};

    // Instance of the graph
    TGraphErrors graph(n_points,x_vals,y_vals,nullptr,y_errs);
    graph.SetTitle("Measurement XYZ;lenght [cm];Arb.Units");

    // Make the plot estetically better
    graph.SetMarkerStyle(kOpenCircle);
    graph.SetMarkerColor(kBlue);
    graph.SetLineColor(kBlue);

    // The canvas on which we'll draw the graph
    auto  mycanvas = new TCanvas();

    // Draw the graph !
    graph.DrawClone("APE");

    // Define a linear function
    TF1 f("Linear law","[0]+x*[1]",.5,10.5);
    // Let's make the function line nicer
    f.SetLineColor(kRed); f.SetLineStyle(2);
    // Fit it to the graph and draw it
    graph.Fit(&f);
    f.DrawClone("Same");

    // Build and Draw a legend
    TLegend leg(.1,.7,.3,.9,"Lab. Lesson 1");
    leg.SetFillColor(0);
    graph.SetFillColor(0);
    leg.AddEntry(&graph,"Exp. Points");
    leg.AddEntry(&f,"Th. Law");
    leg.DrawClone("Same");

    // Draw an arrow on the canvas
    TArrow arrow(8,8,6.2,23,0.02,"|>");
    arrow.SetLineWidth(2);
    arrow.DrawClone();

    // Add some text to the plot
    TLatex text(8.2,7.5,"#splitline{Maximum}{Deviation}");
    text.DrawClone();

    mycanvas->Print("graph_with_law.pdf");
}

int main(){
    macro1();
    }

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
