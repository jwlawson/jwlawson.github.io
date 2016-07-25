---
layout: post
title: Plotting a referendum - Scatter plots
category: interest
sub: true
prev: Mapping the data
prevfile: 2016-07-19-plotting-a-referendum-map
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
Draw various scatter plots of data
"""
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
import seaborn as sns
import textwrap

a = pd.read_pickle('./data/all.pkl')

pairs = [
    ('Non-UK_Growth', 'Result'),
    ('Non-Brit_Growth', 'Result'),
    ('Pct_Non-UK', 'Result'),
    ('Pct_Non-Brit', 'Result'),
    ('Pct_Turnout', 'Result'),
    ('Income', 'Result'),
    ('Pct_Unemployed', 'Result'),
    ('Pct_Unemployed', 'Pct_Turnout'),
    ('Income', 'Pct_Unemployed'),
    ('Total', 'Result'),
    ('Rural_Development', 'Result')
]

names = {
    'Result': 'Proportion of Leave vs Remain votes',
    'Non-UK_Growth': '10 year growth in percent of population not born in UK',
    'Non-Brit_Growth': '10 year growth in percent of population not British',
    'Pct_Non-UK': 'Percent of population not born in the UK (2014)',
    'Pct_Non-Brit': 'Percent of population not British (2014)',
    'Pct_Turnout': 'Percentage turnout at EU referendum',
    'Pct_Unemployed': 'Percent of population aged 16-64 unemployed (May 2015)',
    'Income': 'Median income in GBP',
    'Total': 'Total CAP spend in GBP (2015)',
    'Rural_Development': 'Rural Development fund spend in GBP (2015)' 
}


fig = plt.figure(figsize=(8, 6), dpi=96)
ax = fig.add_subplot(111)
for x, y in pairs:
    ax.clear()
    xj = 0.02 if x=='Pct_Unemployed' else 0
    yj = 0.02 if y=='Pct_Unemployed' else 0
    sns.regplot(x=x, y=y, data=a, ax=ax, robust=True, x_jitter=xj, y_jitter=yj,
        line_kws={'color': sns.xkcd_rgb["slate blue"]})
    title = textwrap.fill('{} compared to {}'.format(names[x], names[y]), 70)
    ax.set_title(title)
    ax.set_xlabel(textwrap.fill(names[x], 40))
    ax.set_ylabel(textwrap.fill(names[y], 40))
    plt.tight_layout()
    fig.savefig('./img/sc/{}-{}.png'.format(x,y))

x = 'Total'
y = 'Result'
ax.clear()
ax.set(xscale='log')
sns.regplot(x=x, y=y, data=a, ax=ax, robust=True, fit_reg=False,
    line_kws={'color': sns.xkcd_rgb["slate blue"]})
title = textwrap.fill('{} compared to {}'.format(names[x], names[y]), 70)
ax.set_title(title)
ax.set_xlabel(textwrap.fill(names[x], 40))
ax.set_ylabel(textwrap.fill(names[y], 40))
plt.tight_layout()
fig.savefig('./img/sc/{}-{}-log.png'.format(x,y))
```

The whole file is available as a [gist].

[gist]: https://gist.github.com/jwlawson/41302a734c6d9b0392cbd60571d755bf#file-scatter-py
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
