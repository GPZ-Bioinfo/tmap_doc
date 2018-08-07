# Visualizing and Exploring TDA Network

A TDA network is a powerful and compressive representation of a high dimensional complex dataset. Plotting the network provides a direct and insightful visualization of the underlying 'data shape', which can help to explore hidden patterns of the data. Nodes in the network correspond to groups of similar samples that have been clustered together. An edge between two nodes means that there are shared/common samples between the two nodes. Both large scale global structure and local group of nodes can be revealed in the network.

Furthermore, mapping a specified target variable onto the network, using a color scheme, can show how the target variable change along with the network. Therefore, plotting a TDA network and coloring the network with interested target variables is the first step to understand the analyzed data. In *tmap*, we can use the `Color` class to make a color scheme for a target variable. Then we can use the `show` function from `tda.plot` to visualize a TDA network, as demonstrated in the following codes:

```python
from sklearn.preprocessing import MinMaxScaler, StandardScaler
from sklearn import datasets
from sklearn.cluster import DBSCAN
from tda import mapper, filter
from tda.cover import Cover
from tda.plot import show, Color


iris = datasets.load_iris()
X = iris.data
y = iris.target

# Step1. initiate a Mapper
tm = mapper.Mapper(verbose=1)

# Step2. Projection
lens = [filter.MDS(components=[0, 1],random_state=100)]
projected_X = tm.filter(X, lens=lens)

# Step3. Covering, clustering & mapping
clusterer = DBSCAN(eps=0.75, min_samples=1)
cover = Cover(projected_data=MinMaxScaler().fit_transform(projected_X), resolution=20, overlap=0.75)
graph = tm.map(data=StandardScaler().fit_transform(X), cover=cover, clusterer=clusterer)
```

## Network Plotting Without a Target Variable

If we don't have a target variable, we could simply use `show` to visualize the network without any specified color map. By default, the network assigns all nodes with a default color of 'red'. This default can be changed via the `color` parameter in `show`.

```python
show(data=X, graph=graph, color='red', fig_size=(10, 10), node_size=15, mode='spring', strength=0.04)
```
![plot network default](img/param/vis_1.png)

## Network Plotting With a Target Variable

Plotting a TDA network with a target variable is very helpful to understand how the variable changes along the network and how it distinguishes between nodes. We can first make a color map for a chosen target variable, and then pass it together with a TDA graph to the `show` function for network plotting.

The following codes use a "categorical" color type for a categorical variable, such as different classes of samples. With a "categorical" color type, a node is assigned a color according to its most common class of its samples.

```python
color = Color(target=y, dtype="categorical")
show(data=X, graph=graph, color=color, fig_size=(10, 10), node_size=15, mode='spring', strength=0.04)
```
![plot network with a target 1](img/param/vis_2.png)

For a continuous target variable, we can use the "numerical" color type to make a color map. In this scenario, a node is assigned a color according to the mean values of its samples.

```python
color = Color(target=y, dtype="numerical")
show(data=X, graph=graph, color=color, fig_size=(10, 10), node_size=15, mode='spring', strength=0.04)
```
![plot network with a target 2](img/param/vis_3.png)

To go beyond network plotting and exploratory analysis, we can perform network-based statistical and enrichment analysis, which are demonstrated and explained in ['Network Statistical Analysis'](statistical.md).
