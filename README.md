# Summary

The purpose of this code is to simulate energy deposits due to cascading of energy levels following neutron capture. 
This code was written for use in nuclear recoil calibration for dark matter detectors, 
but may be useful in other particle physics applications as well, including coherent elastic neutrino nucleus scattering (CE&nu;NS).
Currently, we use a constant acceleration model for the atom slowing down and calculation of the ionization energy.
We also use the Lindhard model for calculating the ionization,
but the output is complete enough to allow the user to choose their ionization yield model after simulation.
The code currently supports Neon, Argon, Silicon, and Germanium cascades
slowing down in a lattice of like material.

# Installation Instructions

This program is designed to run in a Unix-based system and has been tested on Ubuntu and MacOS.

## Dependencies

Required:
* CERN's ROOT, which has its own installation instructions here: [https://root.cern/install/](https://root.cern/install/). Intended compatible with all versions; known compatibile with version 6.
* gcc, which can be installed via the command line (ex: `sudo apt-get install gcc`). Currently compatible with version 4.4.7 or newer.

Optional:
* Anaconda or conda, as optionally used in the ROOT installation process: [https://www.anaconda.com/products/individual](https://www.anaconda.com/products/individual)
* A package for reading *.root files (for Python, one example of this is [uproot](https://pypi.org/project/uproot/) `pip install uproot` - if you choose to use uproot, ensure that you are using version 3.)

## Steps

1) [Download](https://github.com/villano-lab/nrCascadeSim/archive/master.zip) or clone (`git clone https://github.com/villano-lab/nrCascadeSim.git`) the master branch of this repository.
    * If downloaded as a zip, unzip the file (`unzip "nrCascadeSim-master.zip"`).
    * Download/clone to the location you want the final installation to be.
2) Install dependencies
3) Enter the new directory (`cd nrCascadeSim`)
4) Either add ROOT to the current enviornment or switch to an environment in which it is active (see [ROOT instalation instructions](https://root.cern/install/))
5) Run the command `make`

# Quick Start

1) If ROOT is not part of the current environment, switch to an appropriate environment (ex: `conda activate name_of_root_env`)
2) Switch to the appropriate directory (`cd /path/to/directory/nrCascadeSim/`)
3) Run `./realizeCascades -n desired-number-of-events -o /path/to/output/file levelfiles/name_of_levelfile.txt`

# Instructions for Use

The `./realizeCascades` command will run the simulation a specified number of times for a given input file. 
Currently, this is the only program in this package.
It is possible for the user to create additional programs based on teh libraries included,
and we may provide additional programs with the library in the future.

Note that `./realizeCascades` must be run from the top-level `nrCascadeSim` directory unless the user has added it to the path.
Also note that `ROOT` must be present in the current environment for this command to work.

## Arguments

All three of these arguments are required:
* `-n` specfies the total number of cascade events to be simulated. (example: `-n 100000` to simulate one hundred thousand events.)
* `-o` specifies the location of the output file. (example: `-o ~/output.root` to output to a file `output.root` in the home directory.)
* The main argument (no prefix) specifies the input file. (example: `levelfiles/Si28_ngam_all_cascades_rfmt_sorted.txt` to call a levelfile with all cascades for 28Si available.)
Making the full example: `./realizeCascades -n 100000 -o ~/output.root levelfiles/Si28_ngam_all_cascades_rfmt_sorted.txt` to simulate 100000 events for 28Si and output them to a file in the home directory.

(More detailed examples below.)

## Examples

There is also a directory `example-usecase` containing one example of how data can be used once generated in a jupyter notebook `Yields_and_Resolutions.ipynb`. 
This notebook serves to help visualize what the final data can look like as well as provide a few examples of how the data in the output root file can be accessed.
The processed data in this notebook is an example of what might be used for neutron capture-based calibration.

# Levelfile (Input) Format

The levelfile is a singular text file read by the program using regular expressions.
Each row in a levelfile corresponds to one cascade, 
which should include a relative weight for the probability of the cascade's occurence.
While it is helfpul to create columns that are easy for the user to read,
columns can be delineated by any number of spaces.

The general format of one row of an input file is:

weight Sn [..,E2,E1,E0] [..,tau2,tau1,inf]

Each portion of this row is described below.

## Brief Descriptions

``weight``: weight of this cascade realization. It will be normalized to unity with all other cascades

``Sn``: neutron separation energy (MeV), can include several isotopes if energy different. 

``[..,E2,E1,E0]``: ordered list of the energy levels traversed (keV), including the ground state

``[..,tau2,tau1,inf]``: ordered list of the lifetimes of the energy levels traversed (fs), it must be
the same length as the previous list.

## Full Descriptions

