---
interact_link: content/weights/diversity.ipynb
kernel_name: mmp_guide
has_widgets: false
title: 'Measuring diversity'
prev_page:
  url: /weights/examples.html
  title: 'Examples of usage'
next_page:
  url: /weights/two.html
  title: 'Using two spatial weights matrices'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Measuring diversity

As important diversity is for cities, as complicated is to capture it. `momepy` offers several options on how to do that using urban morphometrics. Generally, we can distinguish three types of diversity characters, based on:

1. Absolute values
2. Relative values
3. Categorization (binning)

This notebook provides examples from each of them.



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



### Queen contiguity

Morphological tessellation allows using contiguity-based weights matrix. While `libpysal.weights.contiguity.Queen` will do a classic Queen contiguity matrix; it might not be enough to capture proper context. For that reason, we can use `momepy.sw_high` to capture all neighbours within set topological distance `k`. More in [Generating spatial weights](weights_nb).



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
sw3 = momepy.sw_high(k=3, gdf=tessellation, ids='uID')

```
</div>

</div>



To have a character whose diversity can be measured, we can use the area of tessellation.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
tessellation['area'] = momepy.Area(tessellation).area

```
</div>

</div>



### Range

The range is as simple as it sounds; it measures the range of the values withing all neighbours as captured by `spatial_weights`.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
area_rng = momepy.Range(tessellation, values='area',
                                        spatial_weights=sw3, unique_id='uID')
tessellation['area_rng'] = area_rng.r

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='area_rng', legend=True, cmap='Spectral_r')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/diversity_10_0.png)

</div>
</div>
</div>



However, as we can see from the plot above, there is a massive effect of large-scale buildings, which can be seen as outliers. For that reason, we can define `rng` keyword argument to limit the range taken into account. To get the interquartile range:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
area_iqr = momepy.Range(tessellation, values='area',
                        spatial_weights=sw3, unique_id='uID',
                        rng=(25, 75))
tessellation['area_IQR'] = area_iqr.r

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='area_IQR', legend=True, cmap='Spectral_r')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/diversity_13_0.png)

</div>
</div>
</div>



The effect of outliers has been successfully eliminated.

### Theil index

Theil index is a measure of inequality (as Gini index is). `momepy` is using `pysal`'s implementation of the Theil index to do the calculation.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
area_theil = momepy.Theil(tessellation, values='area',
                          spatial_weights=sw3,
                          unique_id='uID')
tessellation['area_Theil'] = area_theil.t

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='area_Theil', scheme='fisherjenkssampled', k=10, legend=True, cmap='plasma')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/diversity_16_0.png)

</div>
</div>
</div>



Again, the outlier effect is present. We can use the same keyword as above to limit it and measure the Theil index on the inter-decile range.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
area_id_theil = momepy.Theil(tessellation, values='area',
                             spatial_weights=sw3,
                             unique_id='uID',
                             rng=(10, 90))
tessellation['area_Theil_ID'] = area_id_theil.t

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='area_Theil_ID', scheme='fisherjenkssampled', k=10, legend=True, cmap='plasma')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/diversity_19_0.png)

</div>
</div>
</div>



### Simpson's diversity index

Simpson's diversity index is one of the most used indices capturing diversity. However, we need to be careful using it for continuous values, as it depends on the binning of these values to categories. The effect of different binning could be significant. `momepy` uses Head/tail Breaks as a large number of morphometric characters follows power-law distribution (for which Head/tail Breaks are designed). However, you can use any binning provided by `mapclassify` (including user-defined). The default Head/tail Breaks:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
area_simpson = momepy.Simpson(tessellation, values='area',
                              spatial_weights=sw3,
                              unique_id='uID')
tessellation['area_simpson'] = area_simpson.s

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='area_simpson', legend=True, cmap='viridis')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/diversity_22_0.png)

</div>
</div>
</div>



And binning based on quantiles (into 7 bins of equal size):



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
tessellation['area_simpson_q7'] = momepy.Simpson(tessellation, values='area',
                                                 spatial_weights=sw3,
                                                 unique_id='uID',
                                                 binning='quantiles', k=7).s

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, column='area_simpson_q7', legend=True, cmap='viridis')
buildings.plot(ax=ax, color="white", alpha=0.4)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/weights/diversity_25_0.png)

</div>
</div>
</div>



Always consider whether your binning is the optimal one.

