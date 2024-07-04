# SWGO_TANKSIM


This is a GEANT4 based simulation of different optical modules in the IceCube Neutrino Observatory in the South Pole modified for the SWGO Observatory. Currently available optical modules for IceCube are MDOM, LOM16, LOM18, PDOM, D-Egg, WOM (currently under development). Except for WOM, the optical module simulation was initially written by M. Unland and C. Lozano, and later they were modified by Waly M. Z. Karim. Currently, it can simulate muon flux from CCSN neutrinos, background radioactivity inside pressure vessel of MDOM, LOMs, and D-Egg, and photon wave with different Zenith Angle. It also accepts SNEWPY neutrino flux models in a python program called "merger" and use sntools to simulate the positron and electron flux from ibd and enees interactions.


## Quick Start Guide

-Make sure you have Geant4 installed in your local machine.  

### Prerequisite for compiling bulkice\_doumeki: 

- **Set up env.sh file (must do before compilation):**
  - Go to "/path\_to\_swgo\_tanksim/mdom/"
  - Open env.sh file in writing mode.
  - Set G4BUILD variable to your Geant4 installation directory
  - Save and exit.
  - Type ". env.sh" on your terminal to set up the environment
- **Set up build directory and compile:**
  - Go to "/path\_to\_swgo\_tanksim/mdom/"
  - Type "mkdir build"
  - Type "cd build"
  - Type "cmake **..**"
  - Type "make -jN" or "make"

This will generate a binary executable under the "/path\_to\_swgo\_tanksim/mdom/build/" directory.

- **Set Up the Output Folder**
  - Go to "/path\_to\_swgo\_tanksim/mdom/build/"
  - Type "mkdir output". (You can name your output folder however you want).
  - Changing the output folder to a different directory will require changing the code, and will be explained later along with how to change the output fields.
  - Note: by default, Output will be dumped in a folder in the build directory.

## Installation Errors
1. "fatal error: 'boost/property_tree/ptree.hpp' file not found"
	- install boost using your preferred package manager. (e.g. brew install boost)
2. "fatal error: 'bits/stdc++.h' file not found"
	- Follow the instructions found [here](https://apple.stackexchange.com/questions/148401/file-not-found-error-while-including-bits-stdc-h). 
	- Install gcc, download the bits file, and add it to the /Library/Developer/CommandLineTools/usr/include directory.

## Simulating Events

Events can be simulated in two different ways. One is using the Visualization driver OpenGL, where you can see the detector geometry and the particle interaction, but it has limited capabilities in terms of visualizing a large number of particles. Another way is running it in batch mode, where you can simulate a realistic number of events easily.

### Visualization

- Make sure you are in the build directory and the program is compiled and the executable is generated.
- Currently visualization mode is setup to inject a muon from a distance of 5m above tank. 
- An example run could be **"./bulkice_doumeki lom18 angle(deg)"**.

### Batch Mode
- Type **"./bulkice_doumeki [om model] [interaction channel] [depth index] [output folder] [run id]"**
- Available OM Models: [dom, mdom, lom16, lom18, pmt, degg]
- Available interaction channels: [ibd, enees, all, radioactivity]
- The depth index is irrelevant to SWGO, but it must be included for the simulation to run.
- Example run: "**./bulkice_doumeki lom18 muon 88 output 0**"


## Examples

### 1. Inject a muon at 45 degrees
- Visualization mode: **"./bulkice_doumeki lom18 45"**
- Batch mode: currently the batch mode for muons does not accept angle information in the command line. If you want to inject the muon at 45 deg, do the following:
	- vi "/path\_to\_swgo\_tanksim/mdom/src/OMSimPrimaryGenerator.cc"
	- In the "void OMSimPrimaryGeneratorAction::SetupMuons(G4int numParticles)" class, change the position and momentum. 
	- fParticleGun -> SetParticlePosition(G4ThreeVector(5*m, 0, 5 * m));
        - fParticleGun -> SetParticleMomentumDirection(G4ThreeVector(-1, 0, -1));

### 2. Muon injection in visual mode 
- Currently, muons in visual mode are set to be injected at the same spot for each run. You can randomize the place of injection (i.e. different spots around the tank) by doing the following:
	- vi "/path\_to\_swgo\_tanksim/mdom/src/OMSimPrimaryGeneratorAction.cc"
	- Go to "void OMSimPrimaryGeneratorAction::GenerateToVisualize()" class and change "r=0;" to "r=radData -> RandomGen(-1, 1);"
 	 
- If you want to change the distance at which the particle is injected: 
	- Edit the value for "distance" in the GenerateToVisualize() class. 
	- "distance=5;" will insert the particle 5*m above the tank for an angle of 0 deg. 


### 3. To change particles
- The simulation is currently set up to inject muon flux into the SWGO tank. Changing the particle is not trivial. The user must change the "/path\_to\_swgo\_tanksim/mdom/src/OMSimPrimaryGeneratorAction.cc" file to insert a new particle definition under the GenerateToVisualize() class.  


## Explanation of Input Parameters

**om model:**

Available Optical Module models in the simulation. For now, they are MDOM, WOM, DOM, DEGG, LOM16, and LOM18

**interaction channel:**

Possible interaction the input particles might have in the volume. If we are injecting positron flux, the possible interaction is **ibd** (Inverse Beta Decay). If we are injecting an electron flux, the possible interaction would be **enees** (Electron-Neutrino Electron Elastic Scattering). If someone wants both interactions to happen simultaneously, the channel would be **all**.

One could be interested in studying the angular sensitivity of the detector to optical photons. Therefore, they have to use the **opticalphoton** interaction channel. More on this later.

**depth index:**

The ice property varies with depth, and each depth is denoted by a depth index in the range [0, 108]. For example, 2.2Km depth has an index of 88. The depth indexes for each depth are given in this table:

**DEPTH INDEX TABLE**

**output folder**

By default, the output folder would be in the build directory of the simulation. If the user wants to dump the output data to a separate directory, they can provide the full path to the directory on the command line. An example with a full path would be: " **./bulkice\_doumeki mdom ibd 88 /home/waly/dump/ 0".**

**run id:**

Each run can be assigned an unique run id by the user. It is mainly to keep track of the files while running multiple runs in batch mode. One can set it to whatever integer number they like.

**distance from detector center:**

This option is available for simulating plane wave of photons only. It let's the user define a distance (in meters) from the center of the detector from where the photon wave will be generated.

**angle, start angle finish angle, increment:**

This is for simulating a plane wave of photons where the wave vector makes a specific angle with the detector surface.

If one wants to simulate a plane wave only at one angle, they might specify only the angle in degree.

If one simulates a range of angles with a specific step size, they might specify that start angle, final angle and the increment (step size). All are in degrees.

**Advanced Features:**

**\<To be available soon\>**


*NB: * I am still in the process of building and debugging it. Especially, running the simulation in parallel with sntools to have new positron and electron flux each time is still under development and will be updated soon. Nevertheless, please feel free to share any suggestion, criticism, or thoughts. My email: wkarim@u.rochester.edu slack: Waly M Z Karim  
