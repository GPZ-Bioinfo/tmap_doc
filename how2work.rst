How *tmap* work
####################

**Topological data analysis** (**TDA**) is the application of topological analysis techniques to study datasets in high dimensional space, to capture topological properties of a dataset which are invariant to noise and scales. Currently, there are two common techniques of TDA: **persistent homology** and the **Mapper** algorithm. **tmap** is an implementation of the *Mapper* algorithm in Python, and is a TDA framework designed from microbiome data analysis. Because of the generality of the framework, *tmap* can also be used to analyze high dimensional datasets other than microbiome. [Ref1]_

A tmap workflow consists of three major steps, as illustrated in **Fig.1**.

a. The first step is to construct a TDA network.

  * project a microbiome dataset into a low dimensional space using dimension reduction techniques for a specified distance metric. This step uses filters/lenses functions for mapping the original data points to their low dimensional coordinates. For instance, the first two principal coordinates of PCoA can be used as filters.
  * After that, tmap performs topological covering and clustering of the projected data. The choice of different parameters for covering and clustering will result in different resolutions and details of topological patterns to be captured. Depending on the sample size and a desired resolution of microbiome variation, different sets of parameters can be tried and evaluated.
  * This step finally generates a TDA network for visualization and exploration of the microbiome using color maps on the network. Any target variable associated with the nodes can be used to color the network, such as metadata, taxon abundance, statistics or summary score.

b. In the second step, network enrichment analysis is performed to derive a SAFE score on each node for each target variable (see below for details).
c. The last step uses SAFE scores for network stratification, metadata association and ordination, and co-enrichment analysis.

.. image:: img/Fig1.png
    :alt: tmap workflow
    :align: center

Compared with other *Mapper* software, *tmap* was designed with the following advantageous features, especially for analyzing microbiome dataset:

1. Outputs from standard microbiome analysis pipelines, such as QIIME and USEARCH, can be used as direct inputs into *tmap*. Particularly, various precomputed beta-diversity distance matrices can be reused by *tmap*. Other customized and precomputed distance matrix, dimension reduction results, can also be used as input into *tmap* to save computation time.
2. APIs for each step in *tmap* have been carefully designed to allow for easy incorporation of off-the-shelf machine learning methods from other Python library, such as *scikit-learn*. Therefore, users can easily extend the *tmap* workflow using their customized functions, or integrate *tmap* into their own data analysis pipeline.
3. *tmap* comes with network visualization and analysis functions, including mapping of target variables for exploratory analysis and network enrichment for association analysis. These functions are unique in *tmap*, and are designed to help with microbiome analysis, for identifying driver species and microbiome-wide association analysis.

In the following sections, we will cover details on the ***Mapper*** algorithm and the ***SAFE*** algorithm, which are core techniques used in the *tmap* workflow. Having a solid understanding of these techniques can help to use *tmap* in its full potential, or to extend the framework with other off-the-shelf machine learning methods.

The *Mapper* algorithm
===================================

The *Mapper* algorithm consists of four basic steps, from defining a metric space of point cloud, projecting data points onto a low dimensional space, filtering and binning of projected data, to constructing a network representation, as illustrating in the following figure.

.. image:: img/how2work/basic_pipelines.jpg
    :alt: basic_pipelines
    :align: center
    :scale: 35 %

1. When we get a real world dataset, first we need to choose a distance metric to describe the distance between each pair of points. As in the above **figure A**, a three-dimensional Euclidean distance metric is used. Predefined metric can also be used by providing a ``precomputed`` distance matrix to the pipeline. The defined metric will be used by a **filter** function for data projection (or dimension reduction).

2. Next, *Mapper* projects the original data cloud to a low dimensional space, by using a **filter** function, which can be **PCA**, **MDS**, **t-SNE**, or any other dimension reduction method. In the above **fiugre B**, the **filter** function is just the *x-coordinate* of the original data cloud, in which the data points are colored by the filter function. Different filters can also be combined to form *lens*, or a low dimensional space. Usually, *Mapper* uses a two-dimensional space of projected data.

3. After **filtering**, the projected dataset is binned into groups (*hypercube*) of data points, by a **cover**, which is a collection of intervals along the filters. A **cover** can be specified via two resolution parameters (``resolution``: number of intervals, and ``overlap``: percentage of overlap between adjacent intervals). As in the above **figure C**, the original dataset is binned into overlapped intervals along the filter of the *x-coordinate* of the data cloud.

