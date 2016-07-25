---
layout: post
title: Plotting a referendum - Unemployment
category: interest
sub: true
nex: Mapping the data
nextfile: 2016-07-19-plotting-a-referendum-map
prev: Income
prevfile: 2016-07-19-plotting-a-referendum-income
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

As with the income data, the unemployment data is downloaded from nomis, and so
is already close to how we want it. All that is required is removing some rows
which contain invalid data, as well as renaming the columns, before saving as a
pickle.

```python
"""
Data available from:
    NOMIS website.
    Dataset: Claimant count -> Claimant count by sex and age
    Geography: local authorities: district / unitary (as of April 2015)
    Date: May 2016
    Age: All (16+)
    Rates: Claimant count AND Claimants as a proportion of residents aged 16-64
    Sex: Total
    Include Area codes
Assumes resulting filename is: 'nomis-unemployment.csv'
Data is released under the Open Government Licence:
        http://www.nationalarchives.gov.uk/doc/open-government-licence/
"""

import pandas as pd

a = pd.read_csv('raw/nomis-unemployment.csv', header=6)
a.columns = ['Area','Code','Unemployed','Pct_Unemployed']
a.set_index('Code', inplace=True)
a = a.dropna()
a.drop('Column Total', inplace=True)
a.sort_index(inplace=True)

a.to_pickle('./data/unemployment.pkl')
```

The whole file is also available as a [gist].

[gist]: https://gist.github.com/jwlawson/41302a734c6d9b0392cbd60571d755bf#file-pickle_unemployment-py
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
