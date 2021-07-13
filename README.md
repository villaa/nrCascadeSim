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
* CERN's ROOT, which has its own installation instructions here: [https://root.cern/install/](https://root.cern/install/). Intended compatible with all versions; known compatible with version 6.
* gcc, which can be installed via the command line (ex: `sudo apt-get install gcc`). Currently compatible with version 4.4.7 or newer.

Optional:
* Anaconda or Conda, as optionally used in the ROOT installation process: [https://www.anaconda.com/products/individual](https://www.anaconda.com/products/individual)
* A package for reading *.root files (for Python, one example of this is [uproot](https://pypi.org/project/uproot/) `pip install uproot` - if you choose to use uproot, ensure that you are using version 3.)

## Steps

1) [Download](https://github.com/villano-lab/nrCascadeSim/archive/master.zip) or clone (`git clone https://github.com/villano-lab/nrCascadeSim.git`) the master branch of this repository
    * If downloaded as a zip, unzip the file (`unzip "nrCascadeSim-master.zip"`)
    * Download/clone to the location you want the final installation to be
2) Install dependencies
3) Enter the new directory (`cd nrCascadeSim`)
4) Either add ROOT to the current environment or switch to an environment in which it is active (see [ROOT installation instructions](https://root.cern/install/))
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
* `-n` specifies the total number of cascade events to be simulated. (example: `-n 100000` to simulate one hundred thousand events.)
* `-o` specifies the location of the output file. (example: `-o ~/output.root` to output to a file `output.root` in the home directory.)
* The main argument (no prefix) specifies the input file. (example: `levelfiles/Si28_ngam_all_cascades_rfmt_sorted.txt` to call a levelfile with all cascades for 28Si available.)

This makes the full example `./realizeCascades -n 100000 -o ~/output.root levelfiles/Si28_ngam_all_cascades_rfmt_sorted.txt` to simulate 100000 events for 28Si and output them to a file in the home directory.

## Examples

There is also a directory `example-usecase` containing one example of how data can be used once generated in a jupyter notebook `Yields_and_Resolutions.ipynb`. 
This notebook serves to help visualize what the final data can look like as well as provide a few examples of how the data in the output root file can be accessed.
The processed data in this notebook is an example of what might be used for neutron capture-based calibration.

# Levelfile (Input) Format

The levelfile is a singular text file read by the program using regular expressions.
Each row in a levelfile corresponds to one possible cascade, 
which should include a relative weight for the probability of the cascade's occurrence.
While it is helpful to create columns that are easy for the user to read,
columns can be delineated by any number of spaces.

The general format of one row of an input file is:

`weight isotope A [..,E2,E1,E0] [..,tau2,tau1,inf]`

Each portion of this row is described below.

| `Name`    | `Format`              | Description   |
| ---       | ---                   | ---           |
|`weight`   | `#.##` or `#e+/-##`   | The probability of this cascade occuring, normalized to unity with all other cascades. This variable includes a weight for the isotope's abundance compared to other isotopes listed within the same levelfile. (If only one isotope is present within the levelfile, the abundance weight is not needed.) Weights can be given in decimal form or scientific notation (e.g. 0.000671 or 6.71e-04). |
| `isotope` | `##Xx`                | The isotope of the nucleus *after* capture. (For example, if 28Si is present, it will become 29Si, so 29Si should be listed.) This should be formatted as two numbers, one capital letter, and one lower-case letter (e.g. 29Si, 74Ge).|
| `A`       | `##`                  | The number of particles in the nucleus after capture. This should match the first two digits of `isotope`. For example, if `isotope` is 72Ge, `A` should be 72. |
| `energies`| `[... E2 E1 0]`       | An ordered list of the energy levels traversed (keV), including the ground state (0 keV), separated by spaces. These should be in the decreasing order, the order in which the nucleus will go through the states. Do not include the separation energy to account for the initial unbound state before capture; this is already assumed.|
| `lifetimes`| `[... tau2 tau1 inf]`| An ordered list of the lifetimes of the energy levels traversed (as), separated by spaces. It must be the same length as the list of energies, and the lifetimes should be in the same order as the energies. The last entry is `100000000000000.0` (1e+14 as, or 1 ms), which is effectively infinite on the timescale of the simulation, to indicate that the state is stable at the ground state.|

## On Weights

The sum of the probabilities must be less than or equal to one in order for the simulation to 
work properly. If the sum is less than one, the simulation may skip generating some points in 
the output &mdash; for example, when requesting 100 entries, if the total probability is 0.95, 
one would expect 95 entries on average &mdash; but the input cascades will still be at the 
correct proportions with respect to one another. If the sum is greater than one, the simulation 
may not reach certain cascades at all &mdash; for instance, if a file has 12 cascades, and the 
probabilities of the first 10 add up to 1, then the last two will never be generated.

### **An example for calculating weights:**

A detector has three isotopes, which become 29Si, 30Si, and 31Si after capture. 
The abundances within the detector are 60%, 30%, and 10%, respectively. 
Each has three possible cascades we want to model, which we list below in our (incomplete) draft of the levelfile:

```
weight? 29Si 29 [0]         [100000000000000.0]
weight? 29Si 29 [5000 0]    [0.84   100000000000000.0]
weight? 29Si 29 [3000 0]    [0.5    100000000000000.0]
weight? 30Si 30 [0]         [100000000000000.0]
weight? 30Si 30 [4000 0]    [1      100000000000000.0]
weight? 30Si 30 [2000 0]    [0.15   100000000000000.0]
weight? 31Si 31 [0]         [100000000000000.0]
weight? 31Si 31 [4999 0]    [0.15   100000000000000.0]
weight? 31Si 31 [540  0]    [.954   100000000000000.0]
```

Let's say the probabilities of the cascade occurring *within the respective isotopes* are as below:

| 29Si: | `[0]` | `[5000    0]` | `[3000    0]` | 
| ---   | ---   | ---           |           --- |
|       | 0.35  | 0.5           | 0.15          | 


| 30Si: | `[0]` | `[4000    0]` | `[2000    0]` | 
| ---   | ---   | ---           | ---           |
|       | 0.8   | 0.1           | 0.1           | 


| 39Si: | `[0]` | `[4999    0]` | `[540     0]` |
| ---   | ---   | ---           | ---           |
|       | 0.2   | 0.3           | 0.5           |

Then the relative probabilities *within the simulation* are:

| 29Si: | `[0]` | `[5000    0]` | `[3000    0]` | 
| ---   | ---   | ---           |           --- |
|       | 0.21  | 0.3           | 0.09          | 


| 30Si: | `[0]` | `[4000    0]` | `[2000    0]` | 
| ---   | ---   | ---           | ---           |
|       | 0.24  | 0.03          | 0.03          | 


| 39Si: | `[0]` | `[4999    0]` | `[540     0]` |
| ---   | ---   | ---           | ---           |
|       | 0.02  | 0.03          | 0.05          |

Making our completed levelfile:

```
0.21    29Si 29 [0]         [100000000000000.0]
0.30    29Si 29 [5000 0]    [0.84   100000000000000.0]
0.09    29Si 29 [3000 0]    [0.5    100000000000000.0]
0.24    30Si 30 [0]         [100000000000000.0]
0.03    30Si 30 [4000 0]    [1      100000000000000.0]
0.03    30Si 30 [2000 0]    [0.15   100000000000000.0]
0.02    31Si 31 [0]         [100000000000000.0]
0.03    31Si 31 [4999 0]    [0.15   100000000000000.0]
0.05    31Si 31 [540  0]    [.954   100000000000000.0]
```

## On Energies and Lifetimes

In the following levelfile row, the nth lifetime entry corresponds to the nth energy level entry.

```
0.30    29Si 29 [5000 4000 3000 2000 1000 0]    [0.84 0.95 1.35 0.03 0.11 100000000000000.0]
```
Therefore, the program reads this as:

| Energy level: | 5000 keV  | 4000 keV  | 3000 keV  | 2000 keV  | 1000 keV  |
| ---           | ---       | ---       | ---       | ---       | ---       |
| **Lifetime:** | 0.84 as   | 0.95 as   | 1.35 as   | 0.03 as   | 0.11 as   |


# Root file (output) format

*Note: [ROOT](https://root.cern/install/) is needed to open these files.*

A file that contains the separate NR deposits, along with their Ionization deposits (INTERNAL: see
lab notebook N-MISC-16-001 pg 35).  In addition all of the exiting gamma energies and times should be listed.

The output files are *.root files and therefore cannot be read as text.
Instead, they need to be imported to a program to be read out.
One straightforward way of reading these files is with python and the [uproot](https://pypi.org/project/uproot/) package.

The *.root files store information in a tree-like structure. The top-most key in the output files will be `cascade` (there are no other top-level keys). Beneath this, the following keys exist:  

| `Name`    | *Shape*       | **Units** | Description   |
| ---       | ---           | ---       | ---           |
| `n`       | *1D Array*    | N/A       | Array denoting the number of energy levels in a given cascade. This includes intermediate levels and the ground state. |
| `cid`     | *1D Array*    | N/A       | Array of cascade IDs. The cascade ID is the number of the row in the levelfile which contains the cascade used. These count starting from zero. |
| `Elev`    | *Jagged Array*| **keV**   | Array of energy level inputs. Each entry is an array of size `n`. |
| `taus`    | *Jagged Array*| **as**    | Array of lifetime inputs. Each entry is an array of size `n`. |
| `delE`    | *Jagged Array*| **eV**    | Array of energy deposits between energy levels. Each entry is an array of size `n - 1`. It contains the individual energy deposits, not the total energy deposit. If using a custom nonlinear ionization model, these are the best to operate on. |
| `I`       | *Jagged Array*| None      | Array containing the ionization calculations for each energy deposit. Each entry is an array of size `n - 1`. This ionization is given in terms of a number of charges. |
| `Ei`      | *Jagged Array*| **eV**    | Array of calculated ionization energy per step. These energies are conversions of `delE` to ionization energies. Each entry is an array of size `n - 1` containing the individual ionization energies. The Lindhard model is used here. |
| `time`    | *Jagged Array*| **as**    | Array of the time spent at each energy level. Each entry is an array of size `n` containing individual times. |
| `Eg`      | *Jagged Array*| **MeV**    | Array of gamma energies. Each entry is an array of gamma energies, corresponding to an energy deposit. |

The ordering of values in the arrays are consistent; that is, the nth entry of `n` corresponds to the nth entry of `cid`, the nth entry of `Elev`, and so on.
The length of each main array should be equal to the number of simulations; that is, 
if running 10000 events, `n` and `cid` will have lengths of 10000 and the jagged arrays will have first dimensions of length 10000.