The first column is the probability of a cascade occuring. 
This probability can be in scientific notation or a "standard" decimal (0.000671 or 6.71e-04).
It should be weighted by the relative abundance of the material to other materials present in the same levelfile.
It should also be weighted by the cross-section for interactions and the probability of the particular energy levels being reached.
The sum of the probabilities must be less than or equal to one in order for the simulation to work properly.
If the sum is less than one, the simulation may skip generating some points in the output
&mdash; for example, when requesting 100 entries, if the total probability is 0.95, one would expect 95 entries on average &mdash; 
but the input cascades will still be at the correct proportions with respect to one another.
However, if the sum is greater than one, the simulation may not reach certain cascades at all &mdash;
for instance, if a file has 12 cascades, and the probabilities of the first 10 add up to 1, 
then the last two will never be generated.

The second column specifies the isotope after capture
(the isotope that is being captured *on* will be one less than what is listed).
This is used to help the program know what function to call.
The isotope is specified following the standard format ##Xx (ex: 29Si, 72Ge). 
Some levelfiles may have only one isotope, one element with multiple isotopes, or even multiple elements. 

The third column is the number of particles in the nucleus.
For example, for 72Ge, the third column would be 72.

The fourth column contains a series of energy levels. 
The neutron is assumed to be captured at rest, giving it an energy level of the separation energy;
this energy is *not* listed.
All other energy levels of the cascade, including the ground state, are listed in keV within brackets, separated by spaces, in descending order.
(The energy levels are written relative to the ground state; they are positive, and the ground state's energy is zero.)
So, an entry of \[0\] indicates the straight-to-ground case,
while an entry of [8104.8	2235.3	0] will capture to the 8104.8 keV energy level, decay to 2235.3 keV, and then finally go to the ground state.

The final column is formatted much as the forth one.
It contains the lifetimes of the energy levels in attoseconds \[as\],
or thousandths of femtoseconds.
Each entry should match up with an energy level;
that is, the first lifetime will correspond to the first energy level,
and the last lifetime will correspond to the lifetime of the ground state,
which is approximated as 1e+14 \[as\], or 1 \[ms\].
The lifetimes are again all contained within brackets and separated with spaces:
[24000	215000	1e+14]

# Root file (output) format

*Note: [ROOT](https://root.cern/install/) is needed to open these files.*

A file that contains the separate NR deposits, along with their Ionization deposits (INTERNAL: see
lab notebook N-MISC-16-001 pg 35).  In addition all of the exiting gamma energies and times should be listed.

The output files are *.root files and therefore cannot be read as text.
Instead, they need to be imported to a program to be read out.
One straightforward way of reading these files is with python and the [uproot](https://pypi.org/project/uproot/) package.

The *.root files store information in a tree-like structure. The top-most key in the output files will be `cascade` (there are no other top-level keys). Beneath this, the following keys exist:
* `n` - *1D Array* - Array denoting the number of energy levels in a given cascade. This includes intermediate levels and the ground state.
* `cid` - *1D Array* - Array of cascade IDs. The cascade ID is the number of the row in the levelfile which contains the cascade used. These count starting from zero.
* `Elev` - *Jagged Array* - **keV** - Array of energy level inputs. Each entry is an array of size `n`.
* `taus` - *Jagged Array* - **as** - Array of lifetime inputs. Each entry is an array of size `n`.
* `delE` - *Jagged Array* - **eV** - Array of energy deposits between energy levels. Each entry is an array of size `n - 1`. It contains the individual energy deposits, not the total energy deposit. If using a custom nonlinear ionization model, these are the best to operate on.
* `I` - *Jagged Array* - Array containing the ionization calculations for each energy deposit. Each entry is an array of size `n - 1`. This ionization is given in terms of a number of charges.
* `Ei` - *Jagged Array* - **eV** - Array of calculated ionization energy per step. These energies are conversions of `delE` to ionization energies. Each entry is an array of size `n - 1` containing the individual ionization energies. The Lindhard model is used here.
* `time` - *Jagged Array* - **as** - Array of the time spent at each energy level. Each entry is an array of size `n` containing individual times.
* `Eg` - *Jagged Array* - **eV** - Array of gamma energies. Each entry is an array of gamma energies, corresponding to an energy deposit.

The ordering of values in the arrays are consistent; that is, the nth entry of `n` corresponds to the nth entry of `cid`, the nth entry of `Elev`, and so on.
The length of each main array should be equal to the number of simulations; that is, 
if running 10000 events, `n` and `cid` will have lengths of 10000 and the jagged arrays will have first dimensions of length 10000.

![A visual outline of the structure of a ROOT output file named `file.root`. Everything is contained within a top-level key called `cascade`. Beneath `cascade` are several other keys, as described above.](output_structure.svg)

## Ionization

Ionization assumes the [Lindhard](https://www.osti.gov/biblio/4701226) model:

  Y = k*g(&epsilon;)/(1+kg(&epsilon;))  
  g(&epsilon;) = a\*&epsilon;<sup>&gamma;</sup> + b\*&epsilon;<sup>&omega;</sup> + &epsilon;  
  &epsilon;(E<sub>r</sub>) = 11.5E<sub>r</sub>\[keV\]Z<sup>-7/3</sup>

Using the accepted value for Silicon (*k = 0.143*) or Germanium (*k = 0.159*), whichever is
appropriate; *a = 3*; *b = 0.7*; *&gamma; = 0.15*; and *&omega; = 0.6*.