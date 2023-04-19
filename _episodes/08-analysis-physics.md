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

Download and build Delphes:

```bash
$ wget http://cp3.irmp.ucl.ac.be/downloads/Delphes-3.5.0.tar.gz
$ tar -zxf Delphes-3.5.0.tar.gz
$ cd Delphes-3.5.0
$ make
```
