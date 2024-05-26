Installation
=============================================

DASPy can be installed through common methods such as Pip and Conda.

Install via Pip (recommanded)
----------------------------------------

::

    pip install git+https://github.com/HMZ-03/DASPy.git


Install via Conda
--------------------

::

    conda install -c hmz-03 daspy

If an error is reported, please try:

::

    conda update -n base -c conda-forge conda


Manual Installation
--------------------

1. Clone or download the DASPy repository from `Github <https://github.com/HMZ-03/DASPy?tab=readme-ov-file>`_;

2. Install the following dependency packages: numpy, scipy >=1.13, matplotlib, geographiclib, pyproj, h5py, segyio, nptdms;

3. Add the DASPy directory to your Python package path.