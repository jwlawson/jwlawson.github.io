---
layout: post
title: Plotting a Referendum - Chasing immigrants
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

* [Overview] 
* [Sourcing data]
    * [Handling results]
    * [Coding postcodes]
    * [Capping CAPs]
    * [Counting immigrants]
    * [Income]
    * [Unemployment]
* [Mapping data]
* [Pretending I know stats]

---

The Office of National Statistics releases local area migration data
[online][1], available under the [Open Government Licence][2].

This data is provided in a excel file with many sheets showing different metrics
and data. The complete file is included at the bottom of the page.

Of the whole spreadsheet, we are only interested in looking at the immigration
data in local areas. We focus on the number of non-British people in an area as
well as the number of people not born within the UK. The data is provided over
10 years from 2004 throught to 2016.

{% highlight python %}
import pandas as pd

sheets = ['Non-UK Born Population', 'Non-British Population']
years = ['2004', '2005', '2006', '2007', '2008', '2009', '2010', '2011', '2012',
        '2013', '2014']
year_cols = ['Percent', 'CI']
other_cols = ['10 Year Growth', 'Area Name']
{% endhighlight %}

Pandas provides an ExcelFile object, so that even though we create two
DataFrames we don't need to go through the process of opening the file twice.
Using it as a content provider also ensures that it is closed properly once we
are done.

{% highlight python %}
def load_data(filename):
	with pd.ExcelFile(filename) as xls:
		...
{% endhighlight %}

The result of `load_data` is a dictionary mapping each entry in `sheets` to a
DataFrame containing the data from that sheet.

{% highlight python %}
def load_data(filename):
	with pd.ExcelFile(filename) as xls:
		result = {}
		for s in sheets:
{% endhighlight %}

This DataFrame is easily read in using the pandas `read_excel` method, and
`dropna` removes any fully blank rows. The data also includes a number of area
codes starting `95` which are Irish area codes and so not included in the
referendum voting. The area codes are strings, so the string method
`startswith('95')` matches these areas, so the negated call
`df.loc[~df.index.str.startswith('95')]` gives a DataFrame without these rows.


{% highlight python %}
			df = pd.read_excel(xls, s, header=[0, 2]).dropna(how='all')
			df = df.loc[~df.index.str.startswith('95')]
{% endhighlight %}

The option `header=[0,2]` tells pandas which rows to use as the column indices
and gives a multi-indexed data frame with a structure like

```
|           | Jan 2004 to Dec 2004                                | Jan 2005 to Dec 2005  ... |
| Area code | Resident Population | Non-British Estimate | CI +/- | Resident Population | ... |
```
To simplify things a bit we change the top column headers to just the year.

{% highlight python %}
			df.columns.set_levels(['Area Name', '2004', '2005', '2006', '2007',
				'2008', '2009', '2010', '2011', '2012', '2013', '2014'], level=0,
				inplace=True)
{% endhighlight %}

The excel file contains colons for non-existant data, which confuses pandas and
forces each of the data columns to be `object` type rather than numeric which we
need to perform any computations. To force them to be numeric we apply
`pd.to_numeric` to each column and passing `errors='coerce'` tells pandas to set
any non-numeric data to `NaN`.

{% highlight python %}
			for year in years:
				df[year] = df[year].apply(lambda x: pd.to_numeric(x, errors='coerce'))
{% endhighlight %}

The multi-index needs to be sorted, as otherwise pandas cannot easily perform
slices on columns. The column axis is always `axis=1`, whereas the default in
pandas is to normally perform operations like this across the rows (`axis=0`).

{% highlight python %}
			df.sortlevel(axis=1, inplace=True)
			result[s] = df
		return result
{% endhighlight %}

Now that we have the DataFrames set up we want to extract some useful
information from the data. For the sake of simplicity, thoughout this we ignore
the confidence intervals and use the stated estimates as true values. This loses
a chunk of information which should be considered if any rigorous conclusions are to
made from the data, however we are doing this as a fun exersise and out of
interest.

For each year, the numbers of immigrants are converted into a percentage fo the
total local population. Pandas allows appending a column by simply refering to
it and assigning it a set of values.

Using these percentages, we then compute the growth of the percentage of
immigrants in the local population over the 10 years from 2004 to 2014.

Finally after the columns are added we once again sort the columns to ensure
that they appear in the correct place.

{% highlight python %}
def calc_perc(sub, total):
    return sub / total * 100

def calc_growth(start, end):
    return calc_perc(end - start, start)

