Core Result of *tmap*
############################
Although the output and usage of ``tmap`` could be multiple version, the core result of ``tmap`` could be simply summarized in three ways.

Here we will show each way with simple example. The code below is simplify version of `Flemish Gut Flora Project(FGFP) notebook <https://nbviewer.jupyter.org/github/GPZ-BIOINFO/tmap_notebook/blob/master/FGFP/FGFP_pipelines.ipynb>`_

First, we need to generate the graph.

.. code-block:: python

    from sklearn.preprocessing import MinMaxScaler
    from sklearn.cluster import DBSCAN
    from tmap.tda import mapper, Filter
    from tmap.tda.cover import Cover
    from tmap.tda.plot import Color
    from tmap.tda.metric import Metric
    from tmap.tda.utils import optimize_dbscan_eps
    from tmap.netx.SAFE import SAFE_batch, get_SAFE_summary
    from tmap.test import load_data
    from scipy.spatial.distance import squareform,pdist
    import pandas as pd
    import os

    # load taxa abundance data, sample metadata and precomputed distance matrix
    X = load_data.FGFP_genus_profile()
    metadata = load_data.FGFP_metadata_ready()
    dm = squareform(pdist(X,metric='braycurtis'))

    # TDA Step1. initiate a Mapper
    tm = mapper.Mapper(verbose=1)

    # TDA Step2. Projection
    metric = Metric(metric="precomputed")
    lens = [Filter.MDS(components=[0, 1], metric=metric, random_state=100)]
    projected_X = tm.filter(dm, lens=lens)

    # Step4. Covering, clustering & mapping
    eps = optimize_dbscan_eps(X, threshold=95)
    clusterer = DBSCAN(eps=eps, min_samples=3)
    cover = Cover(projected_data=MinMaxScaler().fit_transform(projected_X), resolution=50, overlap=0.75)
    graph = tm.map(data=X, cover=cover, clusterer=clusterer)
    print(graph.info())

    n_iter = 1000
    enriched_scores = SAFE_batch(graph,
                         metadata=pd.concat([X,metadata],axis=1),
                         n_iter=n_iter,
                         _mode = 'enrich')
    safe_summary = get_SAFE_summary(graph=graph,
                                metadata=pd.concat([X,metadata],axis=1),
                                safe_scores=enriched_scores,
                                n_iter=n_iter,
                                p_value=0.01)

With the graph and corresponding SAFE score, we could go further now.

Co-enrichment relations
========================

.. code-blokc:: python

    import itertools
    from tmap.netx.SAFE import get_significant_nodes
    from tmap.netx.coenrichment_analysis import *

    enriched_centroides, enriched_nodes = get_significant_nodes(graph=graph,
                                                                safe_scores=enriched_scores,
                                                                n_iter=n_iter,
                                                                pvalue=0.05,
                                                                r_neighbor=True
                                                                )
    corrected_fe_dis = pairwise_coenrichment(graph,
                                             safe_scores=enriched_scores,
                                             _pre_cal_enriched=enriched_centroides)

    cutoff_val = np.percentile(corrected_fe_dis.values.reshape(-1, 1), 0.5)
    edges = []
    for f1, f2 in itertools.combinations(corrected_fe_dis.index, 2):
        if corrected_fe_dis.loc[f1, f2] <= cutoff_val:
            edges.append((f1, f2, {'weight': 1 - corrected_fe_dis.loc[f1, f2]}))
            import networkx as nx
    G = nx.from_edgelist(edges)
    file_path = 'tmp_FGFP all genus and metadata.edges'
    nx.write_edgelist(G, file_path)

    edge_df = pd.read_csv(file_path, sep=' ', header=None)
    edge_df.index = range(edge_df.shape[0])

    all_nodes = list(set(list(edge_df.iloc[:, 0]) + list(edge_df.iloc[:, 1])))
    node_df = pd.DataFrame(index=all_nodes, columns=['cat'])
    for idx in range(edge_df.shape[0]):
        source_name = edge_df.iloc[idx, 0]
        end_name = edge_df.iloc[idx, 1]

        edge_df.loc[idx, 'weight'] = -np.log(corrected_fe_dis.loc[source_name, end_name])
    node_df.index.name = 'feature'
    edge_df = edge_df.drop([2, 3], axis=1)
    edge_df.columns = ['Source', 'End', 'weight']
    edge_df.to_csv(file_path, index=False, sep='\t')
    node_df.to_csv(file_path.replace('edge', 'node'), sep='\t')


