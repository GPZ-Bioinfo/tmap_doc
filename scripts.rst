Executable scripts of *tmap*
###############################

There are four scripts we have implemented.

Including below:
  * envfit_analysis.py
  * Network_generator.py
  * SAFE_analysis.py
  * SAFE_visualization.py

These four scripts could also be found below ``tmap/api``.

Here are a example for understanding the pipeline of these scripts. The test data could be found at ``tmap/test/test_data``

.. code-block:: bash

    envfit_analysis.py -I tmap/test/test_data/FGFP_genus_data.csv -M tmap/test/test_data/FGFP_metadata.tsv -O output/FGFP_envfit.csv -tn 'temp' --keep -v
    Network_generator.py -I tmap/test/test_data/FGFP_genus_data.csv -O output/FGFP.graph -v
    SAFE_analysis.py both -G output/FGFP.graph -M output/temp.envfit.metadata output/temp.envfit.data -P output/FGFP -i 1000 -p 0.05 --raw -v
    # generate 7 files
    SAFE_visualization.py ranking -G output/FGFP.graph -S2 output/FGFP_temp.envfit.metadata_enrich.csv output/FGFP_envfit.csv -O output/FGFP_ranking.html
    SAFE_visualization.py stratification -G output/FGFP.graph -S1 output/FGFP_raw_enrich -O output/FGFP_stratification.pdf --type pdf --width 1600 --height 1400
    SAFE_visualization.py ordination -S1 output/FGFP_raw_enrich -S2 output/FGFP_temp.envfit.data_enrich.csv output/FGFP_temp.envfit.metadata_enrich.csv -O output/FGFP_ordination.html

envfit_analysis
==================

For comparison, we have using ``rpy2`` which is a interface of **R** to perform envfit for user. It could also help the user to **fillna** and **one-hotted categorical variables** which could avoid SyntaxError at tmap analysis.

If you doesn't want to perform ``envfit`` analysis, you could also pass the params ``--dont_analysis`` to the script.

If you want to use this script to preprocess the metadata, ``--keep`` param is needed to pass or the intermediate file will be deleted.

Network_generator
===================

This script is mainly help user to generate the graph. Because the complex params of tmap, we recommend to watch the ``help`` carefully before using this script.

The generated graph is a ``pickle`` dict from pickle library.

SAFE_analysis
===================
This script is mainly help user to perform SAFE analysis based on generated graph. Because the complex params of SAFE, we recommend to watch the ``help`` carefully before using this script.

It will generate two kinds of files. If you pass ``--raw`` param which we also recommended to pass, you could get ``pickle`` dict contains the ``{data:{feature:{node:SAFE score}},param:{}}``. Normally, it will output SAFE score summary which is a normal csv file.


SAFE_visualization
===================
This script is mainly help user to visualized graph or SAFE score. But actually we recommend user to do it by yourself because currently it doesn't conclude all the params possible to use.

But for roughly visualization, it could help.

In addition, ``plotly`` is core visualization library we had used, it may raise some error if you lack some packages. (such as, `plotly-orca`_. which could not set it at setup.py)


.. _plotly-orca: https://github.com/plotly/orca
