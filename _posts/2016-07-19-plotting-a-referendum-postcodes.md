---
layout: post
title: Plotting a referendum - Postcodes
category: interest
sub: true
prev: Results
prevfile: 2016-07-19-plotting-a-referendum-votes
nex: The CAP
nextfile: 2016-07-19-plotting-a-referendum-cap
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

We need to collect information on the relation between postcdoes and voting
districts as the CAP data is indexed by postcode prefix and is not available
with the ward data. In this file we construct a 'map' which takes postcode
prefix to a corresponding (approximate) district ward.

The OS Codepoint data is provided as a huge number of different `csv` files,
each containing information about a number of postcodes.

```python
"""
Loads OS Open Code Point data and combines all postcodes into one DataFrame.
This maps each postcode to its ward code, and then also creates another which
maps the postcode prefix to its ward code.
"""

import glob
import os
import pandas as pd

codepoint_dir = './raw/codepoint-open_1418967'
header_file = codepoint_dir + '/docs/Code-Point_Open_Column_Headers.csv'
one_let_dir = codepoint_dir + '/one_letter_pc_code'
two_let_dir = codepoint_dir + '/two_letter_pc_code'
```

The column headings are stored separately to the data, so load into a DataFrame.

Each postcode directory contains various `csv` files containing the data, so
using a glob to list each of these files we can concatenate them together into a
single DataFrame using the headers extracted earlier.

```python
def get_header():
    """ Get the column headers for all the postcode data """
    return pd.read_csv(header_file, header=1)

def get_all_csv(d, headers):
    """ Get data from all csv files in the directory d """
    files = glob.glob(os.path.join(d, "*.csv"))
    return pd.concat( (pd.read_csv(f, names=headers) for f in files),
        ignore_index=True)
```

Concatenate the one-letter postcode data with the two-letter data to give the
full dataset.

```python
def combine_all():
    """ Combine the headers, one letter and two letter postcodes """
    h = get_header()
    all_one = get_all_csv(one_let_dir, h)
    all_two = get_all_csv(two_let_dir, h)
    return pd.concat([all_one, all_two], ignore_index=True)
```

The only columns we really care about in the dataset is the district codes and
the postcodes, so throw away everything else and give the columns nice names.

```python
a = combine_all()
a = a[['Postcode', 'Admin_district_code']]
a.columns = ['Postcode', 'Ward']
```
So far we have a list of all postcodes from the UK, each of which has a disrtict
ward code. The CAP data is provided with only the prefix of the postcode, so we
extract the first 4 chars from each and set that as a column `PC Prefix`.

```python
prefix = a['Postcode'].str[0:4].str.strip()
prefix.name = 'PC Prefix'
a = pd.concat([prefix, a], axis=1)
```

Each postcode belongs to a district ward, however the postcode boundaries do not
match the ward boundaries. Ideally we would `groupby` postcode prefix and each
prefix would have a single corresponding ward code, but this doesn't happen.

I chose to select only the most prevalent ward to represent each postcode. This
does mean that any data 'translated' using this will be entirely accurate.

```python
b = a.dropna(how='any').groupby('PC Prefix')['Ward'].agg(
        lambda x: x.value_counts().index[0])
b.to_pickle('./data/pc_prefixes.pkl')
```

The whole file is available in this [gist].

[gist]: https://gist.github.com/jwlawson/41302a734c6d9b0392cbd60571d755bf#file-pickle_postcodes-py
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