def add_percents(df):
    for y in years:
        df[y, 'Percent'] = calc_perc(df[y, 'Estimate'], df[y, 'Population'])
        df[y, 'CI'] = calc_perc(df[y, 'CI +/-'], df[y, 'Population'])
    	
    df['10 Year Growth', 'Val'] = calc_growth(df['2004', 'Percent'],
    	df['2014', 'Percent'])
    df['10 Year Growth', 'CI+'] = ( calc_growth(df['2004', 'Percent'] -
    	df['2004', 'CI'], df['2014', 'Percent'] +	df['2014', 'CI']) -
    	df['10 Year Growth', 'Val'] )
    df['10 Year Growth', 'CI-'] = ( df['10 Year Growth', 'Val'] -
        calc_growth(df['2004', 'Percent'] + df['2004', 'CI'],
        df['2014', 'Percent'] - df['2014', 'CI']) )
    df.sortlevel(axis=1, inplace=True)
{% endhighlight %}

Also add a function which prepends all columns with an additional multi-index
label. Then switch this new label and the previous top level. This means that if
the initial data frame has the headings:

```
      | val1          | val 2         | ...
index | sub 1 | sub 2 | sub 1 | sub 2 | ...
```

After calling this function the headings will be:

```
      | val1          | val2          | ...
      | ind   | ind   | ind   | ind   | ...
index | sub 1 | sub 2 | sub 1 | sub 2 | ...
```

This is useful as we can have the same columns for the two similar data sets for
Non-UK born data and Non-British data, then using this add identifiers to this
data before joining the two tables together.

{% highlight python %}
def add_descriptive_index(df, ind):
    mi = pd.MultiIndex.from_tuples([(ind, x, y) for x, y in df.columns.values])
    df.columns = mi
    return df.swaplevel(0, 1, axis=1)
{% endhighlight %}

With that set up, we just have to call the functions to load the files and
compute the extra columns, before pickling the DataFrames for easy loading later
on.

Load the data into a dictionary `data`.

{% highlight python %}
idx = pd.IndexSlice

filename = './data/v12localareamigrationindicatorsaug15_tcm77-414816.xls'
data = load_data(filename)
nuk = data[sheets[0]]
nbr = data[sheets[1]]
{% endhighlight %}

Once loaded, we extract the population totals for each year. This data will be
stripped out of the data frames later, so we want to keep a copy separate to add
back in later.

We also add a third index to the table so that it has the same index as the rest
of the data.

{% highlight python %}
population = nuk.loc[idx[:, idx[:, 'Population']]]
population.columns = pd.MultiIndex.from_tuples([(x, y, '') for x, y in population.columns.values])
{% endhighlight %}

Now use the functions defined above to add percentages, growth and other data to
each of the dataframes.

{% highlight python %}
add_percents(nuk)
nuk = add_descriptive_index(nuk, 'Non-UK Born')
nuk.sortlevel(axis=1, inplace=True)

add_percents(nbr)
nbr = add_descriptive_index(nbr, 'Non-British')
nbr.sortlevel(axis=1, inplace=True)
{% endhighlight %}

At this point we can look a little at the data.

```
>>> nuk[('10 Year Growth', 'Non-UK Born', 'Val')].dropna().describe()
count    412.000000
mean      64.511163
std       74.077139
min      -76.538462
25%       24.958564
50%       50.170228
75%       88.048223
max      648.936170
Name: (10 Year Growth, Non-UK Born, Val), dtype: float64
```

On average the proportion of non-UK born people in the population has increased
over the past 10 years. In North West Leicestershire (`d['Area Name'].loc[d[('10
Year Growth', 'Non-UK Born', 'Val')].idxmax()]`) this was an increase of over
600%, while 56 areas saw a decrease in numbers (`d[('10 Year Growth', 'Non-UK
Born', 'Val')].loc[d[('10 Year Growth', 'Non-UK Born', 'Val')] < 0].count()`).

We want to combine all the data together into one big DataFrame, so we use some
column slicing to extract what we want, then merge together using an outer join.
This ensures that no data is thrown away. At this point we also have two columns
for `Area Name`, so we tidy that up.

We then need to add the population data back again before saving to a pickle.

{% highlight python %}
nuk = pd.concat([nuk.loc[idx[:, other_cols]], nuk.loc[idx[:, idx[:, :, year_cols]]]], axis=1)
nbr = pd.concat([nbr.loc[idx[:, other_cols]], nbr.loc[idx[:, idx[:, :, year_cols]]]], axis=1)
cols = nbr.columns.difference(nuk.columns)
both = pd.merge(nuk, nbr[cols], left_index=True, right_index=True, how='outer')
both[('Area Name', '')] = both[('Area Name', 'Non-UK Born')]
both.sortlevel(axis=1, inplace=True)
both.drop( [('Area Name', 'Non-UK Born'), ('Area Name', 'Non-British')], axis=1,
    inplace=True)

with_pop = pd.merge(both, population, left_index=True, right_index=True,
    how='outer')
with_pop.sortlevel(axis=1, inplace=True)

with_pop.to_pickle('./data/immigration.pkl')
{% endhighlight %}

The whole file:

