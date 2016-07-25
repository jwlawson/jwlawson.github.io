---
layout: post
title: Plotting a referendum - Income
category: interest
sub: true
nex: Unemployment
nextfile: 2016-07-19-plotting-a-referendum-unemployment
prev: Immigration
prevfile: 2016-07-19-plotting-a-referendum-immigrants
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

The income data downloaded from Nomis is already very structured and succinct,
as we only chose the data we wanted to use. As such we don't need to do much to
it before pickling it.

We rename the columns, set the electoral wards as index and throw away a few
rows which were constructed from the comments in the initial `csv` file.

```python
"""
Data available from:
    NOMIS website.
    Dataset: Annual Survey of Hours and Earnings
        -> annual survey of hours and earnings - resident analysis
    Geography: local authorities: district / unitary (as of April 2015)
    Date: 2015
    Pay and hours: Annual pay - gross
    Sex & full/part-time: full-time workers
    Variable: median
    Include Area codes
Assumes resulting filename is: 'nomis-media-income.csv'
Data is released under the Open Government Licence:
        http://www.nationalarchives.gov.uk/doc/open-government-licence/
"""

import pandas as pd

inc = pd.read_csv('./raw/nomis-median-income.csv',header=7, na_values=['#','-'])
inc.columns = ['Area','Code','Income','Income_Conf']
inc.set_index('Code',inplace=True)
inc.drop('Column Total', inplace=True)
inc.sort_index(inplace=True)
inc = inc.dropna(subset=['Area'])

inc.to_pickle('./data/income.pkl')
```

File also available [here][gist].

[gist]: https://gist.github.com/jwlawson/41302a734c6d9b0392cbd60571d755bf#file-pickle_income-py
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
