---
interact_link: content/elements/blocks.ipynb
kernel_name: mmp_guide
has_widgets: false
title: 'Tessellation-based blocks'
prev_page:
  url: /elements/tessellation
  title: 'Morphological tessellation'
next_page:
  url: /elements/links
  title: 'Linking elements together'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---
# Generating tessellation-based blocks

## Ideal case with ideal data

Assume following situation: We have a layer of buildings for selected city, a street network represented by centrelines, we have generated morphological tessellation, but now we would like to do some work on block scale. We have two options - generate blocks based on street network (each closed loop is a block) or use morphological tessellation and join those cells, which are expected to be part of one block. In that sense, you will get tessellation-based blocks which are following the same logic as the rest of your data and are hence fully compatible. With `momepy` you can do that using `momepy.Blocks`. 



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



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, edgecolor='white', linewidth=0.2)
buildings.plot(ax=ax, color='white', alpha=.5)
streets.plot(ax=ax, color='black')
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/blocks_4_0.png)

</div>
</div>
</div>



This example is useful for illustration why pure network-based blocks are not ideal.
1. In the centre of the area would be the block, while it is clear that there is only open space.
2. Blocks on the edges of the area would be complicated, as streets do not fully enclose them.

None of it is an issue for tessellation-based blocks. `momepy.Blocks` requires buildings, tessellation, streets and IDs. We have it all, so we can give it a try.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blocks = momepy.Blocks(
    tessellation, streets, buildings, id_name='bID', unique_id='uID')

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
Buffering streets...
Generating spatial index...
Difference...
Defining adjacency...
Defining street-based blocks...
Defining block ID...
Generating centroids...
Spatial join...
Attribute join (tesselation)...
Generating blocks...
Multipart to singlepart...
Attribute join (buildings)...
Attribute join (tesselation)...
```
</div>
</div>
</div>



GeoDataFrame containing blocks can be accessed using `blocks`. Moreover, block ID for buildings and tessellation can be accessed using `buildings_id` and `tessellation_id`.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blocks_gdf = blocks.blocks
buildings['bID'] = blocks.buildings_id
tessellation['bID'] = blocks.tessellation_id

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
blocks_gdf.plot(ax=ax, edgecolor='white', linewidth=0.5)
buildings.plot(ax=ax, color='white', alpha=.5)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/blocks_9_0.png)

</div>
</div>
</div>



## Fixing the street network

The example above shows how it works in the ideal case - streets are attached, there are no gaps. However, that is often not true. For that reason, `momepy` includes `momepy.snap_street_network_edge` utility which is supposed to fix the issue. It can do two types of network adaptation:
1. **Snap network onto the network where false dead-end is present.** It will extend the last segment by set distance and snap it onto the first reached segment.
2. **Snap network onto the edge of tessellation.** As we need to be able to define blocks until the very edge of tessellation, in some cases, it is necessary to extend the segment and snap it to the edge of the tessellated area.

Let's adapt our ideal street network and break it to illustrate the issue.



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
import shapely
geom = streets.iloc[33].geometry
splitted = shapely.ops.split(geom, shapely.geometry.Point(geom.coords[5]))
streets.loc[33, 'geometry'] = splitted[1]
geom = streets.iloc[19].geometry
splitted = shapely.ops.split(geom, geom.representative_point())
streets.loc[19, 'geometry'] = splitted[0]

f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, edgecolor='white', linewidth=0.2)
buildings.plot(ax=ax, color='white', alpha=.5)
streets.plot(ax=ax, color='black')
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/blocks_11_0.png)

</div>
</div>
</div>



In this example, one of the lines on the right side is not snapped onto the network and other on the top-left leaves the gap between the edge of the tessellation. Let's see what would be the result of blocks without any adaptation of this network.





<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blocks = momepy.Blocks(tessellation, streets, buildings,
                       id_name='bID', unique_id='uID').blocks

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
Buffering streets...
Generating spatial index...
Difference...
Defining adjacency...
Defining street-based blocks...
Defining block ID...
Generating centroids...
Spatial join...
Attribute join (tesselation)...
Generating blocks...
Multipart to singlepart...
Attribute join (buildings)...
Attribute join (tesselation)...
```
</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
blocks.plot(ax=ax, edgecolor='white', linewidth=0.5)
buildings.plot(ax=ax, color='white', alpha=.5)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/blocks_15_0.png)

</div>
</div>
</div>



We can see that blocks are merged. To avoid this effect, we will first use `momepy.snap_street_network_edge` and then generate blocks.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
snapped = momepy.snap_street_network_edge(streets, buildings, tolerance_street=15,
                                          tessellation=tessellation, 
                                          tolerance_edge=40)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
Building R-tree for network...
Building R-tree for buildings...
Dissolving tesselation...
Snapping...
```
</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, edgecolor='white', linewidth=0.2)
buildings.plot(ax=ax, color='white', alpha=.5)
snapped.plot(ax=ax, color='black')
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/blocks_18_0.png)

</div>
</div>
</div>



What should be snapped is now snapped. We might have noticed that we had to give buildings to the `momepy.snap_street_network_edge`. That is to avoid extending segments through buildings.

With a fixed network, we can then generate correct blocks.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blocks = momepy.Blocks(
    tessellation, snapped, buildings, id_name='bID', unique_id='uID').blocks

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
Buffering streets...
Generating spatial index...
Difference...
Defining adjacency...
Defining street-based blocks...
Defining block ID...
Generating centroids...
Spatial join...
Attribute join (tesselation)...
Generating blocks...
Multipart to singlepart...
Attribute join (buildings)...
Attribute join (tesselation)...
```
</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
blocks.plot(ax=ax, edgecolor='white', linewidth=0.5)
buildings.plot(ax=ax, color='white', alpha=.5)
ax.set_axis_off()
plt.axis('equal')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/elements/blocks_21_0.png)

</div>
</div>
</div>

