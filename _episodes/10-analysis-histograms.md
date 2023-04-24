---
title: "DELPHES with Scripts"
questions:
- "How to analyse DELPHES output files with scripts for ROOT?"
objectives:
- "Learning about using DELPHES with C and Python scripts"
keypoints:
- "Learning about using DELPHES with C and Python scripts"
- "More complex Python code can be written based on a separate analysis package, deteailed at [DelphesAnalysis](https://cp3.irmp.ucl.ac.be/projects/delphes/wiki/WorkBook/DelphesAnalysis)."
---

## DELPHES with basic scripts

Try to save the following script as file  "DelphesScript1.C" and then run it as:

```bash
root
gSystem->Load("libDelphes");
.x DelphesScript1.C("delphes_output.root");
```
The script:

```cpp
void DelphesScript1(const char *inputFile)
{
  gSystem->Load("libDelphes");

  // Create chain of root trees
  TChain chain("Delphes");
  chain.Add(inputFile);
  
  // Create object of class ExRootTreeReader
  ExRootTreeReader *treeReader = new ExRootTreeReader(&chain);
  Long64_t numberOfEntries = treeReader->GetEntries();
  
  // Get pointers to branches used in this analysis
  TClonesArray *branchJet = treeReader->UseBranch("Jet");
  TClonesArray *branchElectron = treeReader->UseBranch("Electron");
  
  // Book histograms
  TH1 *histJetPT = new TH1F("jet_pt", "jet P_{T}", 100, 0.0, 100.0);
  TH1 *histMass = new TH1F("mass", "M_{inv}(e_{1}, e_{2})", 100, 40.0, 140.0);

  // Loop over all events
  for(Int_t entry = 0; entry < numberOfEntries; ++entry)
  {
    // Load selected branches with data from specified event
    treeReader->ReadEntry(entry);
  
    // If event contains at least 1 jet
    if(branchJet->GetEntries() > 0)
    {
      // Take first jet
      Jet *jet = (Jet*) branchJet->At(0);
      
      // Plot jet transverse momentum
      histJetPT->Fill(jet->PT);
      
      // Print jet transverse momentum
      cout << jet->PT << endl;
    }

    Electron *elec1, *elec2;

    // If event contains at least 2 electrons
    if(branchElectron->GetEntries() > 1)
    {
      // Take first two electrons
      elec1 = (Electron *) branchElectron->At(0);
      elec2 = (Electron *) branchElectron->At(1);

      // Plot their invariant mass
      histMass->Fill(((elec1->P4()) + (elec2->P4())).M());
    }
  }

  // Show resulting histograms
  histJetPT->Draw();
}
```

## DELPHES with more advanced scripts

The following script can be run as:

```bash
root DelphesScript2.C'("delphes_output.root")'
```

The scripts "DelphesScript2.C" :