4. The final step of *Mapper* is clustering and network construction. Clustering is performed on each bin/hypercube of data points to identify clusters of data points in the **original data space**, rather than in the projected data space. Network construction is based on the clustering results: each cluster forms a node in the network, and edges are made between two nodes if they have common samples shared by their corresponding clusters. Therefore, the resulted TDA network, as shown in **figure D**, is a compressive network representation of the underlying 'data shape', which is an abstract shape of a human hand in this case. [Ref2]_

The SAFE algorithm
===================================

Once a TDA network has been constructed, network-based enrichment analysis can be performed for target variables, or properties of the studied data samples. *tmap* adopts **the spatial analysis of functional enrichment** (**SAFE**) for network enrichment analysis. This algorithm is originally developed as a systematic method for annotating biological networks and examining their functional associations. [Ref3]_

The **SAFE** algorithm takes the following steps to calculate an enrichment score (SAFE score) of each node in a network for a given target variable:

1. This algorithm starts with a constructed TDA network and a target variable (or multiple target variables) of metadata or taxa. For each node ``u`` in the network, **SAFE** defines a local neighborhood of ``u`` by identifying any other nodes that are closer than or equal to a maximum distance threshold ``d`` to ``u``. Node distance is measured via shortest path length between nodes. By default, **the maximum distance threshold** ``d`` is set to be equal to the *0.5th* percentile of all pairwise node distances in the network.

2. For each neighborhood, **SAFE** sums the values of neighbor nodes for a target variable as an observed neighborhood score ``S``. The score is then compared with the distribution of permuted neighborhood scores obtained by randomly shuffling the target variable among nodes in the network. The significance of enrichment ``P`` is determined as the probability that a random observation from the distribution will fall into the interval between the observed neighborhood score ``S`` and the largest value of the permuted scores. The neighborhood significance of enrichment ``P`` is converted into a neighborhood enrichment score ``O``, named SAFE score, as below (normalized to a range from 0 to 1):

.. math::

    P_{n} = \frac{m}{I}

where ``m`` is the number of times a shuffled value is greater than or equal to the observed value; ``I`` is the number of shuffles;

.. math::

    O_{n} = \frac{-log_{10}(max(P_{n},\frac{1}{I+1}))}{-log_{10}\frac{1}{I+1}}

where ``I`` is the number of shuffles; ``P`` is the neighborhood significance of enrichment of node ``n``; and ``O`` is the neighborhood enrichment score (**SAFE score**) of node ``n``. Random shuffle is performed independently for each target variable when there are more than one.

3. A node is considered to be significantly **enriched** under a p-value threshold of *0.05* (which can be specified in *tmap*), which can be translated to a threshold of SAFE score by:

.. math::

    O_{n} \ge \frac{-\log_{10} 0.05}{-\log_{10} \frac{1}{I+1}}

6. Filter and rank target variables using **number of significant nodes** or **sum of SAFE score of significant nodes** (for more details on SAFE score summary please see the following **SAFE summary in tmap**).

SAFE summary in *tmap*
===================================

After obtaining SAFE scores of each feature, different statistical metrics of enrichment can be calculated and summarized based on the SAFE algorithm. Using providing codes below could help you easily summarized the overall enrichment condition.


.. code-block:: python

  from tmap.netx.SAFE import get_SAFE_summary
  safe_summary = get_SAFE_summary(graph=graph, metadata=X, safe_scores=safe_scores,
                                  n_iter=n_iter, p_value=0.01)

The p-value threshold of ``0.01`` was set to select significant nodes for the calculation. The above ``n_iter`` was used to calculated lowest bound of SAFE scores and it need to be equivalent to the ``n_iter`` of providing safe_scores.

The key params ``n_iter`` could also be None. When you calculate the safe_scores, it will auto stodge used parameters into graph and its safe_scores. If you doesn't provide ``n_iter``, it will automatically take the last used parameters as the parameters.

``get_SAFE_summary`` will result a DataFrame with following headers.

==========================  ========
Header                      Meaning
==========================  ========
SAFE total score            Sum of SAFE enrichment scores of all nodes on the network map.
SAFE enriched score         Sum of SAFE enrichment scores of significantly enriched nodes.
enriched abundance ratio    the sum of the attribute’s abundance for enriched node divide by the sum of attribute ‘s abundance for all nodes
enriched SAFE score ratio   the sum of SAFE enrichment score for enriched nodes divide by the sum of SAFE enrichment score for all nodes
==========================  ========
