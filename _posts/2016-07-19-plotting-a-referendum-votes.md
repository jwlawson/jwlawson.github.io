---
layout: post
title: Plotting a referendum - Results
category: interest
sub: true
nex: Postcodes
nextfile: 2016-07-19-plotting-a-referendum-postcodes
prev: Sourcing data
prevfile: 2016-07-19-plotting-a-referendum-data
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

The Electoral commision has made Referendum results available [online][1]. Note
that the data is available under the [Open Government License v2.0][2].

The following assumes that the `.csv` file is in the current directory and called
`EU-referendum-result-data.csv`.

Having got the data, we construct a pandas DataFrame before pickling the data,
so we can easily access it when we need to.

{% highlight python %}
import pandas as pd

d = pd.read_csv('./EU-referendum-result-data.csv')
d = d.set_index('Area_Code').ix[:, 'Area':].sort_index()

d.to_pickle('results.pkl')
{% endhighlight %}

The data comes in a friendly format, so we don't really need to perform many
conversions to it. All we do is index the data by `Area Code` and throw
away the columns which appear before `Area` before sorting the data by its
index.

The DataFrame now contains the voting records for 382 areas, including the
following information (`d.info()`):

```
Area                       382 non-null object
Electorate                 382 non-null int64
ExpectedBallots            382 non-null int64
VerifiedBallotPapers       382 non-null int64
Pct_Turnout                382 non-null float64
Votes_Cast                 382 non-null int64
Valid_Votes                382 non-null int64
Remain                     382 non-null int64
Leave                      382 non-null int64
Rejected_Ballots           382 non-null int64
No_official_mark           382 non-null int64
Voting_for_both_answers    382 non-null int64
Writing_or_mark            382 non-null int64
Unmarked_or_void           382 non-null int64
Pct_Remain                 382 non-null float64
Pct_Leave                  382 non-null float64
Pct_Rejected               382 non-null float64
```

The final results can be checked to be correct with:

```
>>> d[['Remain', 'Leave']].sum()
Remain    16141241
Leave     17410742
dtype: int64
```

Leave wins by just 1 269 501 votes (try `d[['Remain', 'Leave']].sum().diff()`),
or just 3.78% of the votes (`d[['Remain', 'Leave']].sum().diff().Leave /
d['Valid_Votes'].sum() * 100`).


[1]: http://www.electoralcommission.org.uk/find-information-by-subject/elections-and-referendums/upcoming-elections-and-referendums/eu-referendum/electorate-and-count-information
[2]: http://www.nationalarchives.gov.uk/doc/open-government-licence/version/2/

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
