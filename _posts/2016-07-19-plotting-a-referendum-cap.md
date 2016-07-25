---
layout: post
title: Plotting a referendum - CAP Spending
category: interest
sub: true
prev: Postcodes
prevfile: 2016-07-19-plotting-a-referendum-postcodes
nex: Immigration
nextfile: 2016-07-19-plotting-a-referendum-immigrants
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

## Sanitizing the CAP data

We start off by deciding which of the columns of data we really want to keep. We
need postcodes to place the data geographically, while the various totals of
spending give the data we care about.

The provided column names are not very clear, so we provide 'nice' versions as
well.

```python
"""
Pickles the CAP data, extracting only none-identifiable data
Data available from:
http://cap-payments.defra.gov.uk/Download.aspx
Available under the `Open Government Licence for public sector information`
http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/
"""

import pandas as pd

sheet_columns = ['PostcodePrefix_F202B', 'TownCity_F202C',
                'OtherEAGFTotal', 'DirectEAGFTotal',
                'RuralDevelopmentTotal', 'Total']
final_columns = ['Postcode', 'Town', 'Other_EAGF', 'Direct_EAGF',
                'Rural_Development', 'Total']
```

Now define a function to read in the excel file and combine into a single
DataFrame. As many different sheets are used to hold the data, we use a simple
list comprehension to read each in turn.

Pandas provide an ExcelFile object, which allows us to read each sheet
separately without having to open and close the file each time.

```python
def load_cap_data(filename):
    with pd.ExcelFile(filename) as xls:
        return pd.concat( (pd.read_excel(xls, sheet) for sheet in
            xls.sheet_names), ignore_index=True)
```

Now that we have a DataFrame, we use the column headings defined above to throw
away all others, as well as any null values, before renaming the columns.

The next goal is to combine all the rows with the same postcodes, so that we
only have the sums. There is a slight problem with the `Town` column, where each
entry is of type `str` and so cannot be summed. We deal with this separately,
passing a lambda function to use as the `groupby` aggregator which returns the
most common `Town` appearing for each postcode.

All the numeric columns behave as expected, so the simple
`data.groupby('Postcode').sum()` returns a DataFrame with the sums indexed by
postcodes that we wanted.

```python
def organise_data(df):
    data = df.dropna(how='any', subset=['PostcodePrefix_F202B'])[sheet_columns]
    data.columns = final_columns
    town_names = (
            data[['Postcode', 'Town']].groupby('Postcode')
            .agg(lambda x: x.value_counts().index[0])
    )
    sums = data.groupby('Postcode').sum()
    return pd.concat([town_names, sums], axis=1)
```

All that's left now is to use these functions to load the data from the file,
then pickle it into `'data/CAP2015/pkl'`.

```python
def get_cap(year):
    df = load_cap_data('./raw/' + year + '_All_CAP_Search_Results_Data_P14.xls')
    return organise_data(df)

d15 = get_cap('2015')
d15.to_pickle('./data/CAP2015.pkl')
```

The full code can be found in [this gist][pickle].

## Indexing by electoral wards

The CAP data is provided in terms of postcode prefix, so we constructed a map
from postcodes to electoral wards in the [previous page][Coding postcodes].

```python
"""
Convert the CAP data indexed by postcode to indexed by voting district code.
"""
import pandas as pd


cap = pd.read_pickle('./data/CAP2015.pkl')
pc = pd.read_pickle('./data/pc_prefixes.pkl')
```

Now `pc` is a Series of electoral wards indexed by postcodes, and `cap` is the
CAP data indexed by postcodes. By resetting the index of `cap`, these postcodes
are moved into a separate column, and each row is indexed from 1.

The string functions on `cap.Postcode` ensure that all postcodes in the CAP data
match those in the postcode data, so `cap.Postcode.str.strip().str.upper()` is a
Series of postcodes and the call to `map` replaces each postcode with the
corresponding electoral ward from `pc`.

```python
# Reindex and convert CAP spending in to area wards
cap = cap.reset_index()
cap_codes = cap.Postcode.str.strip().str.upper().map(pc)
cap_codes.name = 'Ward'
```
Now add these electoral wards to the `cap` DataFrame and use them to collect all
the data for each ward.

```python
cap = pd.concat([cap, cap_codes], axis=1).groupby('Ward').sum()
cap.to_pickle('./data/pc_cap2015.pkl')
```

The full code can be found in [this gist][postcode].


[postcode]: https://gist.github.com/jwlawson/41302a734c6d9b0392cbd60571d755bf#file-find_cap_pc-py
[pickle]: https://gist.github.com/jwlawson/41302a734c6d9b0392cbd60571d755bf#file-pickle_cap-py

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

