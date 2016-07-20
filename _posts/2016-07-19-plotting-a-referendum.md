---
layout: post
title: Plotting a referendum
category: stats
nex: Sourcing data
nextfile: 2016-07-19-plotting-a-referendum-data
excerpt_separator: <!--end-exceprt-->
---

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
<!--end-exceprt-->

---

This project was really just an attempt to learn more about handling data and drawing maps. The results and statements are provided with little to no support, no confidence intervals are considered and no rigour is provided. Remember: correlation does not imply caustation, and this data is not entirely accurate. 

## Referendum results

The election was different to all others that I am aware of in England. Every 
votes counted individually; at the end of the day the result was decided on 
precisely how many people had voted each way. None of the usual 
first-past-the-post, regional representation. As such the paper's and 
broadcaster's usual elections maps faced a problem. They had all been designed 
to colour each region by what party had won. In the referendum that didn't 
matter so much as the number of votes each way. 

Many places chose to separate the map into 4 colours: solid wins for each side 
and narrow wins for each side, with narrow usually meaning a 5% or 10% lead. So 
Newcastle with a 

## Correlations

#### Income

#### Unemployment

#### CAP Spending

At the time of the vote there was a feeliung that many of the regions which benefit the most from EU spending had in fact voted to leave, and so stop receiving this spending. The majority of EU spending comes in two forms: the CAP and the European Strucutral and Investment Funds. The EU requires that all CAP expenditure is freely available, so we consider that here.

The CAP (Common Agricultural Policy) is responsible for a large proportion of the UK's spending of EU money, bringing in around £4bn each year. Roughly speaking, it provides subsidies for landowners with agricultural land.

[![Cap map][capmap]][largecapmap]

As such, one would expect that CAP spending would be related to the amount of land in each region. This can be seen to some extent on the map, with the most spending evident in rural Scotland and Wales.

However when compared to the votes cast in the EU referendum, the amount of CAP spending seems to make little difference.

[![CAP Spend vs Result][capresult]][largecapresult]

It would be more interesting to also study the ESIF spending in rural areas. These strucutral and investment projects may have more of an effect on a local area than the CAP which ultimately is paid only to the landowners. In the period 2014-2020, the UK was expected to receive around £10.8bn for the ESIF (see [Vince Cable's letter][esif letter]).

#### Immigration

Imimgration was widely touted as the reason many voted to leave the EU. By leaving, we were told, the UK could regain control over its borders and prevent the free movement of labour. This free movement has frequently been criticized, backed by the popular mantra that immigrants are stealing jobs and claiming benefits, even though it has been shown that overall immigrants to the UK tend to be a net gain for the economy (see [Guardian][guardian immigrants] or [FT][FT immigrants] on research from UCL).

None of the data I considered looked at only EU immigrants. This could skew any results, as the vote was at most about restricting EU immigration, however it could be argued that those most against immigration wouldn't care too much about where immigrants come from.

There are two main aspects of immigration considered below:

 - The proportion of immigrants in the local population
 - The growth in this proportion over 10 years

[![Immigration map][immmap]][largeimmmap]
[![Immigration growth map][imgmap]][largeimgmap]

Both seem to have different effects on how a region voted in the referendum. Those more diverse areas, where the proportion of immigrants is higher, tended to vote more strongly to remain in the EU. On the other hand, those areas where the proportion of immigrants has grown most rapidly voted primarily to leave the EU.

[![Immigration vs result][immres]][largeimmres]
[![Immigration growth vs result][imgres]][largeimgres]



{% include base.html %}
[Overview]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum %}
[Sourcing data]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-data %}
[Handling results]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-votes %}
[Coding postcodes]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-postcodes %}
[Capping CAPs]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-cap %}
[Counting immigrants]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-immigrants %}
[Income]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-income %}
[Unemployment]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-unemployment %}
[Mapping data]: {{ base }}{% post_url 2016-07-19-plotting-a-referendum-map %}
[Pretending I know stats]:  {{ base }}{% post_url 2016-07-19-plotting-a-referendum-stats %}

[capmap]: {{ base }}/assets/2016-07-19/cap_96.png "Map showing CAP spend by region"
[largecapmap]: {{ base }}/assets/2016-07-19/cap_256.png
[capresult]: http://placehold.it/500x350
[largecapresult]: http://placehold.it/500x350

[immmap]: {{ base }}/assets/2016-07-19/percent_non_brit_96.png "Map showing proportion of population of non-British people by region"
[largeimmmap]: {{ base }}/assets/2016-07-19/percent_non_brit_256.png
[imgmap]: {{ base }}/assets/2016-07-19/growth_non_brit_96.png "Map showing growth in proportion of immigrant population by region"
[largeimgmap]: {{ base }}/assets/2016-07-19/growth_non_brit_256.png

[immres]: http://placehold.it/500x350
[largeimmres]: http://placehold.it/500x350
[imgres]: http://placehold.it/500x350
[largeimgres]: http://placehold.it/500x350

[esif letter]: https://www.gov.uk/government/publications/eu-structural-funds-uk-allocations-2014-to-2020
[guardian immigrants]: https://www.theguardian.com/uk-news/2014/nov/05/eu-migrants-uk-gains-20bn-ucl-study
[FT immigrants]: http://www.ft.com/cms/s/0/c49043a8-6447-11e4-b219-00144feabdc0.html
