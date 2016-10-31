---
layout: post
title: Plotting a referendum - Mapping the data
category: interest
sub: true
nex: Scatter plots
nextfile: 2016-07-19-plotting-a-referendum-stats
prev: Unemployment
prevfile: 2016-07-19-plotting-a-referendum-unemployment
---
{% include base.html %}
Following the shock decision by the UK to leave the EU, many ideas were put 
forward as to the reasons behind why so many people voted the ways that they 
did. 

In an effort to learn more about data handling in python, using pandas, 
matplotlib and other fun stuff I scoured the internet for data and set about 
plotting graphs. 

* [Overview] 
* [Sourcing data]
    * [Results]
    * [Postcodes]
    * [The CAP]
    * [Immigration]
    * [Income]
    * [Unemployment]
* [Mapping data]
* [Scatter plots]

---

```python
"""
Uses Ordnance Survey BoundaryLine data to plot district/unitary wards.
"""
import cPickle as pickle
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon
from matplotlib.collections import PatchCollection
import matplotlib.cm as cm
import matplotlib.colors
from mpl_toolkits.basemap import Basemap
import numpy as np
import os.path

basemap_pkl = './data/ward_map.pkl'
text_col = '#555555'
```

First up we need to get a Basemap instance centered roughly on the UK. This
takes a while to create, especially if a detailed resolution is chosen, so we
dump a copy of the map into a pickle file for quicker loading.

This Basemap is chosen to use the trans-mercator map projection, which gives a
recognisable shape to the UK, and we provide the ellipsoid to match the data in
the shapefile. The corners of the map are specified as the bounds of the UK
(`llcrnr` is lower left corner, `ur` is upper right), with the map centered at
(54, -1.7), somewhere just north of Leeds and roughly in the middle of the UK.

The shapefile loaded later provides a coastline as well as the ward boundaries,
so we choose not to load anything into the map using a resolution of `None`.

```python
def get_basemap():
    if os.path.isfile(basemap_pkl):
        with open(basemap_pkl, 'r') as f:
            return pickle.load(f)
    else:
        pmap = Basemap(
                projection='tmerc',
                ellps='WGS84',
                llcrnrlat=49.16,
                llcrnrlon=-7.91,
                urcrnrlat=61.35,
                urcrnrlon=3.77,
                resolution=None,
                lat_0=54,
                lon_0=-1.7)
        with open(basemap_pkl, 'w') as f:
            pickle.dump(pmap, f, pickle.HIGHEST_PROTOCOL)
        return pmap
```

The `draw_data` function is the one which actually draws and colours the map.
Start off with creating a new `matplotlib` figure with axes and get the colormap
to use later, before loading the Basemap.

```python
def draw_data(data, f, colors, mx, mn, title, source='', show=False):
    fig = plt.figure(figsize=(7, 7), dpi=96)
    ax = fig.add_subplot(111, axisbg='w', frame_on=False)
    cmap = plt.get_cmap(colors)

    m = get_basemap()
```

The basemap can be used to read the ward shapefile. This adds two fields to the
basemap instance `wards` and `wardsinfo`: the first contains the shapes stored
in the file, while the second contains the additional information - ward code,
name and so on - which is stored in the shapefile's `dbf`.

As we read the shapes later we will append each shape to `patches`, along with
that shape's colour in `colors` and the ward's name in `names`. This dictionary
is used to provide labels for the maximum and minimum values of the data passed
in. Unfortunately the data doesn't always match exactly with the wards in the
shapefile, for example we have no boundary information on Northern Ireland or
Gibraltar but these do show up in the raw data. To avoid any problems when
looking up the names we manually add these edge cases to the dictionary at the
start.

```python
    m.readshapefile('./data/wards1', 'wards')
    patches = []
    colors = []
    names = {
        'GI': 'Gibraltar',
        'N92000002': 'Northern Ireland',
        'N09000005': 'Derry and Strabane'
    }
```

Now we loop through all the wards, find the correct colour to use and add that
to the `colors` list while also constructing a `Polygon` of the ward and adding
that to the `patches` list. If the ward does not appear in the provided data,
then a placeholder grey color is used instead.

```python
    for info, shape in zip(m.wards_info, m.wards):
        code = info['CODE']
        names[code] = info['NAME']
        if code in data.index.values and ~np.isnan(data[code]):
            colors.append(cmap(data[code]))
        else:
            colors.append((0.3, 0.3, 0.3, 1.0))
        patches.append(Polygon(np.array(shape), True))
```

With our list of patches and colours finished we use a `PatchCollection` to
collect them together and draw them to the axes. The first polygon will be
coloured with the first colour, second with the second, etc, until all wards are
drawn.

