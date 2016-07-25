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

def get_header():
    return pd.read_csv(header_file, header=1)

def get_all_csv(d, headers):
    files = glob.glob(os.path.join(d, "*.csv"))
    return pd.concat( (pd.read_csv(f, names=headers) for f in files),
        ignore_index=True)

def combine_all():
    h = get_header()
    all_one = get_all_csv(one_let_dir, h)
    all_two = get_all_csv(two_let_dir, h)
    return pd.concat([all_one, all_two], ignore_index=True)

a = combine_all()
a = a[['Postcode', 'Admin_district_code']]
a.columns = ['Postcode', 'Ward']

prefix = a['Postcode'].str[0:4].str.strip()
prefix.name = 'PC Prefix'
a = pd.concat([prefix, a], axis=1)
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
