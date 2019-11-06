---
interact_link: content/graph/network.ipynb
kernel_name: mmp_guide
has_widgets: false
title: 'Network analysis'
prev_page:
  url: /graph/convert.html
  title: 'Converting from GeoDataFrame to Graph and back'
next_page:
  url: /graph/centrality.html
  title: 'Multiple Centrality Assessment'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Network analysis

Graph analysis offers three modes, of which the first two are used within `momepy` (as per v0.1):
- node-based
    - value per node
- edge-based
    - value per edge
- network-based
    - single value per network



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import momepy
import geopandas as gpd
import matplotlib.pyplot as plt

```
</div>

</div>



We will use dataset included in `momepy` to load street network, and then we convert it to graph:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
streets = gpd.read_file(momepy.datasets.get_path('bubenec'), layer='streets')
graph = momepy.gdf_to_nx(streets)

```
</div>

</div>



## Node-based analysis

Once we have the graph, we can use momepy functions, like the one measuring clustering:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
graph = momepy.clustering(graph, name='clustering')

```
</div>

</div>



### Using sub-graph

Momepy includes local characters measured on the network within a certain radius from each node, like meshedness. The function will generate `ego_graph` for each node so that it might take a while for more extensive networks. Radius can be defined topologically:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
graph = momepy.meshedness(graph, radius=5, name='meshedness')

```
</div>

</div>



Or metrically, using distance which has been saved as an edge argument by `gdf_to_nx` (or any other weight).



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
graph = momepy.meshedness(graph, radius=400, name='meshedness400',
                          distance='mm_len')

```
</div>

</div>



Once we have finished the graph-based analysis, we can go back to `GeoPandas`. In this notebook, we are interested in nodes only:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
nodes = momepy.nx_to_gdf(graph, points=True, lines=False, spatial_weights=False)

```
</div>

</div>



Now we can plot our results in a standard way, or link them to other elements (using `get_node_id`).

Clustering:



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
nodes.plot(ax=ax, column='clustering', markersize=100, legend=True, cmap='viridis', scheme='quantiles')
streets.plot(ax=ax, color='lightgrey', alpha=0.5)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/graph/network_13_0.png)

</div>
</div>
</div>



Meshedness based on topological distance:



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
nodes.plot(ax=ax, column='meshedness', markersize=100, legend=True, cmap='viridis')
streets.plot(ax=ax, color='lightgrey', alpha=0.5)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/graph/network_15_0.png)

</div>
</div>
</div>



And meshedness based on 400 metres:



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
nodes.plot(ax=ax, column='meshedness400', markersize=100, legend=True, cmap='viridis')
streets.plot(ax=ax, color='lightgrey', alpha=0.5)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/graph/network_17_0.png)

</div>
</div>
</div>