```python
    pc = PatchCollection(patches, edgecolor=text_col, linewidths=.1, zorder=2)
    pc.set_facecolor(colors)
    ax.add_collection(pc)
```

At this point, the map has been drawn and coloured. All that is left is to add
useful data such as a colorbar key, labels, information and smallprint.

Adding a colorbar requires a `ScalarMappable`, which is usually used to both
normalise and colour data. As we have been normalising the data separately we
need to construct a dummy instance which contains our colour map.

```python
    mappable = cm.ScalarMappable(cmap=cmap)
    mappable.set_array([])
    mappable.set_clim(0, 1)
    cbar = fig.colorbar(mappable, ticks=[0, 1], shrink=0.5)
    cbar_labels = cbar.ax.set_yticklabels([mn, mx], color=text_col)
```

Sort the data and extract the maximum and minimum values to show on the side. By
drawing on the colorbar axes, this text appears to the side of the map and out
of the way. The coordinates provided to the text instance are linked to the
colorbar, with y=1 being the top and y=0 the bottom of the bar.

Also add the smallprint to the bottom of the map. We need to use
`transform=ax.transAxes` to map the coordinates of the text instance to the
coordinates of the figure, with the corners of the axes being (0,0), (0,1) etc.

```python
    sort = data.dropna().sort_values(ascending=False)
    highest = '\n'.join([ names[i] for i in sort.index[:4]])
    highest = 'Maximum Wards:\n\n' + highest
    lowest = '\n'.join([ names[i] for i in sort.index[-4:][::-1]])
    lowest = 'Minimum Wards:\n\n' + lowest
    extremes = highest + '\n\n\n' + lowest
    details = cbar.ax.text(
        -1., 1 - 0.007,
        extremes,
        ha='right', va='top',
        size=6,
        color=text_col)

    copyright = 'Contains OS data'
    copyright += '$\copyright$ Crown copyright and database right 2016'
    if len(source) > 0:
        copyright += '\nData provided by ' + source
        copyright += '\nData available under the Open Government License'
    smallprint = ax.text(
        1., 0.05,
        copyright,
        ha='right', va='bottom',
        size=4,
        color=text_col,
        transform=ax.transAxes)
```

Now we save the map in a range of sizes. At each point we resize the text, so
that it appears natural and a consistent size.

```python
    title = ax.set_title(title, color=text_col)
    plt.tight_layout()
    fig.savefig('img/' + f + '_7.png', dpi=96, bbox_inches='tight')

    fig.set_size_inches(12, 12)
    details.set_size(details.get_size() * 12 / 7)
    smallprint.set_size(smallprint.get_size() * 12 / 7)
    title.set_size(title.get_size() * 12 / 7 )
    for l in cbar_labels:
        l.set_size(l.get_size() * 12 / 7 )
    fig.savefig('img/' + f + '_12.png', dpi=96, bbox_inches='tight')

    fig.set_size_inches(36, 36)
    details.set_size(details.get_size() * 36 / 12)
    smallprint.set_size(smallprint.get_size() * 36 / 12)
    title.set_size(title.get_size() * 36 / 12 )
    for l in cbar_labels:
        l.set_size(l.get_size() * 36 / 12 )
    fig.savefig('img/' + f + '_36.png', dpi=96, bbox_inches='tight')

    if(show):
        plt.show()
```

We also provide a helper function which automatically scales some data and uses
that to draw a map. This allows us to easily draw maps without having to copy
the normalising code each time.

```python
def scale_draw(data, f, colors, title, source='', show=False):
    """
    Draws the provided data onto a chloropleth map of the UK. The data will be
    rescaled from 0 to 1 in the process.
    The map is saved in 3 sizes to the provided filename as pngs.
    Arguments:
        data -- pandas Series indexed by district wards
        f -- filename to save plots to
        colors -- string identifying a matplotlib colormap
        title -- Title of the plot
        show -- show the plot as an interactive window (default False)
    """
    mx = data.max()
    mn = data.min()
    if(data._is_view):
        data = data.copy()
    norm = matplotlib.colors.Normalize(vmin=mn, vmax=mx)
    norm(data)
    mx = '{:.3g}'.format(mx)
    mn = '{:.3g}'.format(mn)
    draw_data(data, f, colors, mx, mn, title, source, show)
```

The whole file is available as a [gist].

[gist]: https://gist.github.com/jwlawson/41302a734c6d9b0392cbd60571d755bf#file-map_draw-py
[Overview]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum %}
[Sourcing data]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-data %}
[Results]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-votes %}
[Postcodes]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-postcodes %}
[The CAP]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-cap %}
[Immigration]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-immigrants %}
[Income]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-income %}
[Unemployment]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-unemployment %}
[Mapping data]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-map %}
[Scatter plots]:  {{ base }}{% post_url 2016-07-19-plotting-a-referendum-stats %}
