---
redirect_from:
  - "/weights/weights-nb"
interact_link: content/weights/weights_nb.ipynb
kernel_name: mmp_guide
has_widgets: false
title: 'Generating spatial weights'
prev_page:
  url: /weights/weights.html
  title: 'Using spatial weights matrix'
next_page:
  url: /weights/examples.html
  title: 'Examples of usage'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Generating spatial weights

`momepy` is using `libpysal` to handle spatial weights, but also builds on top of it. This notebook will show how to use different weights.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import momepy
import geopandas as gpd
import matplotlib.pyplot as plt

```
</div>

</div>



We will again use `osmnx` to get the data for our example and after preprocessing of building layer will generate tessellation. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import osmnx as ox

gdf = ox.footprints.footprints_from_place(place='Kahla, Germany')
gdf_projected = ox.project_gdf(gdf)

buildings = momepy.preprocess(gdf_projected, size=30,
                              compactness=True, islands=True)
buildings['uID'] = momepy.unique_id(buildings)
limit = momepy.buffered_limit(buildings)
tessellation = momepy.Tessellation(buildings, unique_id='uID', limit=limit).tessellation

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
Loop 1 out of 2.
Loop 2 out of 2.
Inward offset...
Discretization...
Generating input point array...
Generating Voronoi diagram...
Generating GeoDataFrame...
Dissolving Voronoi polygons...
Preparing limit for edge resolving...
Building R-tree...
Identifying edge cells...
Cutting...
```
</div>
</div>
</div>



## Queen contiguity

Morphological tessellation allows using contiguity-based weights matrix. While `libpysal.weights.contiguity.Queen` will do the standard Queen contiguity matrix of the first order; it might not be enough to capture proper context. For that reason, we can use `momepy.sw_high` to capture all neighbours within set topological distance `k`. It generates spatial weights of higher orders under the hood and joins them together.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sw3 = momepy.sw_high(k=3, gdf=tessellation, ids='uID')

```
</div>

</div>



Queen contiguity of morphological tessellation can capture the comparable level of information across the study area - the number of the neighbour is relatively similar and depends on the morphology of urban form. We can visualize it by counting the number of neighbours (as captured by `sw3`).



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
tessellation['neighbours'] = momepy.Neighbors(tessellation, sw3,'uID').n

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='neighbours', legend=True, cmap='Spectral_r')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/weights_nb_8_0.png)

</div>
</div>
</div>



## Distance

Often we want to define the neighbours based on metric distance. We will look at two options - distance band and k-nearest neighbour.

### Distance band

We can imagine distance band as a buffer of a set radius around each object, for example, 400 meters. For that, we can use `libpysal.weights.DistanceBand`:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import libpysal
dist400 = libpysal.weights.DistanceBand.from_dataframe(buildings, 400,
                                                       ids='uID')

```
</div>

</div>



Because we have defined spatial weights using uID, we can use `dist400` generated on buildings and use it on tessellation:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
tessellation['neighbours400'] = momepy.Neighbors(tessellation, dist400, 'uID').n

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='neighbours400', legend=True, cmap='Spectral_r')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/weights_nb_13_0.png)

</div>
</div>
</div>



### K nearest neighbor

If we want fixed number of neighbours, we can use `libpysal.weights.KNN`:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
knn = libpysal.weights.KNN.from_dataframe(buildings, k=200, ids='uID')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
tessellation['neighboursKNN'] = momepy.Neighbors(tessellation, knn,'uID').n

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='neighboursKNN', legend=True, cmap='Spectral_r')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/weights_nb_17_0.png)

</div>
</div>
</div>



All of them can be used within morphometric analysis. Theoretical and practical differences are discussed in Fleischmann, Romice and Porta (2019).

There are other options, see [lipysal API](https://libpysal.readthedocs.io/en/latest/api.html).



#### References
- Fleischmann M, Romice O and Porta S (2019) Applicability of morphological tessellation and its topological derivatives in the quantitative analysis of urban form. XXVI International Seminar on Urban Form: Cities as Assemblages. In: Nicosia.