Ranking of any features
========================

.. code-block:: python

    from plotly import tools
    import plotly.graph_objs as go
    import plotly

    fig = tools.make_subplots(1, 1)

    safe_summary_metadata = safe_summary.reindex(metadata.columns)
    sorted_df = safe_summary_metadata.sort_values('SAFE enriched score', ascending=False)

    fig.append_trace(go.Bar(x=sorted_df.loc[:, 'SAFE enriched score'],
                            y=sorted_df.index,
                            marker=dict(line=dict(width=1)),
                            orientation='h',
                            showlegend=False), 1, 1)

    fig.layout.yaxis.autorange = 'reversed'
    fig.layout.margin.l = 200
    fig.layout.height = 1500
    plotly.offline.plot(fig)


.. raw:: html

    <iframe src="_static/core_r_ranking.html" height="500px" width="100%"></iframe>

Ordination with SAFE scores
============================

.. code-block:: python

    import plotly
    from plotly import graph_objs as go
    from sklearn.decomposition import PCA
    from sklearn.preprocessing import MinMaxScaler

    metadata_category = pd.read_csv('FGFP_metadata_category.csv',index_col=0,sep=',')
    # FGFP_metadata_category.cs could be download at https://media.githubusercontent.com/media/GPZ-Bioinfo/tmap_notebook/master/FGFP/FGFP_metadata_category.csv

    metadata_category = metadata_category.reindex(enriched_scores.columns)
    metadata_category.loc[metadata_category.index.isin(X.columns), 'Category'] = 'Genus'

    pca = PCA()
    pca_result = pca.fit_transform(enriched_scores.T)

    top_metadata = list(safe_summary_metadata.sort_values('SAFE enriched score', ascending=False).index[:10])
    top_genus = list(safe_summary.sort_values('SAFE enriched score', ascending=False).index[:10])
    mx_scale = MinMaxScaler(feature_range=(10, 40)).fit(safe_summary.loc[:, ["SAFE enriched score"]])

    data = []
    for cat in set(metadata_category.loc[:, 'Category']):
        vals = safe_summary.loc[metadata_category.index[metadata_category.Category == cat], ["SAFE enriched score"]]
        data.append(go.Scatter(x=pca_result[metadata_category.Category == cat, 0],
                               y=pca_result[metadata_category.Category == cat, 1],
                               mode="markers",
                               # legendgroup=''
                               name=cat,
                               marker=dict(
                                           size=mx_scale.transform(vals),
                                           opacity=0.5),
                               text=metadata_category.index[metadata_category.Category == cat]))

    data.append(go.Scatter(x=pca_result[:, 0],
                           y=pca_result[:, 1],
                           # visible=False,
                           mode="text",
                           hoverinfo='none',
                           textposition="middle center",
                           name='name for searching',
                           showlegend=False,
                           textfont=dict(size=13),
                           text=''))

    traces_index = {trace.name: idx for idx, trace in enumerate(data) if trace['name'] in metadata_category.Category.unique()}
    reset_buttons = [dict(args=[{'text': [' ']}, {}, str(len(data) - 1)], label='ClearAll', method='update')]
    all_buttons = [dict(args=[{'text': [enriched_scores.columns]}, {}, str(len(data) - 1)], label='ShowAll', method='update')]

    updatemenus = [dict(active=-1,
                        buttons=[dict(args=['text', [_trace.text for _trace in data if _trace.name in traces_index] + [
                            [each if metadata_category.loc[each, 'Category'] == cat else ' ' for each in enriched_scores.columns]]
                                            ]
                                      , label=cat, method='restyle') for cat in metadata_category.Category.unique().tolist()] + reset_buttons + all_buttons,
                        ),
                   ]

    layout = dict(xaxis=dict(title="PC1({:.2f}%)".format(pca.explained_variance_ratio_[0] * 100)),
                  yaxis=dict(title="PC2({:.2f}%)".format(pca.explained_variance_ratio_[1] * 100)),
                  title="SAFE total enriched score based PCA (FGFP metadata+genus)",
                  font=dict(size=15),
                  hovermode='closest',
                  updatemenus=updatemenus)
    plotly.offline.plot(dict(data=data, layout=layout))

.. raw:: html

    <iframe src="_static/core_r_PCA.html" height="500px" width="100%"></iframe>
