---
layout: post
title: Plotting a referendum - Sourcing data
category: stats
sub: true
prev: Overview
prevfile: 2016-07-19-plotting-a-referendum
nex: Counting votes
nextfile: 2016-07-19-plotting-a-referendum-votes
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
    * [Counting votes]
    * [Coding postcodes]
    * [The CAP]
    * [Immigration]
    * [Income]
    * [Unemployment]
* [Mapping data]
* [Pretending I know stats]

---

Before playing around with the data, we first must get hold of it. This meant a
large amount of seraching around and trying to piece together datasets from the
vast number of government websites, each with different formats, styles and ideas.

The python scripts all assume that the files downloaded here are in a
subdirectory `raw`.

#### CAP data

The CAP spending data is required to be freely available by the EU. There is a
dedicated website for this, which provides an online lookup as well as an option
to download the data.

Go to the [download page](http://cap-payments.defra.gov.uk/Download.aspx) and
grab the 2015 dataset.

#### Map info

The CAP data is stored by postcode, rather than by voting ward. As such we need
to grab the postcode information to link each postcode back to the voting ward
it lies in.

The point data for the centers of all UK postcodes is available as part of
Ordnance Survey's Open Data scheme, so head over to
[their website](https://www.ordnancesurvey.co.uk/opendatadownload/products.html).
Here we need their `Code-Point Open` data and while you're there you should also
get their `Boundary-Line` data as a `ESRI Shapefile`.

The Boundary-Line data is the shapefile containing polygons of all the electoral
and administrative bundaries across the country. We will be using the District
Electoral boundaries to draw the maps.

#### Immigration data

The Office for National Statistics provides immigration data split into local
areas. This dataset is huge, with much more than we need, but is the best I
could find.

Go [here][ons-imm] and download the excel file.

#### Referendum results

The Electoral Commission released the EU referendum results online, available
[here][ec-res], so grab the full dataset.

#### Nomis - unemployment and income

The best source for data on income and unemployment by local area was from the
Office for National Statistics's [Nomis website](https://www.nomisweb.co.uk/),
which provides a range of data on the UK labour markets. We need to download
two sets of data:

<div class="panel panel-default">
<div class="panel-heading">Unemployment data</div>
<table class="table">
<tr><td>Dataset</td><td>Claimant count &rarr; Claimant count by sex and age</td></tr>
<tr><td>Geography</td><td>local authorities: district / unitary (as of April 2015)</td></tr>
<tr><td>Date</td><td>May 2016</td></tr>
<tr><td>Age</td><td>All (16+)</td></tr>
<tr><td>Rates</td><td>Claimant count AND Claimants as a proportion of residents aged 16-64</td></tr>
<tr><td>Sex</td><td>Total</td></tr>
<tr><td></td><td>Include Area codes</td></tr>
</table>
</div>

Rename the resulting file `nomis-unemployment.csv`.

<div class="panel panel-default">
<div class="panel-heading">Income data</div>
<table class="table">
<tr><td>Dataset</td><td>Annual Survey of Hours and Earnings &rarr; annual survey of hours and earnings - resident analysis</td></tr>
<tr><td>Geography</td><td>local authorities: district / unitary (as of April 2015)</td></tr>
<tr><td>Date</td><td>2015</td></tr>
<tr><td>Pay and hours</td><td>Annual pay - gross</td></tr>
<tr><td>Sex & full/part-time</td><td>full-time workers</td></tr>
<tr><td>Variable</td><td>median</td></tr>
<tr><td></td><td>Include Area codes</td></tr>
</table>
</div>

Rename the resulting file `nomis-media-income.csv`.


[Overview]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum %}
[Sourcing data]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-data %}
[Counting votes]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-votes %}
[Coding postcodes]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-postcodes %}
[The CAP]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-cap %}
[Immigration]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-immigrants %}
[Income]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-income %}
[Unemployment]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-unemployment %}
[Mapping data]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-map %}
[Pretending I know stats]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-stats %}
[ons-imm]: https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/migrationwithintheuk/datasets/localareamigrationindicatorsunitedkingdom
[ec-res]: http://www.electoralcommission.org.uk/find-information-by-subject/elections-and-referendums/upcoming-elections-and-referendums/eu-referendum/electorate-and-count-information