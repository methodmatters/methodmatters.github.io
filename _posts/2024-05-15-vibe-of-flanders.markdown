---
layout: post
title: 'The Vibe of Flanders: Part 1'
date: '2024-05-15T06:00:00.000 CET'
author: Method Matters
tags:

- Belgium
- Flanders
- surveys
- Gemeente-Stadsmonitor
- Stadsmonitor
- PCA
- Principal Components Analysis
- FactoMineR
- statistics
- data analysis
- data vizualization
- R


---


What's it like to live in Flanders these days? 

[Flanders](https://en.wikipedia.org/wiki/Flanders){:target="_blank"}, the Northern, Dutch-speaking part of Belgium, conducts a regular survey of the people who live here. The survey is called **De Gemeente-Stadsmonitor** (*The Municipality and City Monitor*), and covers a great many topics, from large societal issues to housing to mobility and climate.

This post describes an analysis of the summary data (aggregated per municipality) from the most-recent survey wave, giving a data-driven view of the main underlying themes of the survey. We will analyze these themes in some detail, and make a map of Flemish municipalities (*gemeenten* in Dutch) and provinces (West Flanders, East Flanders, Flemish Brabant, Antwerp, and Limburg) according to their relationship to the themes and to one another.

In which municipalities and regions do residents feel good about where they live, and in contrast, where do residents say they feel unsafe? In which municipalities and regions do residents use environmentally sustainable transport and where do they seem dependent on the car? Read on to find out!

# The Gemeente-Stadsmonitor Survey

The [**Gemeente-Stadsmonitor**](https://gemeente-stadsmonitor.vlaanderen.be/over-de-monitor){:target="_blank"} (*Municipality and City Monitor*) is conducted every three years by the Agency of the Interior and Statistics Flanders, and the most recent survey wave was conducted in 2023. For the 2023 survey, a representative sample of residents between 17 and 85 years old was sent the survey, and in in total 389,714 people filled it out. According to the website, all 300 Flemish municipalities should be in the data, but in fact only 299 are.[^1]  

The survey contains questions on 11 broad topics, as designated by the creators of the survey: 

- Armoede (*Poverty*) 
- Cultuur en vrije tijd (*Culture and leisure*) 
- Demografie (*Demography*) 
- Klimaat, milieu en natuur (*Climate, environment and nature*) 
- Lokaal bestuur (*Local government*) 
- Mobiliteit (*Mobility*) 
- Onderwijs en vorming (*Education and training*) 
- Samenleven (*Living together*) 
- Werk (*Work*) 
- Wonen en woonomgeving (*Living and living environment*) 
- Zorg en gezondheid (*Care and health*)

For more information about the survey, you can check out [this website](https://gemeente-stadsmonitor.vlaanderen.be/over-de-monitor){:target="_blank"} (in Dutch). The 2023 survey form containing all of the question text and answer options can be found [here](https://gemeente-stadsmonitor.vlaanderen.be/_gatsby/file/83ed6d93b1f641040c79f6c0b8edc6de/vragenlijst_gemeentemonitor_2023.pdf){:target="_blank"}

# Language Use in This Post

The *Gemeente-Stadsmonitor* survey is conducted in Dutch; all survey questions are written in this language. I'm writing this post in English in order for it to be more widely-accessible.

The question descriptions in the charts below will be displayed with the Dutch language descriptions. I'll provide English language translations (created with machine-translation via Co-Pilot) throughout the text and tables. It should be possible to follow everything described in this post, even if you don't know any Dutch!

Finally, though the full name of the survey is the "Gemeente-Stadsmonitor", in the text below I will refer to the survey as the "Stadsmonitor" for simplicity. This is also the term that is used in the Flemish media when talking about the survey and its results.  

# The Data

The data, at the gemeente / municipality level, are [freely available to the public](https://gemeente-stadsmonitor.vlaanderen.be/download-alle-cijfers){:target="_blank"}. You can download subsets of the data, or have all of it in a 100+ tab Excel file.

I downloaded the Excel file with all of the data, and spent a significant amount of effort preparing it for analysis. All of the data preparation and analysis code is available on Github [here](https://github.com/methodmatters/vibe_of_flanders_part_1){:target="_blank"}. 

The present analysis considers only data from the most recent survey wave, conducted in 2023. The data for the analyses below are taken from the answer options that indicate agreement with the question topic. For example, the underlying data for the question "Zich thuis voelen bij mensen in de buurt" ("*Feeling at home with people in the neighborhood*") are the percentage of respondents per municipality who **agree** with this statement. 

The dataset contains 299 rows (1 per gemeente/municipality), with the answers to each question contained in 200 columns.

# Analysis Part 1: Using PCA to Uncover Latent Themes in the Stadsmonitor Data

The goal of the first set of analyses is to understand the latent themes of the survey items. Each survey question is written to assess residents' thoughts or feelings about a specific topic (the 11 subjects outlined above, e.g. Mobiliteit / Mobility). However, it is often the case that survey items have a higher-level grouping that is evidenced by the iter-relationship of responses to the questions.

One common data analytic technique that is often used to bring clarity to this underlying structure is called [PCA (Principal Components Analysis)](https://en.wikipedia.org/wiki/Principal_component_analysis){:target="_blank"}. Principal Components Analysis is a technique that tries to reduce a set of variables into a smaller dimensional space. In the current case, we have variables describing the gemeente / municipality-level responses to the 200 questions in the Stadsmonitor. PCA allows us to find a smaller number of independent components that describe the variation in the responses to these questions. Within this reduced-dimensional space, we are better able understand the relationships among the questions, municipalities and regions.[^2] 

In this blog post, I will use the term *topic* to describe the designation given by the survey authors, and *theme* to describe the data-driven groupings of survey questions from our statistical analysis (e.g. the principal components).

# Revealed Themes (Principal Components)

In this post, we will focus on the top two themes uncovered by the PCA analysis. Our analysis gives us a list of underlying themes (called *Principal Components* or *PCs* in statistical terms), ranked in terms of their importance in explaining the variation in the responses to the survey questions. Each question gets a score (called a *loading* in statistical terms) for each Principal Component. The loadings range in between -1 and +1, and the larger a question's loading on a principal component (either in a positive or negative direction), the more the question is reflective of the theme represented by that component.[^3] 

## Theme 1 (PC 1)

The table below shows the questions with the highest scores (loadings) - both positive and negative - on the first principal component. Examining these questions will give us an idea of the subject matter of the first theme.

As is clear in the table, the first theme is heavily concerned with **feelings about one's place of residence**, in the neighborhood or gemeente / municipality. 

On the positive end of this principal component, we find items that focus on **feeling good about where one lives**. The questions deal with feeling at home, comfortable, safe, having good contacts with neighbors, etc. On the negative end of this principal component, we find questions related to **feeling insecure or unsafe where one lives**. The questions with the largest negative scores concern nuisances in the neighborhood, conflict, attitudes towards diversity, feeling unsafe, and interestingly, trust in the federal government. 

**A Word on Interpretation**

PCA analysis is based upon the underlying correlations among the responses to the survey items at the gemeente / municipality level. We can therefore use the analysis to understand how responses to the survey questions are related to one another. 

Firstly, items that have higher scores on PC 1 are positively correlated with one another, and items that have lower scores on PC 1 are also positively correlated with one another. For example, gemeenten / municipalities that have higher average scores on the item "Feeling at home with people in the neighborhood" (PC 1 loading of .88) also have higher average scores on the item "Satisfaction with contact in the neighborhood" (PC 1 loading of .87). And at the other end of PC 1, gemeenten / municipalities with higher average scores on the item "Feeling of insecurity in the neighborhood" (PC 1 loading of -.73) also have higher average scores on the item "Chat with people of non-Belgian origin" (PC 1 loading of -.72).

Furthermore, items with positive loadings on a given principal component are negatively correlated with items with negative loadings on that component. For example, gemeenten / municipalities with higher average scores on the item "Feeling at home with people in the neighborhood" (PC1 loading of .85) have lower average scores on the item "Feeling of insecurity in the municipality" (PC1 loading of -.74).  

Note that these relationships are correlations, and not causal relationships. The table of loadings below shows that gemeenten / municipalities where residents speak more with people of non-Belgian origin (loading of -.72) also feel less safe in their neighborhood (loading of -.73). However, it is *not* the case that people feel less safe *because* they speak more frequently with individuals of non-Belgian origin. 


<style>

    table { 
        margin-left: auto;
        margin-right: auto;
        table-layout: fixed;
        width: 100%;
    word-wrap: break-word;
    }
    table, th, td {
        border: 1px solid grey;
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
        width: 85px;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
<div style="width:1000px;overflow-x: scroll;">
<table>
 <thead>
  <tr>
   <th style="text-align:center;"> Item - Dutch </th>
   <th style="text-align:center;"> Item - English </th>
   <th style="text-align:center;"> PC 1 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> Sociaal weefsel in de buurt - Zich thuis voelen bij mensen in de buurt </td>
   <td style="text-align:center;"> Social fabric in the neighborhood - Feeling at home with people in the neighborhood </td>
   <td style="text-align:center;"> 0.88 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over contact in de buurt </td>
   <td style="text-align:center;"> Satisfaction with contact in the neighborhood </td>
   <td style="text-align:center;"> 0.87 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over de buurt </td>
   <td style="text-align:center;"> Satisfaction with the neighborhood </td>
   <td style="text-align:center;"> 0.85 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Zich thuis voelen in de buurt </td>
   <td style="text-align:center;"> Feeling at home in the neighborhood </td>
   <td style="text-align:center;"> 0.85 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Sociaal weefsel in de buurt </td>
   <td style="text-align:center;"> Social fabric in the neighborhood </td>
   <td style="text-align:center;"> 0.85 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Sociaal weefsel in de buurt - Mensen in de buurt zijn te vertrouwen </td>
   <td style="text-align:center;"> Social fabric in the neighborhood - People in the neighborhood are trustworthy </td>
   <td style="text-align:center;"> 0.84 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Sociaal weefsel in de buurt - Mensen in de buurt willen hun buren helpen </td>
   <td style="text-align:center;"> Social fabric in the neighborhood - People in the neighborhood want to help their neighbors </td>
   <td style="text-align:center;"> 0.84 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Graag wonen in de gemeente </td>
   <td style="text-align:center;"> Like living in the municipality </td>
   <td style="text-align:center;"> 0.80 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Plaats om fietsen te stallen in of bij de woning </td>
   <td style="text-align:center;"> Place to park bicycles in or at the house </td>
   <td style="text-align:center;"> 0.79 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Duurzaamheid van de woning - Zonnepanelen </td>
   <td style="text-align:center;"> Sustainability of the house - Solar panels </td>
   <td style="text-align:center;"> 0.79 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Private buitenruimte of garage - Garage </td>
   <td style="text-align:center;"> Private outdoor space or garage - Garage </td>
   <td style="text-align:center;"> 0.76 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Netheid van het centrum </td>
   <td style="text-align:center;"> Cleanliness of the center </td>
   <td style="text-align:center;"> 0.76 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Buurthinder: lastiggevallen worden op straat </td>
   <td style="text-align:center;"> Neighborhood nuisance: being harassed on the street </td>
   <td style="text-align:center;"> -0.62 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Diversiteit vriendenkring - Vrienden niet-Belgische herkomst </td>
   <td style="text-align:center;"> Diversity of friends - Friends non-Belgian origin </td>
   <td style="text-align:center;"> -0.63 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Vertrouwen in federale overheid </td>
   <td style="text-align:center;"> Trust in federal government </td>
   <td style="text-align:center;"> -0.63 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen vrije tijd - Bus, tram of metro </td>
   <td style="text-align:center;"> Leisure travel - Bus, tram or metro </td>
   <td style="text-align:center;"> -0.64 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Houding tegenover diversiteit - Teveel verschillende herkomst </td>
   <td style="text-align:center;"> Attitude towards diversity - Too much different origin </td>
   <td style="text-align:center;"> -0.64 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Buurthinder: vandalisme en drugsdealing - Drugsdealing </td>
   <td style="text-align:center;"> Neighborhood nuisance: vandalism and drug dealing - Drug dealing </td>
   <td style="text-align:center;"> -0.69 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Intensiteit van contacten - Praatje met mensen van niet-Belgische herkomst </td>
   <td style="text-align:center;"> Intensity of contacts - Chat with people of non-Belgian origin </td>
   <td style="text-align:center;"> -0.72 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Onveiligheidsgevoel in de buurt </td>
   <td style="text-align:center;"> Feeling of insecurity in the neighborhood </td>
   <td style="text-align:center;"> -0.73 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Buurthinder: milieuhinder - Hondenpoep </td>
   <td style="text-align:center;"> Neighborhood nuisance: environmental nuisance - Dog poop </td>
   <td style="text-align:center;"> -0.73 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Onveiligheidsgevoel in de gemeente </td>
   <td style="text-align:center;"> Feeling of insecurity in the municipality </td>
   <td style="text-align:center;"> -0.74 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Buurthinder: vandalisme en drugsdealing - Vandalisme </td>
   <td style="text-align:center;"> Neighborhood nuisance: vandalism and drug dealing - Vandalism </td>
   <td style="text-align:center;"> -0.76 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Buurthinder </td>
   <td style="text-align:center;"> Neighborhood nuisance </td>
   <td style="text-align:center;"> -0.82 </td>
  </tr>
</tbody>
</table>
</div>

## Theme 2 / PC 2

The questions with the highest scores (both positive and negative) on the second underlying theme are shown in the table below. This theme is heavily focused on **transport and mobility**, important topics in a country where the near-constant traffic jams have enormous [negative impacts on the economy](https://www.brusselstimes.com/360094/over-e4-5-billion-lost-due-to-traffic-jams-in-belgium-last-year){:target="_blank"} and [commuter well-being](https://www.belganewsagency.eu/belgium-ranked-worst-in-european-traffic-behaviour-survey){:target="_blank"}. 

On the positive dimension of the principal component, the questions focus on **environmentally sustainable transportation**. Many of the top items concern *bicycle use* - whether respondents frequently use their bikes, the state of the bike infrastructure, and whether they feel safe while cycling. Mixed in with these items are questions related to walking and being physically active. At the negative end of the second theme, we find mostly items about **frequent car usage**. 


<style>

    table { 
        margin-left: auto;
        margin-right: auto;
        table-layout: fixed;
        width: 100%;
    word-wrap: break-word;
    }
    table, th, td {
        border: 1px solid grey;
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
        width: 85px;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
<div style="width:1000px;overflow-x: scroll;">
<table>
 <thead>
  <tr>
   <th style="text-align:center;"> Item - Dutch </th>
   <th style="text-align:center;"> Item - English </th>
   <th style="text-align:center;"> PC 2 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> Duurzaam verplaatsingsgedrag voor korte afstanden </td>
   <td style="text-align:center;"> Sustainable travel behavior for short distances </td>
   <td style="text-align:center;"> 0.68 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voldoende fietsenstallingen </td>
   <td style="text-align:center;"> Enough bicycle parking </td>
   <td style="text-align:center;"> 0.64 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen </td>
   <td style="text-align:center;"> Environmentally conscious behavior </td>
   <td style="text-align:center;"> 0.64 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Veilig fietsen </td>
   <td style="text-align:center;"> Safe cycling </td>
   <td style="text-align:center;"> 0.63 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voldoende autoluwe en autovrije zones </td>
   <td style="text-align:center;"> Enough car-free and car-free zones </td>
   <td style="text-align:center;"> 0.62 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen vrije tijd - Fiets / elektrische fiets </td>
   <td style="text-align:center;"> Leisure travel - Bike / electric bike </td>
   <td style="text-align:center;"> 0.62 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen vrije tijd - Fiets algemeen </td>
   <td style="text-align:center;"> Leisure travel - Bike in general </td>
   <td style="text-align:center;"> 0.62 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Korte afstanden te voet </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Short distances on foot </td>
   <td style="text-align:center;"> 0.61 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Duurzaam verplaatsingsgedrag voor korte afstanden - Korte afstanden te voet </td>
   <td style="text-align:center;"> Sustainable travel behavior for short distances - Short distances on foot </td>
   <td style="text-align:center;"> 0.61 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over staat van fietsinfrastructuur </td>
   <td style="text-align:center;"> Satisfaction with the condition of cycling infrastructure </td>
   <td style="text-align:center;"> 0.60 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Actief bewegen </td>
   <td style="text-align:center;"> Active movement </td>
   <td style="text-align:center;"> 0.58 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voldoende fietsinfrastructuur </td>
   <td style="text-align:center;"> Enough cycling infrastructure </td>
   <td style="text-align:center;"> 0.58 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen vrije tijd - Te voet </td>
   <td style="text-align:center;"> Leisure travel - On foot </td>
   <td style="text-align:center;"> 0.57 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voldoende openbaar vervoer </td>
   <td style="text-align:center;"> Enough public transport </td>
   <td style="text-align:center;"> 0.56 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Buurthinder: milieuhinder - Zwerfvuil </td>
   <td style="text-align:center;"> Neighborhood nuisance: environmental nuisance - Litter </td>
   <td style="text-align:center;"> -0.54 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen vrije tijd - Autopassagier </td>
   <td style="text-align:center;"> Leisure travel - Car passenger </td>
   <td style="text-align:center;"> -0.57 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Buurthinder: verkeershinder - Snel rijden </td>
   <td style="text-align:center;"> Neighborhood nuisance: traffic nuisance - Fast driving </td>
   <td style="text-align:center;"> -0.57 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen woon-werk/woon-school: dominant vervoermiddel - Auto </td>
   <td style="text-align:center;"> Commuting: dominant mode of transport - Car </td>
   <td style="text-align:center;"> -0.60 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen vrije tijd - Autobestuurder </td>
   <td style="text-align:center;"> Leisure travel - Car driver </td>
   <td style="text-align:center;"> -0.60 </td>
  </tr>
</tbody>
</table>
</div>

## Plotting the Stadsmonitor Questions According to Their Scores on Themes 1 & 2

We can plot the individual survey items in the tables above in the two-dimensional space defined by the scores of each question on the first two principal components, mapping out where the questions fall with regards to one another within this space. The information shown in the plot below is the same as that contained in the above tables, but the visualization provides another way of understanding the relationships among the questions and the themes of the first two principal components. The items are colored by the question topic, as determined by the agency that administered the survey.

Note that we are showing just a subset of the questions in this plot; there are 200 questions, and plotting them all yields in an enormous jumble that is difficult to read and to interpret.

The main concerns of the first theme (displayed on the horizontal or x-axis) - of feelings about one's place of residence - are very clear. On the positive (right-hand) side, we see the questions describing positive feelings about where one lives (e.g. "Zich thuis voelen in de buurt" / "*Feeling at home in the neighborhood*"). On the negative (left-hand) side, we see the items about feeling insecure or unsafe in the place where one lives (e.g. "Onveiligheidsgevoel in de buurt" / "*Feeling of insecurity in the neighborhood*").

The main concerns of the second theme (displayed on the vertical or y-axis) - all about transport and mobility - are also quite clear. At the positive (upper) side, we see the items about using bikes or other environmentally sustainable transportation modes, whereas on the negative (bottom) side, we see items about using the car.

Notice that the question topics (assigned by the survey writers) are grouped into higher level themes by the PCA analysis. For example, the first theme / principal component - about feeling good or bad where one lives - contains survey items from the Wonen en woonomgeving (*Living and living environment*), Samenleven (*Living together*), and Cultuur en vrije tijd (*Culture & leisure*) topics.

![biplot questions points]({{site.baseurl}}/assets/img/2024-05-15-vibe-of-flanders/biplot_questions_points_20240513.png)

# Analysis Part 2: Plotting the Gemeenten / Municipalities and Provinces According to Their Scores on Themes 1 & 2 

We can use the results of our PCA analysis to plot each gemeente / municipality in the two-dimensional space defined by the first two principal components, coloring the points by the color of the Flemish province in which they are located. We can also compute the average of each province along the principal components; these province averages are indicated by the larger dots on the plot below. 

There is a clear ordering of the provinces according to the first theme / principal component (displayed on the horizontal axis), which concerns **feelings about one's place of residence**. Gemeenten / municipalities in West Flanders have, on average, the highest scores on this dimension, indicating that residents in this province *feel the most positive* about where they live. West Flanders is followed closely by Limburg, while Antwerp and East Flanders fall somewhere in the middle. Flemish Brabant has the lowest average scores on the first theme / principal component, indicating that gemeenten / municipalities in this province are the least positive about their place of residence, and that on average *feelings of insecurity* are higher there.  

There is much less variation by province along the second theme / principal component (displayed on the vertical axis). On this **transport and mobility** dimension, Antwerp scores the highest, indicating that gemeenten / municipalities in this province, on average, indicate greater use of *environmentally sustainable transportation*. West Flanders falls in the middle, while the remaining provinces are located on the *frequent car usage* side of this dimension.


![biplot gemeenten municipalities provinces]({{site.baseurl}}/assets/img/2024-05-15-vibe-of-flanders/biplot_omnibus_municp_provs_20240513.png)


## Focusing on the Gemeenten / Municipalities at the Extremes of Themes 1 & 2

In the plot above, I've focused on the center of the coordinates defined by the first two principal components. However, there are gemeenten / municipalities that fall at both positive and negative extremes along both of these two dimensions. The plots below display the gemeenten / municipalities with the most extreme scores at the positive and negative ends of each theme. 

### Highest Scores On Theme / PC 1 - Feelings About One's Place of Residence

The plot below shows the gemeenten / municipalities with the highest scores on theme / principal component 1. These are the places where survey respondents **feel the most positive about where they live**.


![most positive pc 1]({{site.baseurl}}/assets/img/2024-05-15-vibe-of-flanders/biplot_highest_pc1_20240513.png)


### Lowest Scores On Theme / PC 1 - Feelings About One's Place of Residence

The plot below shows the gemeenten / municipalities with the lowest scores on theme / principal component 1. These are the places where survey respondents report the greatest **feelings of insecurity where they live**. 

![most negative pc 1]({{site.baseurl}}/assets/img/2024-05-15-vibe-of-flanders/biplot_lowest_pc1_20240513.png)

### Highest Scores On Theme / PC 2 - Transport & Mobility

The plot below shows the gemeenten / municipalities with the highest scores on theme / principal component 2. These are the places where survey respondents report the greatest use of **environmentally sustainable transportation**; people here regularly walk or use the bike in their daily lives.  


![most positive pc 2]({{site.baseurl}}/assets/img/2024-05-15-vibe-of-flanders/biplot_highest_pc2_20240513.png)


### Lowest Scores On Theme / PC 2 - Transport & Mobility

The plot below shows the gemeenten / municipalities with the lowest scores on theme / principal component 2. These are the places where survey respondents report the highest degrees of **frequent car usage**; people here travel more by car in their daily lives. 


![most negative pc 2]({{site.baseurl}}/assets/img/2024-05-15-vibe-of-flanders/biplot_lowest_pc2_20240513.png)

# Summary and Conclusion

In this blog post, we used principal components analysis to analyze the summary data per gemeente / municipality from the 2023 Stadsmonitor survey. This analysis allowed us to uncover the main themes that underlie the survey responses. We focused here on the first two themes or principal components. 

The first theme concerned **feelings about one's place of residence**. At the positive end of this dimension, we found questions related to *feeling good about where one lives*. At the negative end of this dimension, we found questions related to *feeling insecure or unsafe where one lives*. 

The second theme concerned **transport and mobility**. At the positive end of this dimension, we found questions related to the usage of *environmentally sustainable transportation*, while at the negative end of this dimension, we found questions related to *frequent car usage*.

We then plotted the gemeenten / municipalities and provinces in the two-dimensional space defined by the first two themes / principal components. There was a clear ordering of the provinces according to the first dimension. Respondents in *West Flemish* municipalities feel, on average, the best about their places of residence, while residents in *Flemish Brabant* report the greatest feelings of insecurity about where they live. There was less variation by province according to the second dimension; the one clear difference was that gemeenten / municipalities in *Antwerp* were much more likely to report greater use of environmentally sustainable transportation such as biking or walking, compared to the other provinces. Finally, we made plots of the gemeenten / municipalities that scored the highest on the positive and negative dimensions of the two dimensions, highlighting the places that feel the best vs. worst about where they live, and the places that make the greatest use of environmentally sustainable transport options vs. the places where residents are most likely to make use of the car in their daily lives.

**Coming Up Next**

The next two posts will also focus on the data from the 2023 Stadsmonitor survey. We will further explore the dimensions of the PCA analysis described above, and understand how Flemish gementeen and provinces differ according to the third theme / principal component.[^4] The final post in this series will be a technical one, and will present (with code) the details of the analyses described above.

*Stay tuned!*

---

[^1]:If anyone from the survey team sees this, what happened to Herstappe - NIS Code 73028? 

[^2]:For an excellent introduction to PCA, I highly recommend these [wonderfully clear](https://youtu.be/kpuQqOzQXfM){:target="_blank"} [videos](https://youtu.be/YCwSrtSoZ9M){:target="_blank"} from the Hastie and Tibshirani "Introduction to Statistical Learning" online course. François Husson, a developer of the R package we'll use to do the PCA, has a great [open course about sensographics](https://www.youtube.com/playlist?list=PLnZgp6epRBbQiG5UBFU2eflRKFX8hszRf){:target="_blank"} on YouTube (in French only), and some [interesting](https://www.youtube.com/watch?v=tmApJUWWnyI){:target="_blank"} [tutorials](https://www.youtube.com/watch?v=CTSbxU6KLbM){:target="_blank"} about PCA in English. Also, for a similar application of PCA in a different context, you can check out this [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-09-10-sensographics-and-mapping-consumer/2017-09-10-sensographics-and-mapping-consumer.markdown %} ){:target="_blank"} describing a market-mapping study of beverages, based on consumer responses to a survey describing how the beverages made them feel.

[^3]:The sign of a question's score (e.g. positive or negative) is determined by the direction and magnitude of the variable’s contribution to the principal component and is arbitrary; the relative signs of loadings within a component, which indicate the pattern of correlations among variables, allow us to interpret the component's meaning.

[^4]:Spoiler alert: the third principal component concerns lefty, green, or "crunchy" themes such as organic and fair trade product purchases, vegetarian eating, limiting plastic use, etc.