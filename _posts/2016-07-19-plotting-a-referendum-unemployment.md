---
layout: post
title: Plotting a referendum - Unemployment
category: stats
sub: true
---
{% include base.html %}
Following the shock decision by the UK to leave the EU, many ideas were put 
forward as to the reasons behind why so many people voted the ways that they 
did. 

In an effort to learn more about data handling in python, using pandas, 
matplotlib and other fun stuff I scoured the internet for data and set about 
plotting graphs. 

1. [Overview] 
1. [Sourcing data]
    1. [Handling results]
    1. [Coding postcodes]
    1. [Capping CAPs]
    1. [Counting immigrants]
    1. [Income]
    1. [Unemployment]
1. [Mapping data]
1. [Pretending I know stats]

---

The Office of National Statistics releases unemployment data every month. This
is provided in an excel spreadsheet which pandas struggles to decode properly.
As such a lot of the code below is trying to fix the inconsistencies introduced
by reading the file.

The overall percentage of unemployed people for each reason is not calculated in
this script, but left till we have the total population of each region (provided
by the immigration data).

```
"""
Pickles the regional unemployment data available from the ONS.

Data available from:
		https://www.ons.gov.uk/employmentandlabourmarket/peoplenotinwork/
		unemployment/datasets/claimantcountbyunitaryandlocalauthorityexperimental

Assuming filename is: './data/lmregtabcc01may2016.xls'

Data is released under the Open Government Licence:
	http://www.nationalarchives.gov.uk/doc/open-government-licence/
"""

import pandas as pd
```

Start off by reading the excel file into a pandas DataFrame, then trim off any
rows which have a null index. Also drop the Scilly Isles (E06000053) as no
unemployment data is held about that region.

```
filename = './data/lmregtabcc01may2016.xls'
data = pd.read_excel(filename, header=[2, 3, 4], na_values=['..'])
data = data[map(pd.notnull, data.index)]
data.drop('E06000053', inplace=True)
```

To make it easier to access data and to help readability, we change the column
names. After doing so we can drop any columns under 'All', which are read
incorrectly from the spreadsheet.

```
data.columns = (
    data.columns.set_levels(['Unemployment_Count', 'Unemployment_Change_On_Year',
		    'Area_Name'], level=0)
    .set_levels(['Number', 'Percentage', 'Percentage', ''], level=1)
    .set_levels(['Men', 'All', '', 'Women'], level=2)
)
data.sortlevel(axis=1, inplace=True)
data.drop('All', axis=1, level=2, inplace=True)
```

The values of 'Count'->'Number'->'All' is given by summing up the number of men
and the number of women, so these columns can be recalculated easily.

For the percentages, we need to know the total population, before the percentage
of unemployed people can be computed, so we leave that until we combine all the
data together.

```
for l0 in ['Unemployment_Count', 'Unemployment_Change_On_Year']:
    data[(l0, 'Number', 'All')] = data[(l0, 'Number', 'Men')] + data[(l0,
        'Number', 'Women')]
data.sortlevel(axis=1, inplace=True)
```

Finally save the DataFrame to a pickle.

```
data.to_pickle('./data/unemployment.pkl')
```


[Overview]: {% post_url 2016-07-19-plotting-a-referendum %}
[Sourcing data]: {% post_url 2016-07-19-plotting-a-referendum-data %}
[Handling results]: {% post_url 2016-07-19-plotting-a-referendum-counting-votes %}
[Coding postcodes]: {% post_url 2016-07-19-plotting-a-referendum-postcodes %}
[Capping CAPs]: {% post_url 2016-07-19-plotting-a-referendum-cap %}
[Counting immigrants]: {% post_url 2016-07-19-plotting-a-referendum-chasing-immigrants %}
[Income]: {% post_url 2016-07-19-plotting-a-referendum-income %}
[Unemployment]: {% post_url 2016-07-19-plotting-a-referendum-unemployment %}
[Mapping data]: {% post_url 2016-07-19-plotting-a-referendum-map %}
[Pretending I know stats]:  {% post_url 2016-07-19-plotting-a-referendum-stats %}