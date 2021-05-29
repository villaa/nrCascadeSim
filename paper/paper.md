---
title: '`nrCascadeSim` - A simulation tool for nuclear recoil cascades resulting from neutron capture'
bibliography: references.bib
tags:
  - C++
  - Simulation
  - Nuclear Physics
authors:
  - name: A.N. Villano
    affiliation: 1
    orcid: 0000-0002-3893-7259
  - name: Kitty Harris
    affiliation: 1
    orcid: 0000-0001-5406-8367
  - name: Staci Brown
    affiliation: 2
affiliations:
 - name: Department of Physics, University of Colorado Denver, Denver CO 80217, USA
   index: 1
 - name: Department of Applied Mathematics & Statistics, University of New Mexico, Albuquerque NM 87131, USA
   index: 2
date: 12 December 2020
nocite: '@*'
---

# Summary

Neutron capture-induced nuclear recoils have emerged as an important
tool for detector calibrations in direct dark matter detection and coherent elastic neutrino-nucleus scattering (CE$\mathrm{\nu}$NS).

`nrCascadeSim` is a C++ command-line tool for generating simulation data for energy deposits
resulting from neutron capture on pure materials. Presently, capture events within Silicon, Germanium, Neon, and Argon are
supported. While the software was developed for solid state detector calibration, it can be used
for any application which requires simulated neutron capture-induced nuclear recoil data.

A "cascade" occurs when a neutron becomes part of a nucleus.  The neutron can be captured to one
of many discrete energy levels, or states; if the energy level is nonzero (not the ground state),
then the state will eventually change so that it is zero.  This can happen either all at once or in
multiple steps &mdash; that is, the captured neutron may go from its state to the ground state, or
it may go to another state with lower energy that is not the ground state (provided that one
exists).  The cascade refers to the particular "path" of energy levels that a captured neutron
takes to get to the ground state from the neutron separation energy. Currently the code assumes
that the neutrons that enter the nuclear system have zero kinetic energy; this is a
good approximation for thermal neutrons because 0.0254\ eV (the average kinetic energy of a
thermal neutron) is small compared to most nuclear recoil energy scales, making it negligible.

`nrCascadeSim` models many of these cascades at once and saves the energies along with other
useful data to a single file. 
While there are tools, such as the GEANT4 [@Geant4] framework, that allow users to simulate neutron capture, 
existing tools are not built specifically for neutron capture-based nuclear recoils as NR Cascade Sim is and therefore uses some underlying assumptions that 
NR Cascade Sim does not. The main approximation used in GEANT4 that we avoid in NR Cascade Sim is that all recoils 
decay directly to the ground state. While this works for some applications, it is necessary to be more precise 
when an accurate spectrum of neutron capture-based recoils is needed for analyses such as calibration or noise subtraction.

# Models Used

When modeling deposits from neutron capture events, we want to look at the recoil of the nucleus
as a result of these cascades.  To determine how much energy is deposited, we must track how
much the nucleus slows down between steps of the cascade as well as *how* each state change
affects the nucleus' travel.  `nrCascadeSim` assumes a constant deceleration that results from the
nucleus colliding with other nearby nuclei.  This means that it must simulate, along with the
steps of the cascade, the time between each state &mdash; to calculate how much the nucleus slows
down &mdash; and the angle between the nucleus' momentum before a decay and the momentum boost
(gamma ray) resulting from the decay &mdash; to calculate the resulting momentum.  The time
between steps is simulated as an exponentially-decaying random variable based on the state's
half-life\footnote{It is most correct to use the half-life for the state given the state it will decay to. 
However, these are not generally well-known unless the branching ratios are well-known. 
If the ratios are well-known, then a correction can be made and incorporated into the input file.}, 
and the angle is simulated as having a uniform distribution on the surface of a sphere.
Cascade selection is weighted by isotope abundance and cross-section as well as the probability of
the energy level.  In existing levelfiles, energy levels are derived from [@Ge] for Germanium
and from [@Si] for Silicon.

The above process models the recoil energies, and the output gives both the total recoil energy
for a cascade as well as the energy per step.  For some applications, this may be the desired
output, or the user may already have a particular process they will use for converting this
energy to what they wish to measure.  However, we also include, for convenience, the ionization yield
and ionization energy of these recoils.  Ionization yield is a fraction that, when multiplied by the 
energy, gives the ionization energy, and ionization energy is the amount of energy that would be 
read out if an otherwise equivalent electron recoil were to occur. This calculation is useful because
many solid-state detectors read out the ionization energy for nuclear recoils. This ionization yield
assumes the Lindhard model [@lindhard].

Figure \ref{LindvSor_fig} compares the normalized frequencies of ionization energies from the Lindhard [@lindhard] 
model with the Sorensen [@sorensen] yield model, which is applied after the simulation using python, and applies 
detector resolution models applied to both. This figure demonstrates one example of user-applied analysis utilizing 
the energy deposits at each step instead of the ionization energy.

![An overlaid histogram showing an example use case in which points are generated and then multiple yield models and resolutions are applied.  The "Small Res (1/5)" histograms have Gaussians with 1/5 of the width of their counterparts. \label{LindvSor_fig}](SorVsLin_fig.pdf)

# Statement of Need

`nrCascadeSim` allows users to generate nuclear recoil simulations that reflect a variety of single-element detector setups.
The energy levels that the recoiling nuclei may pass between and their respective lifetimes are
customizable, and multiple isotopes of the same element can be present within the same simulation.
Pre-defined energy level files exist for Silicon and Germanium, which are constructed from the
data in [@abundances] and [@nudat2].  Output values include energy deposits at each step along each individual cascade, total
kinetic energy deposits, and ionization energy deposits, making them useful for a variety of
applications, including nuclear recoil calibrations for dark matter direct detection or coherent
neutrino detection (CE$\mathrm{\nu}$NS).


# Acknowledgements

This material is based upon work supported by the U.S. Department of Energy, Office of Science, Office of High Energy Physics (HEP) under Award Number DE-SC0021364.

# References