```cpp
#include "TH1.h"
#include "TSystem.h"

#ifdef __CLING__
R__LOAD_LIBRARY(libDelphes)
#include "classes/DelphesClasses.h"
#include "external/ExRootAnalysis/ExRootTreeReader.h"
#include "external/ExRootAnalysis/ExRootResult.h"
#endif

//------------------------------------------------------------------------------

struct MyPlots
{
  TH1 *fJetPT[2];
  TH1 *fMissingET;
  TH1 *fElectronPT;
};

//------------------------------------------------------------------------------

class ExRootResult;
class ExRootTreeReader;

//------------------------------------------------------------------------------

void BookHistograms(ExRootResult *result, MyPlots *plots)
{
  THStack *stack;
  TLegend *legend;
  TPaveText *comment;

  // book 2 histograms for PT of 1st and 2nd leading jets

  plots->fJetPT[0] = result->AddHist1D(
    "jet_pt_0", "leading jet P_{T}",
    "jet P_{T}, GeV/c", "number of jets",
    50, 0.0, 100.0);

  plots->fJetPT[1] = result->AddHist1D(
    "jet_pt_1", "2nd leading jet P_{T}",
    "jet P_{T}, GeV/c", "number of jets",
    50, 0.0, 100.0);

  plots->fJetPT[0]->SetLineColor(kRed);
  plots->fJetPT[1]->SetLineColor(kBlue);

  // book 1 stack of 2 histograms

  stack = result->AddHistStack("jet_pt_all", "1st and 2nd jets P_{T}");
  stack->Add(plots->fJetPT[0]);
  stack->Add(plots->fJetPT[1]);

  // book legend for stack of 2 histograms

  legend = result->AddLegend(0.72, 0.86, 0.98, 0.98);
  legend->AddEntry(plots->fJetPT[0], "leading jet", "l");
  legend->AddEntry(plots->fJetPT[1], "second jet", "l");

  // attach legend to stack (legend will be printed over stack in .eps file)

  result->Attach(stack, legend);

  // book more histograms

  plots->fElectronPT = result->AddHist1D(
    "electron_pt", "electron P_{T}",
    "electron P_{T}, GeV/c", "number of electrons",
    50, 0.0, 100.0);

  plots->fMissingET = result->AddHist1D(
    "missing_et", "Missing E_{T}",
    "Missing E_{T}, GeV", "number of events",
    60, 0.0, 30.0);

  // book general comment

  comment = result->AddComment(0.64, 0.86, 0.98, 0.98);
  comment->AddText("demonstration plot");
  comment->AddText("produced by Example2.C");

  // attach comment to single histograms

  result->Attach(plots->fJetPT[0], comment);
  result->Attach(plots->fJetPT[1], comment);
  result->Attach(plots->fElectronPT, comment);

  // show histogram statisics for MissingET
  plots->fMissingET->SetStats();
}

//------------------------------------------------------------------------------

void AnalyseEvents(ExRootTreeReader *treeReader, MyPlots *plots)
{
  TClonesArray *branchJet = treeReader->UseBranch("Jet");
  TClonesArray *branchElectron = treeReader->UseBranch("Electron");
  TClonesArray *branchMissingET = treeReader->UseBranch("MissingET");

  Long64_t allEntries = treeReader->GetEntries();

  cout << "** Chain contains " << allEntries << " events" << endl;

  Jet *jet[2];
  MissingET *met;
  Electron *electron;

  Long64_t entry;

  Int_t i;

  // Loop over all events
  for(entry = 0; entry < allEntries; ++entry)
  {
    // Load selected branches with data from specified event
    treeReader->ReadEntry(entry);

    // Analyse two leading jets
    if(branchJet->GetEntriesFast() >= 2)
    {
      jet[0] = (Jet*) branchJet->At(0);
      jet[1] = (Jet*) branchJet->At(1);

      plots->fJetPT[0]->Fill(jet[0]->PT);
      plots->fJetPT[1]->Fill(jet[1]->PT);
    }

    // Analyse missing ET
    if(branchMissingET->GetEntriesFast() > 0)
    {
      met = (MissingET*) branchMissingET->At(0);
      plots->fMissingET->Fill(met->MET);
    }

    // Loop over all electrons in event
    for(i = 0; i < branchElectron->GetEntriesFast(); ++i)
    {
      electron = (Electron*) branchElectron->At(i);
      plots->fElectronPT->Fill(electron->PT);
    }
  }
}

//------------------------------------------------------------------------------

void PrintHistograms(ExRootResult *result, MyPlots *plots)
{
  result->Print("png");
}

//------------------------------------------------------------------------------

void DelphesScript2(const char *inputFile)
{
  gSystem->Load("libDelphes");

  TChain *chain = new TChain("Delphes");
  chain->Add(inputFile);

  ExRootTreeReader *treeReader = new ExRootTreeReader(chain);
  ExRootResult *result = new ExRootResult();

  MyPlots *plots = new MyPlots;

  BookHistograms(result, plots);

  AnalyseEvents(treeReader, plots);

  PrintHistograms(result, plots);

  result->Write("results.root");

  cout << "** Exiting..." << endl;

  delete plots;
  delete result;
  delete treeReader;
  delete chain;
}
```
## DELPHES with python scripts

The following script can be run as:

```bash
python DelphesPythonScript.py delphes_output.root
```

The script "DelphesPythonScript.py" :

```python
#!/usr/bin/env python

import sys

import ROOT

try:
  input = raw_input
except:
  pass

if len(sys.argv) < 2:
  print(" Usage: Example1.py input_file")
  sys.exit(1)

ROOT.gSystem.Load("libDelphes")

try:
  ROOT.gInterpreter.Declare('#include "classes/DelphesClasses.h"')
  ROOT.gInterpreter.Declare('#include "external/ExRootAnalysis/ExRootTreeReader.h"')
except:
  pass

inputFile = sys.argv[1]

# Create chain of root trees
chain = ROOT.TChain("Delphes")
chain.Add(inputFile)

# Create object of class ExRootTreeReader
treeReader = ROOT.ExRootTreeReader(chain)
numberOfEntries = treeReader.GetEntries()

# Get pointers to branches used in this analysis
branchJet = treeReader.UseBranch("Jet")
branchElectron = treeReader.UseBranch("Electron")

# Book histograms
histJetPT = ROOT.TH1F("jet_pt", "jet P_{T}", 100, 0.0, 100.0)
histMass = ROOT.TH1F("mass", "M_{inv}(e_{1}, e_{2})", 100, 40.0, 140.0)

# Loop over all events
for entry in range(0, numberOfEntries):
  # Load selected branches with data from specified event
  treeReader.ReadEntry(entry)

  # If event contains at least 1 jet
  if branchJet.GetEntries() > 0:
    # Take first jet
    jet = branchJet.At(0)

    # Plot jet transverse momentum
    histJetPT.Fill(jet.PT)

    # Print jet transverse momentum
    print(jet.PT)

  # If event contains at least 2 electrons
  if branchElectron.GetEntries() > 1:
    # Take first two electrons
    elec1 = branchElectron.At(0)
    elec2 = branchElectron.At(1)

    # Plot their invariant mass
    histMass.Fill(((elec1.P4()) + (elec2.P4())).M())

# Show resulting histograms
histJetPT.Draw()

input("Press Enter to continue...")
```

More complex Python code can be written based on a separate analysis package, deteailed at [DelphesAnalysis](https://cp3.irmp.ucl.ac.be/projects/delphes/wiki/WorkBook/DelphesAnalysis).