{% highlight python %}
"""
Pickles the immigration data

Data available at:
https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/
	migrationwithintheuk/datasets/localareamigrationindicatorsunitedkingdom

Data is released under the Open Government Licence:
	http://www.nationalarchives.gov.uk/doc/open-government-licence/
"""

import pandas as pd

sheets = ['Non-UK Born Population', 'Non-British Population']
years = ['2004', '2005', '2006', '2007', '2008', '2009', '2010', '2011', '2012',
        '2013', '2014']
year_cols = ['Percent', 'CI']
other_cols = ['10 Year Growth', 'Area Name']
	
def load_data(filename):
    with pd.ExcelFile(filename) as xls:
        result = {}
        for s in sheets:
            df = pd.read_excel(xls, s, header=[0, 2]).dropna(how='all')
            df = df.loc[~df.index.str.startswith('95')]
            df.columns.set_levels(['Area Name', '2004', '2005', '2006', '2007',
                '2008', '2009', '2010', '2011', '2012', '2013', '2014'], level=0,
                inplace=True)
            df.columns.set_levels(['CI +/-', 'Estimate', 'Population', ''],
                level=1, inplace=True)
            for year in years:
                df[year] = df[year].apply(lambda x: pd.to_numeric(x, errors='coerce'))
            df.sortlevel(axis=1, inplace=True)
            result[s] = df
        return result

def calc_perc(sub, total):
    return sub / total * 100

def calc_growth(start, end):
    return calc_perc(end - start, start)

def add_percents(df):
    for y in years:
        df[y, 'Percent'] = calc_perc(df[y, 'Estimate'], df[y, 'Population'])
        df[y, 'CI'] = calc_perc(df[y, 'CI +/-'], df[y, 'Population'])
    	
    df['10 Year Growth', 'Val'] = calc_growth(df['2004', 'Percent'],
    	df['2014', 'Percent'])
    df['10 Year Growth', 'CI+'] = ( calc_growth(df['2004', 'Percent'] -
    	df['2004', 'CI'], df['2014', 'Percent'] +	df['2014', 'CI']) -
    	df['10 Year Growth', 'Val'] )
    df['10 Year Growth', 'CI-'] = ( df['10 Year Growth', 'Val'] -
        calc_growth(df['2004', 'Percent'] + df['2004', 'CI'],
        df['2014', 'Percent'] - df['2014', 'CI']) )
    df.sortlevel(axis=1, inplace=True)

def add_descriptive_index(df, ind):
    mi = pd.MultiIndex.from_tuples([(ind, x, y) for x, y in df.columns.values])
    df.columns = mi
    return df.swaplevel(0, 1, axis=1)

idx = pd.IndexSlice

filename = './data/v12localareamigrationindicatorsaug15_tcm77-414816.xls'
data = load_data(filename)
nuk = data[sheets[0]]
nbr = data[sheets[1]]

population = nuk.loc[idx[:, idx[:, 'Population']]]
population.columns = pd.MultiIndex.from_tuples([(x, y, '') for x, y in population.columns.values])

add_percents(nuk)
nuk = add_descriptive_index(nuk, 'Non-UK Born')
nuk.sortlevel(axis=1, inplace=True)

add_percents(nbr)
nbr = add_descriptive_index(nbr, 'Non-British')
nbr.sortlevel(axis=1, inplace=True)

nuk = pd.concat([nuk.loc[idx[:, other_cols]], nuk.loc[idx[:, idx[:, :, year_cols]]]], axis=1)
nbr = pd.concat([nbr.loc[idx[:, other_cols]], nbr.loc[idx[:, idx[:, :, year_cols]]]], axis=1)
cols = nbr.columns.difference(nuk.columns)
both = pd.merge(nuk, nbr[cols], left_index=True, right_index=True, how='outer')
both[('Area Name', '', '')] = both[('Area Name', 'Non-UK Born', '')]
both.sortlevel(axis=1, inplace=True)
both.drop( [('Area Name', 'Non-UK Born'), ('Area Name', 'Non-British')], axis=1,
    inplace=True)

with_pop = pd.merge(both, population, left_index=True, right_index=True,
    how='outer')
with_pop.sortlevel(axis=1, inplace=True)

with_pop.to_pickle('./data/immigration.pkl')
{% endhighlight %}


[1]: https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/migrationwithintheuk/datasets/localareamigrationindicatorsunitedkingdom

[2]: http://www.nationalarchives.gov.uk/doc/open-government-licence/

[Overview]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum %}
[Sourcing data]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-data %}
[Handling results]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-counting-votes %}
[Coding postcodes]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-postcodes %}
[Capping CAPs]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-cap %}
[Counting immigrants]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-chasing-immigrants %}
[Income]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-income %}
[Unemployment]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-unemployment %}
[Mapping data]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-map %}
[Pretending I know stats]:  {{ base }}{% post_url 2016-07-19-plotting-a-referendum-stats %}