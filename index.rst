.. tmap documentation master file, created by
   sphinx-quickstart on Thu Aug  2 14:10:53 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to tmap's documentation!
================================

What is *tmap*?
################

For large scale and integrative microbiome research, it is expected to apply advanced data mining techniques in microbiome data analysis.

Topological data analysis (TDA) provides a promising technique for analyzing large scale complex data. The most popular *Mapper* algorithm is effective in distilling data-shape from high dimensional data, and provides a compressive network representation for pattern discovery and statistical analysis.

**tmap** is a topological data analysis framework implementing the TDA *Mapper* algorithm for population-scale microbiome data analysis. We developed **tmap** to enable easy adoption of TDA in microbiome data analysis pipeline, providing network-based statistical methods for enterotype analysis, driver species identification, and microbiome-wide association analysis of host meta-data.

How to Install *tmap*?
########################

To install tmap, run::

    # (recommend)
    git clone https://github.com/GPZ-Bioinfo/tmap.git
    cd tmap
    python setup.py install
    # For some dependency problems. please install following packages.
    pip install scikit-bio
    R -e "install.packages('vegan',repo='http://cran.rstudio.com/')"

Or using pip::

    pip install tmap  # it may not the latest version, so be carefull for it.

If you encounter any error like ``Import error: tkinter``, you need to run ``sudo apt install python-tk`` or ``sudo apt install python3-tk``.

For more convenient usage, we implement some executable scripts which will automatically build upon ``$PATH``. For more information about these scripts, you could see.

Contents
##########

.. toctree::
   :maxdepth: 1

   basic.rst
   param.rst
   vis.rst
   statistical.rst
   how2work.rst
   example.rst
   scripts.rst
   api.rst
   reference.rst
   FAQ.rst

Indices and tables
####################

* :ref:`genindex`
* :ref:`search`

*tmap* Quick Guides
########################

1. You can read the :doc:`Basic Usage of tmap<basic>` for general use of *tmap*.
2. Or follow the :doc:`Microbiome examples<example>` for using *tmap* in microbiome analysis.

*tmap* Publication
########################
