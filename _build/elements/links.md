---
interact_link: content/elements/links.ipynb
kernel_name: mmp_guide
has_widgets: false
title: 'Linking elements together'
prev_page:
  url: /elements/blocks.html
  title: 'Tessellation-based blocks'
next_page:
  url: /simple/simple.html
  title: 'Calculating simple characters'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Linking elements together

As explained in the Data Structure chapter, `momepy` relies on links between different morphological elements. Each element needs ID, and each of the small-scale elements also needs to know the ID of the relevant higher-scale element. The case of block ID is explained in the previous chapter, `momepy.Blocks` generates it together with blocks gdf.

## Getting the ID of the street network

This notebook will explore how to link street network, both nodes and edges, to buildings and tessellation.

### Edges

For linking street network edges to buildings (or tessellation or other elements), `momepy` offers `momepy.get_network_id`. It simply returns a `Series` of network IDs for analysed gdf.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import momepy
import geopandas as gpd
import matplotlib.pyplot as plt

```
</div>

</div>



For illustration, we can use `bubenec` dataset embedded in `momepy`.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
path = momepy.datasets.get_path('bubenec')
buildings = gpd.read_file(path, layer='buildings')
streets = gpd.read_file(path, layer='streets')
tessellation = gpd.read_file(path, layer='tessellation')

```
</div>

</div>



First, we have to be sure that streets segments have their unique IDs.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
streets['nID'] = momepy.unique_id(streets)

```
</div>

</div>



Then we can link it to buildings. The only argument we might want to look at is `min_size`, which should be a value such that if you build a box centred in each building centroid with edges of size `2 * min_size`, you know a priori that at least one segment is intersected with the box. You can see it as a sort of tolerance.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
buildings['nID'] = momepy.get_network_id(buildings, streets,
                                         'nID', min_size=100)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
Snapping: 100%|██████████| 144/144 [00:00<00:00, 2232.66it/s]```
</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
Generating centroids...
Generating rtree...
```
</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```

```
</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
buildings.plot(ax=ax, column='nID', categorical=True, cmap='tab20b')
streets.plot(ax=ax)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/links_8_0.png)

</div>
</div>
</div>



Note: colormap does not have enough colours, that is why everything on the top-left looks the same. It is not.



### Nodes

The situation with nodes is slightly more complicated as you usually don't have or even need nodes. However, `momepy` includes some functions which are calculated on nodes (mostly in `graph` module). For that reason, we will pretend that we follow the usual workflow:
1. Street network `GeoDataFrame` (edges only)
2. networkx `Graph`
3. Street network - edges and nodes as separate `GeoDataFrames`.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
graph = momepy.gdf_to_nx(streets)

```
</div>

</div>



Some [graph-based analysis](graph/graph) happens here.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
nodes, edges = momepy.nx_to_gdf(graph)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
edges.plot(ax=ax, column='nID', categorical=True, cmap='tab20b')
nodes.plot(ax=ax)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/links_14_0.png)

</div>
</div>
</div>



For attaching node ID to buildings, we will need both, nodes and edges. We have already determined which edge building belongs to, so now we only have to find out which end of the edge is the closer one. Nodes come from `momepy.nx_to_gdf` automatically with node ID:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
nodes.head()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">



<div markdown="0" class="output output_html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nodeID</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>POINT (1603585.640 6464428.774)</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>POINT (1603413.206 6464228.730)</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>POINT (1603268.502 6464060.781)</td>
    </tr>
    <tr>
      <td>3</td>
      <td>4</td>
      <td>POINT (1603363.558 6464031.885)</td>
    </tr>
    <tr>
      <td>4</td>
      <td>5</td>
      <td>POINT (1603607.303 6464181.853)</td>
    </tr>
  </tbody>
</table>
</div>
</div>


</div>
</div>
</div>



The same ID is now inlcuded in edges as well, denoting each end of edge. (Length of the edge is also present as it was necessary to keep as an attribute for the graph.)



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
edges.head()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">



<div markdown="0" class="output output_html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry</th>
      <th>nID</th>
      <th>mm_len</th>
      <th>node_start</th>
      <th>node_end</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>LINESTRING (1603585.640 6464428.774, 1603413.2...</td>
      <td>0</td>
      <td>264.103950</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <td>1</td>
      <td>LINESTRING (1603561.740 6464494.467, 1603564.6...</td>
      <td>14</td>
      <td>70.020202</td>
      <td>1</td>
      <td>9</td>
    </tr>
    <tr>
      <td>2</td>
      <td>LINESTRING (1603585.640 6464428.774, 1603603.0...</td>
      <td>15</td>
      <td>88.924305</td>
      <td>1</td>
      <td>7</td>
    </tr>
    <tr>
      <td>3</td>
      <td>LINESTRING (1603607.303 6464181.853, 1603592.8...</td>
      <td>2</td>
      <td>199.746503</td>
      <td>2</td>
      <td>5</td>
    </tr>
    <tr>
      <td>4</td>
      <td>LINESTRING (1603363.558 6464031.885, 1603376.5...</td>
      <td>5</td>
      <td>203.014090</td>
      <td>2</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>
</div>


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
buildings['nodeID'] = momepy.get_node_id(buildings, nodes, edges,
                                         'nodeID', 'nID')

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
100%|██████████| 144/144 [00:00<00:00, 254.66it/s]
```
</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
buildings.plot(ax=ax, column='nodeID', categorical=True, cmap='tab20b')
nodes.plot(ax=ax)
edges.plot(ax=ax)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/links_20_0.png)

</div>
</div>
</div>



### Transfer IDs to tessellation
All IDs are now stored in buildings gdf. We can copy them to tessellation using `merge`. First, we select columns we are interested in, then we merge them with tessellation based on the shared unique ID. Usually, we will have more columns than we have now.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
buildings.columns

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
Index(['uID', 'geometry', 'nID', 'nodeID'], dtype='object')
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
columns = ['uID', 'nID', 'nodeID']
tessellation = tessellation.merge(buildings[columns], on='uID')
tessellation.head()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">



<div markdown="0" class="output output_html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>uID</th>
      <th>geometry</th>
      <th>nID</th>
      <th>nodeID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1.0</td>
      <td>POLYGON ((1603578.489 6464344.527, 1603577.040...</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2.0</td>
      <td>POLYGON ((1603067.112 6464177.926, 1603054.848...</td>
      <td>33</td>
      <td>10</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3.0</td>
      <td>POLYGON ((1602978.618 6464156.859, 1603006.384...</td>
      <td>10</td>
      <td>12</td>
    </tr>
    <tr>
      <td>3</td>
      <td>4.0</td>
      <td>POLYGON ((1603056.595 6464093.903, 1603011.539...</td>
      <td>10</td>
      <td>12</td>
    </tr>
    <tr>
      <td>4</td>
      <td>5.0</td>
      <td>POLYGON ((1603110.459 6464114.367, 1603109.099...</td>
      <td>8</td>
      <td>12</td>
    </tr>
  </tbody>
</table>
</div>
</div>


</div>
</div>
</div>



Now we should be able to link all elements together as needed for all types of morphometric analysis in `momepy`.

