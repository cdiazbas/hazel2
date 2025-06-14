# Hazel v2.0


[![github](https://img.shields.io/badge/GitHub-aasensio%2Fhazel2-blue.svg?style=flat)](https://github.com/aasensio/hazel2)
[![Build Status](https://travis-ci.org/aasensio/hazel2.svg?branch=master)](https://travis-ci.org/aasensio/hazel2)
[![Coverage Status](https://coveralls.io/repos/github/aasensio/hazel2/badge.svg?branch=master)](https://coveralls.io/github/aasensio/hazel2?branch=master)
[![license](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://github.com/aasensio/hazel2/blob/master/LICENSE)
[![ADS](https://img.shields.io/badge/ADS-2008ApJ...683..542A-red.svg)](http://adsabs.harvard.edu/abs/2008ApJ...683..542A)
[![arxiv](http://img.shields.io/badge/arXiv-0804.2695-orange.svg?style=flat)](https://arxiv.org/abs/0804.2695)

## Introduction


Hazel (an acronym for HAnle and ZEeman Light) is a computer program for the 
synthesis and inversion of Stokes profiles caused by the joint action of atomic 
level polarization and the Hanle and Zeeman effects. It is based on the quantum 
theory of spectral line polarization, which takes into account rigorously all the 
relevant physical mechanisms and ingredients: optical pumping, atomic level 
polarization, level crossings and repulsions, Zeeman, Paschen-Back and Hanle effects. 

The new Hazel v2.0 is a complete rewrite of the code, putting emphasis on its
usability. The code is now able to synthesize photospheric lines under the 
assumption of local thermodynamic equilibrium, chromospheric lines under
the multi-term approximation (like the He I multiplets) and a selection of
arbitrary systematic effects like telluric lines or fringes.

The code is written in Python with the most computationally heavy parts coded in Fortran 90. 
It can be controlled from a user-friendly configuration file, but it can also
be called programmatically. It can be used in synthesis mode for obtaining emerging
Stokes parameters from a given atmosphere. It can also be used in inversion mode
to infer the model parameters from a set of observed Stokes parameters.
Graphical front-ends are also provided.

## Features

- It can invert photospheric, chromospheric (mainly He I lines and the Ca II 854.2 nm line) and a variety of systematics (i.e., telluric lines).
- It can seamlessly handle 1D or 3D input/output files, making it very easy to invert large maps.
- Large supercomputers can be used to invert large maps. It scales practically linearly with the number of cores.
- It provides a programmatic access to the SIR synthesis module, which can be handy for many purposes.
- User-friendly API.


## Getting started

Read the [documentation](http://aasensio.github.io/hazel2) for getting 
details on how to use the code.

### Programmatically

For simple calculations, like synthesizing spectral lines in simple models,
Hazel v2.0 can be used in programmatic mode. For instance, let us generate a spectral
window in the near-infrared and synthesize the He I 10830 A line with some
parameters.

    
    import hazel
    mod = hazel.Model(working_mode='synthesis')
    mod.add_spectral({'Name': 'spec1', 'Wavelength': [10826, 10833, 150], 'topology': 'ch1'})
    mod.add_chromosphere({'Name': 'ch1', 'Spectral region': 'spec1', 'Height': 3.0, 'Line': '10830', 'Wavelength': [10826, 10833]})
    mod.setup()

    mod.atmospheres['ch1'].set_parameters([0.0,0.0,100.0*j,1.0,0.0,8.0,1.0,0.0,1.0])
    mod.synthesize()

As you see, we first generate the Hazel model for synthesis. We then create a spectral window with parameters
passed as a dictionary. Then we add a chromosphere and finally call `setup` to finalize the model setup.
We then change the model parameters of the chromosphere and synthesize the spectrum.
All the details of the dictionaries to pass and how to generate more complicated
atmospheres can be found in the documentation.

### With configuration file

Perhaps the easiest way of running Hazel v2.0 is through the human-friendly configuration
files described in the documentation.

#### Single pixel mode

Calculations in single-pixel mode (only one pixel synthesis/inversion) are very easy
to do. The following code uses a configuration file that can be found in the [examples](https://github.com/aasensio/hazel2/tree/master/examples) which uses `1d` inputs files. The following one carries out synthesis:

    mod = hazel.Model('conf_single_syn.ini')
    mod.open_output()
    mod.synthesize()
    mod.close_output()

and this one carries out the inversion:



    mod = hazel.Model('conf_single_inv.ini')
    mod.read_observation()
    mod.open_output()
    mod.invert()
    mod.close_output()

Summarizing, just create the model using the configuration file and then 
call synthesize/invert. Note that we open and close the output because
we want to generate a file with the synthetic profiles or the model
parameters of the inversion.

#### Serial mode

When many pixels need to be synthesized/inverted, one can pass appropriate input
multidimensional files that can deal with many pixels. They can be synthesized
in serial mode (using only one CPU) with the following code. It makes use of
an `iterator`, an object that  carries out the work for all the pixels in the
input files.



    iterator = hazel.Iterator(use_mpi=False)    
    mod = hazel.Model('conf_nonmpi_syn1d.ini')
    iterator.use_model(model=mod)
    iterator.run_all_pixels()

This case is slightly more complicated. We need to instantiate an iterator, telling it
not to use MPI. The, we instantiate the model and pass it to the iterator, which is
then used to run through all pixels.

#### MPI mode

When working with many pixels, many-core computers can be used. This will use a
master-slave approach, in which the master sends pixels to each one of the available
slaves to do the work. In this case, we tell the iterator to use MPI and act differently 
for the master (rank=0) or the salves (rank>0). Note that only the master reads the
configuration file, and it will be broadcasted to all slaves internally.



    iterator = hazel.Iterator(use_mpi=True)
    rank = iterator.get_rank()

    if (rank == 0):    
        mod = hazel.Model('conf_mpi_invh5.ini')
        iterator.use_model(model=mod)
    else:
        iterator.use_model()

    iterator.run_all_pixels()

## Building docs from sources

Install the following packages:

    conda install sphinx sphinx_rtd_theme numpydoc nbsphinx

And then compile them typing the following in the ``docs`` directory:

    ./compile

## Issues

MacOSX Catalina has some issues with the paths to the include files. Defining the following environment variables might help.

    export CFLAGS="-isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk"
    export CCFLAGS="-isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk"
    export CXXFLAGS="-isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk"
    export CPPFLAGS="-isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk"

## Attribution

Please, cite [Asensio Ramos, Trujillo Bueno & Landi Degl'Innocenti (2008)](https://ui.adsabs.harvard.edu/abs/2008ApJ...683..542A/abstract) if you find this code useful in your research. The BibTeX entry for the paper is:

    @ARTICLE{2008ApJ...683..542A,
       author = {{Asensio Ramos}, A. and {Trujillo Bueno}, J. and {Landi Degl'Innocenti}, E.},
        title = "{Advanced Forward Modeling and Inversion of Stokes Profiles Resulting from the Joint Action of the Hanle and Zeeman Effects}",
      journal = {ApJ},
         year = 2008,
        month = aug,
       volume = {683},
       number = {1},
        pages = {542-565},
          doi = {10.1086/589433},
           }
