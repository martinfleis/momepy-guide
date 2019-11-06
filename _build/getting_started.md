---
redirect_from:
  - "/getting-started"
interact_link: content/getting_started.ipynb
kernel_name: geo_dev
has_widgets: false
title: 'Getting started'
prev_page:
  url: /intro.html
  title: 'Home'
next_page:
  url: /data_structure.html
  title: 'Data structure'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Getting started

## An introduction to momepy

Momepy is a library for quantitative analysis of urban form - urban morphometrics. It is built on top of [GeoPandas](http://geopandas.org), [PySAL](http://pysal.org) and [networkX](http://networkx.github.io).

Some of the functionality that momepy offers:

- Measuring [dimensions](https://docs.momepy.org/en/latest/api.html#dimension) of morphological elements, their parts, and aggregated structures.
- Quantifying [shapes](https://docs.momepy.org/en/latest/api.html#shape) of geometries representing a wide range of morphological features.
- Capturing [spatial distribution](https://docs.momepy.org/en/latest/api.html#spatial-distribution) of elements of one kind as well as relationships between different kinds.
- Computing density and other types of [intensity](https://docs.momepy.org/en/latest/api.html#intensity) characters.
- Calculating [diversity](https://docs.momepy.org/en/latest/api.html#diversity) of various aspects of urban form.
- Capturing [connectivity](https://docs.momepy.org/en/latest/api.html#graph) of urban street networks
- Generating relational [elements](https://docs.momepy.org/en/latest/api.html#elements) of urban form (e.g. morphological tessellation)

Momepy aims to provide a wide range of tools for a systematic and exhaustive analysis of urban form. It can work with a wide range of elements, while focused on building footprints and street networks.

## Installation

Momepy can be easily installed using conda from `conda-forge`. For detailed installation instructions, please refer to the [documentation](https://docs.momepy.org/en/latest/install.html).



```
conda install momepy -c conda-forge
```



### Dependencies

To run all examples in this notebook, you will need some optional dependencies. `matplotlib`, `descartes` and `mapclassify` (or `pysal`) for plotting (these are GeoPandas dependencies) and `osmnx` to download data from OpenStreetMap. You can install all using the following commands: 



```
conda install matplotlib descartes osmnx -c conda-forge
pip install mapclassify
```



## Simple examples

Here are simple examples using embedded `bubenec` dataset (part of the Bubeneč neighborhood in Prague).



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import momepy
import geopandas as gpd
import matplotlib.pyplot as plt

```
</div>

</div>



Morphometric analysis using momepy usually starts with GeoPandas GeoDataFrame containing morphological elements. In this case, we will begin with buildings represented by their footprints. 

We have imported `momepy`, `geopandas` to handle the spatial data, and `matplotlib` to get a bit more control over plotting. To load `bubenec` data, we need to get retrieve correct path using `momepy.datasets.get_path("bubenec")`. The dataset itself is a GeoPackage with more layers, and now we want `buildings`.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
buildings = gpd.read_file(momepy.datasets.get_path('bubenec'),
                          layer='buildings')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
buildings.plot(ax=ax)
ax.set_axis_off()
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_8_0.png)

</div>
</div>
</div>



*Note: To see the code used to plot the image above, click on the plus button in the top right corner.*

Momepy uses classes for each measurable character, which stores the resulting values but also original input and all parameters used for the computation. The only exception is the [`graph` module](graph/graph). To illustrate how momepy classes work, we can try to measure a few simple characters.

### Area

`momepy.Area` measures the area of polygon geometry. It is a simple wrapper around `gdf.geoemtry.area`, included in momepy for consistency. `momepy.Area` does not need any attributes apart from source GeoDataFrame.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blg_area = momepy.Area(buildings)
buildings['area'] = blg_area.area

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
buildings.plot('area', ax=ax, legend=True, scheme='quantiles', cmap='Blues', legend_kwds={'loc': 'lower left'})
ax.set_axis_off()
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_11_0.png)

</div>
</div>
</div>



Above, we have calculated the area of each building and then extracted resulting values (pandas.Series) to save them as a new column of `buildings`. You can also get original GeoDataFrame as documented in the [API](https://docs.momepy.org/en/latest/generated/momepy.Area.html#momepy.Area):



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blg_area.gdf.head()

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
      <th>area</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>POLYGON ((1603599.221 6464369.816, 1603602.984...</td>
      <td>728.557495</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>POLYGON ((1603042.880 6464261.498, 1603038.961...</td>
      <td>11216.093578</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>POLYGON ((1603044.650 6464178.035, 1603049.192...</td>
      <td>641.059515</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>POLYGON ((1603036.557 6464141.467, 1603036.969...</td>
      <td>903.746689</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>POLYGON ((1603082.387 6464142.022, 1603081.574...</td>
      <td>641.629131</td>
    </tr>
  </tbody>
</table>
</div>
</div>


</div>
</div>
</div>



### Equivalent rectangular index

`momepy.EquivalentRectangularIndex` is an example of a morphometric character capturing the shape of each object. It can be calculated in an analogical way as the area above:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blg_ERI = momepy.EquivalentRectangularIndex(buildings)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
100%|██████████| 144/144 [00:00<00:00, 1343.34it/s]
```
</div>
</div>
</div>



To calculate the equivalent rectangular index, we need to know the area and perimeter of each polygon. While momepy can compute both automatically, we might want to save time passing already computed areas directly:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blg_ERI = momepy.EquivalentRectangularIndex(buildings, areas='area')
buildings['eri'] = blg_ERI.eri

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
100%|██████████| 144/144 [00:00<00:00, 1175.62it/s]
```
</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
buildings.plot('eri', ax=ax, legend=True)
ax.set_axis_off()
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_18_0.png)

</div>
</div>
</div>



Apart from resulting values stored in `blg_ERI.eri` we can also see all values used for computation, including perimeters we have not passed above.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blg_ERI.areas.head()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
0      728.557495
1    11216.093578
2      641.059515
3      903.746689
4      641.629131
Name: area, dtype: float64
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
blg_ERI.perimeters.head()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
0    137.186310
1    991.345770
2    107.488923
3    141.740042
4    107.158092
Name: mm_p, dtype: float64
```


</div>
</div>
</div>



## Capturing relation between different elements

Urban form is a complex entity that needs to be represented by multiple morphological elements, and we need to be able to describe the relationship between them. With the absence of plots for our `bubenec` case, we can use [morphological tessellation](elements/tessellation) as the smallest spatial division.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
tessellation = gpd.read_file(momepy.datasets.get_path('bubenec'),
                             layer='tessellation')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot(ax=ax, edgecolor='white')
buildings.plot(ax=ax, color='white', alpha=.5)
ax.set_axis_off()
plt.show()


```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_24_0.png)

</div>
</div>
</div>



### Coverage area ratio

Now we can calculate how big part of each tessellation cell is covered by a related building, using `momepy.AreaRatio`. Momepy classes usually accept any list-like object to be passed as values on top of a column name. Below we are passing Series to `left_areas` and column name to `right_areas`. Buildings have the same `uID` as related tessellation cells, which is used to link both together (see [Data Structure](data_structure)).



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
coverage = momepy.AreaRatio(tessellation, buildings, left_areas=tessellation.area,
                            right_areas='area', unique_id='uID')
tessellation['CAR'] = coverage.ar

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
tessellation.plot('CAR', ax=ax, edgecolor='white', legend=True, scheme='NaturalBreaks', k=10, legend_kwds={'loc': 'lower left'})
buildings.plot(ax=ax, color='white', alpha=.5)
ax.set_axis_off()
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_27_0.png)

</div>
</div>
</div>



Finally, to cover the last of the essential elements, we import the street network. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
streets = gpd.read_file(momepy.datasets.get_path('bubenec'),
                        layer='streets')

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
plt.show()


```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_30_0.png)

</div>
</div>
</div>



### Street Profile

`momepy.StreetProfile` captures the relations between the segments of the street network and buildings. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
profile = momepy.StreetProfile(streets, buildings)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
100%|██████████| 35/35 [00:04<00:00,  7.56it/s]
```
</div>
</div>
</div>



We have now captured multiple characters of the street profile. As we did not specify building height (we do not know it for our data), we have widths (mean) - `profile.w`, standard deviation of widths (along the segment) - `profile.wd`,  and the degree of openness - `profile.o`.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
streets['width'] = profile.w
streets['openness'] = profile.o

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
buildings.plot(ax=ax)
streets.plot('width', ax=ax, legend=True, cmap='viridis_r')
ax.set_axis_off()
ax.set_title('width')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_35_0.png)

</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
buildings.plot(ax=ax)
streets.plot('openness', ax=ax, legend=True)
ax.set_axis_off()
ax.set_title('openness')
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_36_0.png)

</div>
</div>
</div>



## Using OpenStreetMap data

In some cases (based on the completeness of OSM), we can use OpenStreetMap data without the need to save them to file and read them via GeoPandas. We can use OSMnx to retrieve them directly. In this example, we will download the building footprint of Kahla in Germany and project it to projected CRS. Momepy expects that all GeoDataFrames have the same (projected) CRS.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import osmnx as ox

gdf = ox.footprints.footprints_from_place(place='Kahla, Germany')
gdf_projected = ox.project_gdf(gdf)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
gdf_projected.plot(ax=ax)
ax.set_axis_off()
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_39_0.png)

</div>
</div>
</div>



Now we are in the same situation as we were above, and we can start our morphometric analysis as illustrated on the equivalent rectangular index below.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
gdf_ERI = momepy.EquivalentRectangularIndex(gdf_projected)
gdf_projected['eri'] = gdf_ERI.eri

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">
{:.output_stream}
```
100%|██████████| 2932/2932 [00:01<00:00, 1478.48it/s]
```
</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area hidecode" markdown="1">
```python
f, ax = plt.subplots(figsize=(10, 10))
gdf_projected.plot('eri', ax=ax, legend=True, scheme='NaturalBreaks', k=10)
ax.set_axis_off()
plt.show()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](images/getting_started_42_0.png)

</div>
</div>
</div>



## Next steps

Now we have basic knowledge of what momepy is and how it works. It is time to [install](https://docs.momepy.org/en/latest/install.html) momepy (if you haven't done so yet) and browse the rest of this user guide to see more examples. Once done, head to the [API reference](https://docs.momepy.org/en/latest/api.html) to see the full extent of characters momepy can capture and find those you need for your research.

