---
title: "Assignment 1 Hints"
questions:
- "How to complete Assignment 1  ?"
---
### Read and write file with arrays as branches and calculating invariant mass

The following script should help you understand how to read and write a file with arrays as branches, and some hints on calculating invariant mass of a particle that decays to two objects which are reconstructed. Do read the comments embedded within the code.

```C++
void myScript() {
    auto inputfile = TFile::Open("DYJetsToLL.root");

    auto inputtree = (TTree*)inputfile->Get("Events");
    long nentries = inputtree->GetEntries();

    const int nmuonmax = 50;// a number higher than maximum number of muons in any event in the file
    const int ntaumax = 50;// a number higher than maximum number of taus in any event in the file
    
    uint nmuon;
    uint ntau;
    float taupt[ntaumax];//these are arrays, that is they contain multiple numbers
    float tauphi[ntaumax];
    float taueta[ntaumax];
    float taumass[ntaumax];
    int taucharge[ntaumax];
    bool tauisotight[ntaumax]; //this is a boolean array

    TLorentzVector lvectau1, lvectau2, lvecZ;//Lorentz vectors
    float ditaumass;
    
    //Setting branches of input file and tree to the variables

    inputtree->SetBranchAddress("nMuon", &nmuon);
    inputtree->SetBranchAddress("nTau", &ntau);
    inputtree->SetBranchAddress("Tau_pt", taupt); //syntax for assigning branches to arrays
    inputtree->SetBranchAddress("Tau_phi", tauphi);
    inputtree->SetBranchAddress("Tau_eta", taueta);
    inputtree->SetBranchAddress("Tau_charge", taucharge);
    inputtree->SetBranchAddress("Tau_mass", taumass);
    inputtree->SetBranchAddress("Tau_idIsoTight", tauisotight); //this is a boolean array
    
    //Output file
  
    TFile* newFile( TFile::Open("newfile.root", "RECREATE") );
    auto newtree = std::make_unique<TTree>("newtree", "Events");
    newtree->Branch("nTau", &ntau, "nTau/I");//no need for the "nTau/I" as it takes it as default, but specifying for following cases
    newtree->Branch("Tau_pt", taupt, "Tau_pt[nTau]/F");//for arrays, need to specify in the third argument that it is an array (notice, no & sign in second arguments for array) of size ntau and store float variable


    //Output histograms
    auto hntau = new TH1F("hntau", "Number of Taus", 50, -0.5, 49.5);
    auto htaupt = new TH1F("htaupt", "Tau pT", 100, 0.0, 200.0);
    
    //Event loop
     std::cout<<"Entering event loop"<<std::endl;

    for (int i = 0; i < int(nentries); i++){
        inputtree->GetEntry(i);
        hntau->Fill(ntau);
        for (auto j = 0; j < ntau; j++){
            htaupt->Fill(taupt[j]);
        }
        if (ntau >= 2){
            lvectau1.SetPtEtaPhiM(taupt[0],taueta[0],tauphi[0],taumass[0]); //leading taus are the first two, that is with index 0,1
            lvectau2.SetPtEtaPhiM(taupt[1],taueta[1],tauphi[1],taumass[1]);
            lvecZ = lvectau1+lvectau2;
            ditaumass = lvecZ.Mag(); // invariant mass
        }
        newtree->Fill();

    }

    std::cout<<"End of event loop"<<std::endl;

    inputfile->Close();
    
    //Writing output file
    newFile->cd();
    newFile->Write();
    newFile->Close();
    newFile->Delete();
    std::cout<<"End of script"<<std::endl;
}
    
    
```
