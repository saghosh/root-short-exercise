---
title: "Introduction to DELPHES"
questions:
- "How to install and run DELPHES ?"
---

## DELPHES

Delphes framework for fast detector response simulation. The simulation includes a tracking system, embedded into a magnetic field, calorimeters and a muon system. The framework is interfaced to standard file formats (e.g. Les Houches Event File or HepMC) and outputs observables such as isolated leptons, missing transverse energy and collection of jets which can be used for dedicated analyses. The simulation of the detector response takes into account the effect of magnetic field, the granularity of the calorimeters and sub-detector resolutions. Visualisation of the final state particles is also built-in using the corresponding ROOT library.

## DELPHES documentation

The official webpage for DELPHES can be found [here](https://cp3.irmp.ucl.ac.be/projects/delphes). 
The publication detailing DELPHES can be found at [JHEP 02 (2014) 057](http://dx.doi.org/10.1007/JHEP02(2014)057) or [arXiv:1307.6346](http://arxiv.org/abs/1307.6346). 

## Installing DELPHES 

To successfully install Delphes the following prerequisite packages should be installed: [ROOT](https://saghosh.github.io/root-short-exercise/02-get-root/index.html) and [Tcl scripting language](http://www.tcl.tk/).

Download and build Delphes after sourcing ROOT:

```bash
$ wget http://cp3.irmp.ucl.ac.be/downloads/Delphes-3.5.0.tar.gz
$ tar -zxf Delphes-3.5.0.tar.gz
$ cd Delphes-3.5.0
$ make
```
## Running DELPHES 

Running Delphes can be done through the executables in the directory while passing arguments and paramteres. Eg.:
```bash
./DelphesHepMC3
```
But with parameters.
When running Delphes without parameters or when supplying an invalid command line, the following message will be shown:

```bash
 Usage: DelphesHepMC3 config_file output_file [input_file(s)]
 config_file - configuration file in Tcl format,
 output_file - output file in ROOT format,
 input_file(s) - input file(s) in HepMC format,
 with no input_file, or when input_file is -, read standard input.
 ```
 
Running Delphes with HepMC input files (example):

```bash
./DelphesHepMC3 cards/delphes_card_CMS.tcl output.root input.hepmc
 ```
 
 Try using with the file ttbar.lhe shared with you.
```bash
./DelphesLHEF cards/delphes_card_CMS.tcl delphes_output.root ttbar.lhe
```

## DELPHES produced output ROOT file 

To understand the content of the ROOT file produced, a handy reference is available at the Delphes webpage linked [here](https://cp3.irmp.ucl.ac.be/projects/delphes/wiki/WorkBook/RootTreeDescription).

To open the file with ROOT, you need to first load the Delphes library with which the contents of the file can be read:
```bash
$ root 
root[0] gSystem->Load("libDelphes");
```
Then you can open the Delhes output ROOT file as:
```bash
$ root 
root[0] gSystem->Load("libDelphes");
root[1] TFile *_file0 = TFile::Open("delphes_output.root")
root[3] TBrowser browser
```

## Visualising DELPHES Detector Simulation

Delphes also allows to viualise event by event detector simulation. Try it out:
```bash
$ make display
$ root -l examples/EventDisplay.C'("cards/delphes_card_CMS.tcl","delphes_output.root")'
```


## Running DELPHES directly with PYTHIA8

For this, you need a working PYTHIA8 installation (assuming you have that from the first part of the course).

Define an environment variable for the path to your PYTHIA installation directory:
```bash
$ export PYTHIA8=path_to_PYTHIA8_installation 
$ echo $PYTHIA8
```

Then, in your DELPHES directory, build the "DelphesPythia8" executable with the following command:
```bash
$ make HAS_PYTHIA8=true
```

Now, you can run a simple example for generating Pythia8 events within Delphes:
```bash
$ ./DelphesPythia8 cards/delphes_card_CMS.tcl examples/Pythia8/configNoLHE.cmnd delphes_pythia8.root
```
