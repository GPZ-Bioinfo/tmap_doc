FAQ
##########

1. **Why can't I find some samples in the resulting graph?**

  During the process of tmap, it will drop some samples which is lack of required number of neighbors. Usually, these samples is taken as outlier or noise which is need to be obsoleted. If you are worried about the number of samples retaining in the graph, you could use the function called `tmap.tda.utils.cover_ratio` to calculate the cover ratio. Usually, cover ratio higher than 80% is basic need for tmap result.

2. **There are too much pairs which is significant co-enriched found in the result of coenrich. How could I choose from them?**

  In the FGFP example, it could see that it do have too much pairs found in the result of coenrich. The scores or p-value of co-enrich pairs are calculated base on the graph. Significant coenrichment is defined as pairs of features that share similar distribution along with the graph by SAFE score.

  Currently, we don't have better way to filter it again and extract more significant pairs. We could use several graphs with different parameters to find significant pairs under individual situations. We could also use p-value of coenrich to rank all pairs and extract the top.

3. **Why would I choose tmap instead of other traditional ordination analysis?**

  Most significantly, *tmap* presents a TDA network for a microbiome dataset, which has the following advantages than traditional ordination techniques:

  * TDA network captures hidden patterns of a microbiome dataset in terms of nodes and edges between them, with nodes as a group of *highly similar samples*, and edges as a continuity between groups of samples.
  * TDA network enables users to explore the microbiome landscape of large cohorts, especially population-scale studies, more intuitively and effectively via *tmap* visualization.
  * TDA network enables users to test for species enrichment and metadata association using network-based statistical analysis, as implemented in *tmap*.

  You can go to :doc:`example` for detailed examples of application of *tmap* to different microbiome datasets.

4. **What is the difference between tmap and other Mapper algorithm?**

  We developed *tmap* mainly for population-scale microbiome data analysis, although it can also be used to analyze other high dimensional dataset. *tmap* is unique in its network-based statistical analysis for identifying driver species and for microbiome-wide association analysis. For more details, please see :doc:`how2work`.

5. **Why are some microbiome samples missing (discarded) in the final TDA network?**

  *tmap* uses **DBSCAN** for clustering by default, which will drop *noise* samples which are lack of the required number of neighbors. Usually, these samples are taken as outlier or noise in density-based clustering analysis.

  You can use ``tmap.tda.utils.cover_ratio`` to calculate how many samples were retained in the final TDA network. By using other clustering methods, such as **k-means clustering**, you can retain all the samples. *tmap* can take any other cluster method from *scikit-learn* (or with *scikit-learn* compatible APIs), although we recommend using the default **DBSCAN** method. For how to keep more samples in the final TDA network, you can see :doc:`param` for more details.


For further questions, and any suggestion, you are welcome to contact us via email: haokui.zhou@gmail.com.
