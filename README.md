# PyImfit

PyImfit is a Python wrapper for [Imfit](https://github.com/perwin/imfit), a C++-based program for fitting
2D models to scientific images. It is specialized for fitting astronomical images of galaxies, but can in 
principle be used to fit any 2D Numpy array of data. The underlying, Imfit-based library -- and thus the main
part of PyImfit's computation -- is multithreaded and naturally takes advantage of multiple CPU cores, and can
thus be very fast.

[Changelog](./CHANGELOG.md)

[comment]: <> ([![Build Status]&#40;https://travis-ci.org/perwin/pyimfit.svg?branch=master&#41;]&#40;https://travis-ci.org/perwin/pyimfit&#41;)

[comment]: <> (![PyImfit]&#40;https://github.com/perwin/pyimfit/workflows/PyImfit/badge.svg&#41;)
![PyImfit](https://github.com/perwin/pyimfit/actions/workflows/python-package.yml/badge.svg)
[![Documentation Status](https://readthedocs.org/projects/pyimfit/badge/?version=latest)](http://pyimfit.readthedocs.io/en/latest/?badge=latest)


## A Simple Example of Use

Assuming you want to fit an astronomical image (stored as a FITS file) named `galaxy.fits` using a model defined
in an Imfit configuration file named `config_galaxy.dat` (models can also be defined from within Python):

    from astropy.io import fits
    import pyimfit
    
    imageFile = "<path-to-FITS-file-directory>/galaxy.fits"
    imfitConfigFile = "<path-to-config-file-directory>/config_galaxy.dat"

    # read in image data, convert to proper double-precision, little-endian format
    image_data = pyimfit.FixImage(fits.getdata(imageFile))

    # construct model from config file (this can also be done programmatically within Python)
    model_desc = pyimfit.ModelDescription.load(imfitConfigFile)

    # create an Imfit object, using the previously created model configuration
    imfit_fitter = pyimfit.Imfit(model_desc)

    # load the image data and image characteristics (the specific values are
    # for a typical SDSS r-band image, where a sky-background value of 130.14
    # has already been subtracted), and then do a standard fit
    # (using default chi^2 statistics and Levenberg-Marquardt solver)
    imfit_fitter.fit(image_data, gain=4.725, read_noise=4.3, original_sky=130.14)
    
    # check the fit and print the resulting best-fit parameter values
    if imfit_fitter.fitConverged is True:
        print("Fit converged: chi^2 = {0}, reduced chi^2 = {1}".format(imfit_fitter.fitStatistic,
            imfit_fitter.reducedFitStatistic))
        bestfit_params = imfit_fitter.getRawParameters()
        print("Best-fit parameter values:", bestfit_params)


See the Jupyter notebook `pyfit_emcee.ipynb` in the `docs` subdirectory for
an example of using PyImfit with the Markov Chain Monte Carlo code [`emcee`](http://dfm.io/emcee/current/). (Online
version of notebook available [here](https://pyimfit.readthedocs.io/en/latest/pyimfit_emcee.html).)

Online documentation: [https://pyimfit.readthedocs.io/en/latest/](https://pyimfit.readthedocs.io/en/latest/).

Also useful: [Onine documentation for Imfit](https://imfit.readthedocs.io); and the main Imfit manual in PDF format:
[imfit_howto.pdf](http://www.mpe.mpg.de/~erwin/resources/imfit/imfit_howto.pdf)


## Requirements and Installation

PyImfit is designed to work with modern versions of Python 3 (3.10 or later)  on Linux and macOS; no support for Python 2 is planned.

### Installation from source

To build PyImfit from the Github source, you will need the following:

   * Most of the same external (C/C++) libraries that Imfit requires: specifically 
   [FFTW3](https://www.fftw.org) [version 3], [GNU Scientific Library](https://www.gnu.org/software/gsl/) [version 2.0
   or later!], and [NLopt](https://nlopt.readthedocs.io/en/latest/)
   
   * This Github repository

   * A reasonably modern C++ compiler -- e.g., GCC 4.8.1 or later, or any C++-11-aware version of 
   Clang++/LLVM that includes support for OpenMP. See below for special notes about using
   the Apple-built version of Clang++ that comes with Xcode for macOS.


#### Steps for building PyImfit from source:

1. Install necessary external libraries (FFTW3, GSL, NLopt)

    * These can be installed from source, or via package managers (e.g., Homebrew on macOS)
        
    * Note that version 2.0 or later of GSL is required! (For Ubuntu, this means
    the `libgsl-dev` package for Ubuntu 16.04 or later.)

2. Clone the PyImfit repository (use `--recurse-submodules` to ensure the Imfit repo is also downloaded)

        $ git clone --recurse-submodules https://github.com/IVChugunov/pyimfit.git

3. Install the Python package

    * **[macOS only:] First, specify a valid, OpenMP-compatible C++ compiler**
   
            $ export CC=<c++-compiler-name>; export CXX=<c++-compiler-name>
        
    (Note that you need to point CC and CXX to the *same*, Open-MP-compatible C++ compiler!
    This should not be necessary on a Linux system, assuming the default compiler is standard GCC.)
    
    * Versions of Apple's Clang compiler from Xcode 9 or later *can* compile OpenMP code, but you
      will need to also install the OpenMP library (e.g., `brew install libomp` if using Homebrew).
      See [here](https://iscinumpy.gitlab.io/post/omp-on-high-sierra/) for more details.
   
    * If you want to use PyImfit as is, then build and install it immediately:

            $ cd pyimfit
            $ python3 setup.py install

    * If you want to add your own functions, a few extra steps are needed:

        * First, add them to embedded Imfit (see Sec. 14 in the [Imfit manual](http://www.mpe.mpg.de/~erwin/resources/imfit/imfit_howto.pdf). When rebuilding Imfit, ensure that static library was built as well:

                $ cd imfit
                $ scons imfit && scons imfit-mcmc && scons makeimage && scons libimfit.a
                $ cp libimfit.a ../prebuilt/linux64/ && cp libimfit.a ../libimfit
                $ cd ..

        * Next, build and install it by yourself:

                $ python3 setup.py build
                $ pip install .


## Credits

PyImfit originated as [python-imfit](https://github.com/streeto/python-imfit), written by André Luiz de Amorim; 
the updated version is [by Peter Erwin](https://github.com/perwin/pyimfit).

(See [the Imfit manual](http://www.mpe.mpg.de/~erwin/resources/imfit/imfit_howto.pdf) for additional credits.)

This particular version includes minor modifications by Ilia V. Chugunov.


## License

PyImfit is licensed under version 3 of the GNU Public License.

