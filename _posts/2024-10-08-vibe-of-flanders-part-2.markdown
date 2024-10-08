---
layout: post
title: 'The Vibe of Flanders: Part 2'
date: '2024-10-08T06:00:00.000 CET'
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

This blog post is the second installment in a series detailing analyses of the 2023 **De Gemeente-Stadsmonitor** (*The Municipality and City Monitor*) survey, conducted in the region of Flanders in Belgium. You can check out the first post [here]{{ site.baseurl}}({% link _posts/2024-05-15-vibe-of-flanders.markdown %}){:target="_blank"}.

In the [previous post]{{ site.baseurl}}({% link _posts/2024-05-15-vibe-of-flanders.markdown %} ){:target="_blank"}, we used Principal Components Analysis and data visualization techniques to understand the first 2 principal components evident from the survey responses at the municipality level. The results of these analyses showed that the first two principal components concerned *feelings about one's place of residence* and *transport and mobility*, respectively. An analysis of the principal components according to province revealed, for example, that residents in West Flanders felt best about where they lived and that residents in Antwerp Province were most likely to use environmentally sustainable transport.

In this post, we will focus on the third and fourth principal components from the PCA analysis, doing a deep dive into the revealed themes, and map the towns and provinces with the highest and lowest scores on these dimensions.

Read on to learn more!

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

For more information about the survey, you can check out [this website](https://gemeente-stadsmonitor.vlaanderen.be/over-de-monitor){:target="_blank"} (in Dutch). The 2023 survey form containing all of the question text and answer options can be found [here](https://gemeente-stadsmonitor.vlaanderen.be/_gatsby/file/83ed6d93b1f641040c79f6c0b8edc6de/vragenlijst_gemeentemonitor_2023.pdf){:target="_blank"}.

# Language Use in This Post

The *Gemeente-Stadsmonitor* survey is conducted in Dutch; all survey questions are written in this language. I'm writing this post in English in order for it to be more widely-accessible.

The question descriptions in the charts below will be displayed with the Dutch language descriptions. I'll provide English language translations (created with machine-translation via Co-Pilot) throughout the text and tables. It should be possible to follow everything described in this post, even if you don't know any Dutch!

Finally, though the full name of the survey is the "Gemeente-Stadsmonitor", in the text below I will refer to the survey as the "Stadsmonitor" for simplicity. This is also the term that is used in the Flemish media when talking about the survey and its results.  

# The Data

The data, at the gemeente / municipality level, are [freely available to the public](https://gemeente-stadsmonitor.vlaanderen.be/download-alle-cijfers){:target="_blank"}. You can download subsets of the data, or have all of it in a 100+ tab Excel file.

I downloaded the Excel file with all of the data, and spent a significant amount of effort preparing it for analysis. The data preparation code is available on Github [here](https://github.com/methodmatters/vibe_of_flanders_part_1){:target="_blank"}. The code for the analysis presented below is available [here](https://github.com/methodmatters/vibe_of_flanders_part_2/tree/master){:target="_blank"}.

The present analysis considers only data from the most recent survey wave, conducted in 2023. The data for the analyses below are taken from the answer options that indicate agreement with the question topic. For example, the underlying data for the question "Zich thuis voelen bij mensen in de buurt" ("*Feeling at home with people in the neighborhood*") are the percentage of respondents per municipality who **agree** with this statement. 

The dataset contains 299 rows (1 per gemeente/municipality), with the answers to each question contained in 200 columns.

# Analysis Part 1: Using PCA to Uncover Latent Themes in the Stadsmonitor Data

The goal of the first set of analyses is to understand the latent themes of the survey items. Each survey question is written to assess residents' thoughts or feelings about a specific topic (the 11 subjects outlined above, e.g. Mobiliteit / Mobility). However, it is often the case that survey items have a higher-level grouping that is evidenced by the iter-relationship of responses to the questions.

One common data analytic technique that is often used to bring clarity to this underlying structure is called [PCA (Principal Components Analysis)](https://en.wikipedia.org/wiki/Principal_component_analysis){:target="_blank"}. Principal Components Analysis is a technique that tries to reduce a set of variables into a smaller dimensional space. In the current case, we have variables describing the gemeente / municipality-level responses to the 200 questions in the Stadsmonitor. PCA allows us to find a smaller number of independent components that describe the variation in the responses to these questions. Within this reduced-dimensional space, we are better able understand the relationships among the questions, municipalities and regions.[^2] 

In this blog post, I will use the term *topic* to describe the designation given by the survey authors, and *principal components* to describe the data-driven groupings of survey questions from our statistical analysis.

# Principal Components 3 and 4

In this post, we will focus on the third and fourth dimensions uncovered by our PCA analysis. PCA analysis returns a list of underlying themes (called *Principal Components* or *PCs* in statistical terms), ranked in terms of their importance in explaining the variation in the responses to the survey questions. Each question gets a score (called a *loading* in statistical terms) for each Principal Component. The loadings range in between -1 and +1, and the larger a question's loading on a principal component (either in a positive or negative direction), the more the question is reflective of the theme represented by that component.[^3] 

Note that, below, we will begin our discussion with principal component 3. For a detailed description of the first two dimensions from the principal components analysis, please see the [first blog post in this series]{{ site.baseurl}}({% link _posts/2024-05-15-vibe-of-flanders.markdown %} ){:target="_blank"}.

## Principal Component 3

The table below shows the questions with the highest scores (loadings) - both positive and negative - on the third principal component. Examining these questions will give us an idea of the subject matters of the third principal component.

As can be seen in the table below, the third principal component encompasses questions that focus on a variety of different social and societal themes. 

On the positive end of this principal component, we find items that focus on **environmentally conscious behavior** (e.g. vegetarian eating, buying organic products, limiting plastic use, etc.) and **positive attitudes towards diversity** (e.g. believing that having residents from different backgrounds enriches life in the town/municipality). On the negative end of this principal component, we find questions related to **negative attitudes towards diversity** (e.g. having unpleasant neighbors of foreign backgrounds, feeling that municipality residents come from too many different origins, etc.) and being **satisfied with town services** (e.g. satisfaction with facilities for elderly people, childare, etc.). Interestingly, the negative side of PC3 also encompasses items about **subjective poverty and payment difficulties** (suggesting that municipalities that score low on this dimension are less wealthy) and **working in one's own municipality** (suggesting that more residents in municipalities that score low on this dimension work where they live).


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
   <th style="text-align:center;"> PC3 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Vegetarisch eten </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Vegetarian eating </td>
   <td style="text-align:center;"> 0.70 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Houding tegenover diversiteit </td>
   <td style="text-align:center;"> Attitude towards diversity </td>
   <td style="text-align:center;"> 0.70 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Bioproducten gekocht </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Bought organic products </td>
   <td style="text-align:center;"> 0.67 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Fair trade </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Fair trade </td>
   <td style="text-align:center;"> 0.64 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Plastic beperkt </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Limited plastic </td>
   <td style="text-align:center;"> 0.62 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Thuiswerk </td>
   <td style="text-align:center;"> Home work </td>
   <td style="text-align:center;"> 0.60 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Vertrouwen in Europese overheid </td>
   <td style="text-align:center;"> Trust in European government </td>
   <td style="text-align:center;"> 0.56 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Houding tegenover diversiteit - Verschillende herkomst verrijking </td>
   <td style="text-align:center;"> Attitude towards diversity - Different origin enrichment </td>
   <td style="text-align:center;"> 0.55 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen </td>
   <td style="text-align:center;"> Environmentally conscious behavior </td>
   <td style="text-align:center;"> 0.54 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Sportparticipatie </td>
   <td style="text-align:center;"> Sports participation </td>
   <td style="text-align:center;"> 0.53 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Weggooien eten beperken </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Limit throwing away food </td>
   <td style="text-align:center;"> 0.50 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Vertrouwen in medemens </td>
   <td style="text-align:center;"> Trust in fellow human beings </td>
   <td style="text-align:center;"> 0.50 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over zicht op groen vanuit de woning </td>
   <td style="text-align:center;"> Satisfaction with view of green from the house </td>
   <td style="text-align:center;"> 0.50 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Kraantjeswater als drinkwater </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Tap water as drinking water </td>
   <td style="text-align:center;"> 0.48 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Seizoensgroenten </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Seasonal vegetables </td>
   <td style="text-align:center;"> 0.48 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Houding tegenover diversiteit - Goed samenleven </td>
   <td style="text-align:center;"> Attitude towards diversity - Good living together </td>
   <td style="text-align:center;"> 0.48 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Vervoermiddelenbezit - Abonnement openbaar vervoer </td>
   <td style="text-align:center;"> Vehicle ownership - Public transport subscription </td>
   <td style="text-align:center;"> 0.41 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Betalingsmoeilijkheden </td>
   <td style="text-align:center;"> Payment difficulties </td>
   <td style="text-align:center;"> -0.41 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Subjectieve armoede </td>
   <td style="text-align:center;"> Subjective poverty </td>
   <td style="text-align:center;"> -0.42 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Werken in eigen gemeente </td>
   <td style="text-align:center;"> Working in own municipality </td>
   <td style="text-align:center;"> -0.42 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over uitgaansgelegenheden </td>
   <td style="text-align:center;"> Satisfaction with nightlife </td>
   <td style="text-align:center;"> -0.43 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over kinderopvang </td>
   <td style="text-align:center;"> Satisfaction with childcare </td>
   <td style="text-align:center;"> -0.44 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over ouderenvoorzieningen </td>
   <td style="text-align:center;"> Satisfaction with elderly facilities </td>
   <td style="text-align:center;"> -0.46 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Houding tegenover diversiteit - Teveel verschillende herkomst </td>
   <td style="text-align:center;"> Attitude towards diversity - Too much different origin </td>
   <td style="text-align:center;"> -0.49 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Houding tegenover diversiteit - Onprettig buren andere herkomst </td>
   <td style="text-align:center;"> Attitude towards diversity - Unpleasant neighbors other origin </td>
   <td style="text-align:center;"> -0.58 </td>
  </tr>
</tbody>
</table>
</div>

## Principal Component 4


The questions with the highest scores (both positive and negative) on the fourth principal component are shown in the table below. 

As can be seen from an examination of the items in the table, this principal component deals with two different topics at its positive and negative poles.  

On the positive end of this principal component, we find items that focus on **satisfaction with one's local government** (e.g. trust in government, satisfaction with communication towards and consultation of residents, satisfaction with contact with the municipality, etc.). On the negative end of this principal component, we find questions related to **bike usage** (owning and using bicycles for for short distances, leisure travel, etc.) and **sustainable living** (e.g. having homes which are energy efficient and well-insulated & commitment to climate-friendly investments; note that bike use could also be seen as a facet of sustainable living).

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
   <th style="text-align:center;"> PC4 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> Voldoende consultatie van inwoners - Tevreden omgaan met vragen </td>
   <td style="text-align:center;"> Enough consultation of residents - Satisfied dealing with questions </td>
   <td style="text-align:center;"> 0.67 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Vertrouwen in gemeentebestuur </td>
   <td style="text-align:center;"> Trust in municipal government </td>
   <td style="text-align:center;"> 0.67 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voldoende consultatie van inwoners </td>
   <td style="text-align:center;"> Enough consultation of residents </td>
   <td style="text-align:center;"> 0.63 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over communicatie van gemeentebestuur </td>
   <td style="text-align:center;"> Satisfaction with municipal communication </td>
   <td style="text-align:center;"> 0.59 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voldoende consultatie van inwoners - Consulatie bij veranderingen </td>
   <td style="text-align:center;"> Enough consultation of residents - Consultation with changes </td>
   <td style="text-align:center;"> 0.54 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voldoende informatie door gemeente - Beslissingen gemeente/stad </td>
   <td style="text-align:center;"> Enough information by municipality - Decisions municipality/city </td>
   <td style="text-align:center;"> 0.51 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Contact met de gemeente </td>
   <td style="text-align:center;"> Contact with the municipality </td>
   <td style="text-align:center;"> 0.50 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over loketvoorzieningen </td>
   <td style="text-align:center;"> Satisfaction with counter facilities </td>
   <td style="text-align:center;"> 0.45 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Tevredenheid over staat van voetpaden </td>
   <td style="text-align:center;"> Satisfaction with the condition of sidewalks </td>
   <td style="text-align:center;"> 0.41 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voldoende informatie door gemeente </td>
   <td style="text-align:center;"> Enough information by municipality </td>
   <td style="text-align:center;"> 0.41 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Duurzaamheid van de woning - Isolatie muren </td>
   <td style="text-align:center;"> Sustainability of the house - Wall insulation </td>
   <td style="text-align:center;"> -0.41 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Voortransport - vervoermiddel - Fiets / elektrische fiets </td>
   <td style="text-align:center;"> Pre-transport - mode of transport - Bike / electric bike </td>
   <td style="text-align:center;"> -0.41 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Vervoermiddelenbezit - Elektrische fiets </td>
   <td style="text-align:center;"> Vehicle ownership - Electric bike </td>
   <td style="text-align:center;"> -0.43 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Inzetten op klimaatvriendelijke investeringen </td>
   <td style="text-align:center;"> Commitment to climate-friendly investments </td>
   <td style="text-align:center;"> -0.45 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Duurzaamheid woning - energiezuinig </td>
   <td style="text-align:center;"> Sustainability house - energy efficient </td>
   <td style="text-align:center;"> -0.50 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Vervoermiddelenbezit - Fiets </td>
   <td style="text-align:center;"> Vehicle ownership - Bike </td>
   <td style="text-align:center;"> -0.58 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen vrije tijd - Fiets / elektrische fiets </td>
   <td style="text-align:center;"> Leisure travel - Bike / electric bike </td>
   <td style="text-align:center;"> -0.60 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Verplaatsingen vrije tijd - Fiets algemeen </td>
   <td style="text-align:center;"> Leisure travel - Bike in general </td>
   <td style="text-align:center;"> -0.60 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Milieubewust handelen - Korte afstanden met fiets </td>
   <td style="text-align:center;"> Environmentally conscious behavior - Short distances by bike </td>
   <td style="text-align:center;"> -0.62 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> Duurzaam verplaatsingsgedrag voor korte afstanden - Korte afstanden met fiets </td>
   <td style="text-align:center;"> Sustainable travel behavior for short distances - Short distances by bike </td>
   <td style="text-align:center;"> -0.62 </td>
  </tr>
</tbody>
</table>
</div>

## Plotting the Stadsmonitor Questions According to Their Scores on Principal Components 3 & 4

We can plot the individual survey items in the tables above in the two-dimensional space defined by the scores of each question on the third and fourth principal components, mapping out where the questions fall with regards to one another within this space. The information shown in the plot below is the same as that contained in the above tables, but the visualization provides another way of understanding the relationships among the questions and the themes of the third and fourth principal components. The items are colored by the question topic, as determined by the agency that administered the survey.

Note that we are showing just a subset of the questions in this plot; there are 200 questions, and plotting them all yields in an enormous jumble that is difficult to read and to interpret.

The main concerns of principal component 3 (displayed on the horizontal or x-axis) come through clearly in the plot. On the positive (right-hand) side of principal component 3, we see questions in yellow describing environmentally conscious behavior (e.g. "Vegetarisch eten" / "*Vegetarian eating*") and questions in red describing positive attitudes towards diversity (e.g. "Verschillende herkomst verrijking" / "*Different origins bring enrichment*"). On the negative (left-hand) side, we see items in light purple and green describing satisfaction with town services (e.g. "Tevredenheid over kinderopvang" / "*Satisfaction with childcare*"), items in red describing negative attitudes towards diversity (e.g. "Onprettig buren andere herkomst" / "*Unpleasant neighbors with foreign backgrounds*") and items in black representing subjective poverty and payment difficulties (e.g. "Subjectieve armoede" / "*Subjective poverty*").

The main concerns of the fourth principal component (displayed on the vertical or y-axis) are also evident in the plot. At the positive (upper) side of component 4, we see the items about satisfaction with local government in light blue (e.g. "Vertrouwen in gemeentebestuur" / "*Trust in municipal government*"). Conversely, at the negative pole (bottom end) of this dimension, we see items in red-violet and yellow representing bike use (e.g. "Verplaatsingen vrije tijd - Fiets algemeen" / "*Leisure travel - Bike in general*") and items in dark blue representing sustainable living (e.g. "Duurzaamheid woning - energiezuinig" / "*Sustainability house - energy efficient*").

![biplot questions points]({{site.baseurl}}/assets/img/2024-10-08-vibe-of-flanders-part-2/biplot_questions_points_20240807.png)

# Analysis Part 2: Plotting the Gemeenten / Municipalities and Provinces According to Their Scores on Principal Components 3 & 4 

We can use the results of our PCA analysis to plot each gemeente / municipality in the two-dimensional space defined by the third and fourth principal components, coloring the points according to the Flemish province in which they are located. We can also compute the average of each province along the principal components; these province averages are indicated by the larger dots on the plot below. 

There is a clear ordering of the provinces according to the third principal component (displayed on the horizontal axis). The positive end of this component concerns **environmentally conscious behaviors & positive attitudes towards diversity**. Gemeenten / municipalities in Flemish Brabant have, on average, the highest scores on this dimension, indicating that residents in this province are more likely to report *performing environmentally conscious behaviors* and *having positive attitudes towards diversity*. The provinces of East Flanders and Antwerp fall somewhere in the middle, while Limburg falls more to the lower side of principal component 3. The negative end of principal component 3 concerns **negative attitudes towards diversity and satisfaction with town services**; the province with the lowest average score is West Flanders. The average low score on principal component three indicates that residents in West Flanders report on average more *negative attitudes towards diversity* and are also more *satisfied with town services* than their fellow citizens in the other Flemish provinces.   

There is much less variation by province along the fourth principal component (displayed on the vertical axis). The one data point that sticks out to me is the low average score for Antwerp province on principal component 4. This indicates that residents in Antwerp province, on average, report **greater bike use and more environmentally sustainable behaviors** than residents in the other Flemish provinces. 


![biplot gemeenten municipalities provinces]({{site.baseurl}}/assets/img/2024-10-08-vibe-of-flanders-part-2/biplot_omnibus_municp_provs_20240807.png)


## Focusing on the Gemeenten / Municipalities at the Extremes of Principal Components 3 & 4

In the plot above, I've focused on the center of the coordinates defined by the principal components three and four. However, there are gemeenten / municipalities that fall at both positive and negative extremes along both of these two dimensions. The plots below display the gemeenten / municipalities with the most extreme scores at the positive and negative ends of each principal component. 

### Highest Scores On PC 3 - Environmentally Conscious Behavior & Positive Attitudes Towards Diversity

The plot below shows the gemeenten / municipalities with the highest scores on principal component 3. These are the places where survey respondents say that they engage in **environmentally conscious behavior** and have **positive attitudes towards diversity**.


![most positive pc 3]({{site.baseurl}}/assets/img/2024-10-08-vibe-of-flanders-part-2/biplot_highest_pc3_20240807.png)


### Lowest Scores On PC 3 - Negative Attitudes Towards Diversity & Satisfaction With Town Services 

The plot below shows the gemeenten / municipalities with the lowest scores on theme / principal component 3. These are the places where survey respondents report **negative attitudes towards diversity, satisfaction with town services, and subjective poverty**. 

![most negative pc 3]({{site.baseurl}}/assets/img/2024-10-08-vibe-of-flanders-part-2/biplot_lowest_pc3_20240807.png)

### Highest Scores On PC 4 - Satisfaction With Local Government

The plot below shows the gemeenten / municipalities with the highest scores on principal component 4. These are the places where survey respondents report the greatest **satisfaction with their local government**.  


![most positive pc 2]({{site.baseurl}}/assets/img/2024-10-08-vibe-of-flanders-part-2/biplot_highest_pc4_20240807.png)


### Lowest Scores On PC 4 - Bike Usage and Sustainable Living

The plot below shows the gemeenten / municipalities with the lowest scores on principal component 4. These are the places where survey respondents report the highest degrees of **bike usage and sustainable living**. 


![most negative pc 2]({{site.baseurl}}/assets/img/2024-10-08-vibe-of-flanders-part-2/biplot_lowest_pc4_20240907.png)

# Summary and Conclusion

In this blog post, we used principal components analysis to analyze the summary data per gemeente / municipality from the 2023 Stadsmonitor survey. This analysis followed up on the [first post in this series]{{ site.baseurl}}({% link _posts/2024-05-15-vibe-of-flanders.markdown %}){:target="_blank"}; we focused here on the third and fourth principal components of our PCA analysis. 

The third principal component concerned **environmentally conscious behavior and positive attitudes towards diversity** at the positive end of the dimension and **negative attitudes towards diversity and satisfaction with town services** on the negative end. The fourth principal component concerned **satisfaction with one’s local government** at the positive end and **bike usage and sustainable living** at the negative end.

We then plotted the gemeenten / municipalities and provinces in the two-dimensional space defined by principal components 3 and 4. There was a clear ordering of the provinces according to the third principal component. Respondents *Flemish Brabant* are more likely to report performing environmentally conscious behaviors and having positive attitudes towards diversity, while respondents in *West Flemish* municipalities feel, on average, greater negative attitudes towards diversity and greater satisfaction with town services. 

There was less variation by province according to the fourth principal component; the one clear difference was that gemeenten / municipalities in *Antwerp province* were much more likely to report bicycle ownership and use and were more likely to engage in environmentally sustainable behaviors. Finally, we made plots of the gemeenten / municipalities that scored the highest on the positive and negative poles of the third and fourth principal components.

This is the final post examining the themes revealed by our principal component analysis of the 2023 Stadsmonitor survey. In contrast to the first post, the separation of themes between the principal components was less clear here. For example, both principal components 3 and 4 contained items related to environmentally-friendly behaviors (e.g. environmentally conscious behavior like fair trade for PC 3 and sustainable living practices such as energy efficient housing for PC 4) and attitudes towards local government (e.g. satisfaction with services for PC 3 and satisfaction with communication, consultation and contact for PC 4). While there is a logic underlying this separation, the differences are somewhat subtle. This, combined with the fact that each subsequent principal component explains less variation in the underlying survey responses, suggests that we have reached the limit of what we can learn from our PCA analysis of the Stadsmonitor survey data.

**Coming Up Next**

There are two posts coming in the hopefully not-too-distant future. The first will be a technical deep-dive into the PCA analyses presented in this post and the previous one, and we will describe (with code) the details of the analyses presented above. The second is based on a talk that I gave at the Royal Statistical Society of Belgium with [Cesar Legendre](https://www.linkedin.com/in/cesarlegendre/){:target="_blank"} of [Prophecy Labs](https://www.prophecylabs.com/){:target="_blank"}. In this post, we will describe lessons learned from our professional experiences in doing data science in an organizational context. 

*Stay tuned!*

---

[^1]:If anyone from the survey team sees this, what happened to Herstappe - NIS Code 73028? 

[^2]:For an excellent introduction to PCA, I highly recommend these [wonderfully clear](https://youtu.be/kpuQqOzQXfM){:target="_blank"} [videos](https://youtu.be/YCwSrtSoZ9M){:target="_blank"} from the Hastie and Tibshirani "Introduction to Statistical Learning" online course. François Husson, a developer of the R package we'll use to do the PCA, has a great [open course about sensographics](https://www.youtube.com/playlist?list=PLnZgp6epRBbQiG5UBFU2eflRKFX8hszRf){:target="_blank"} on YouTube (in French only), and some [interesting](https://www.youtube.com/watch?v=tmApJUWWnyI){:target="_blank"} [tutorials](https://www.youtube.com/watch?v=CTSbxU6KLbM){:target="_blank"} about PCA in English. Also, for a similar application of PCA in a different context, you can check out this [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-09-10-sensographics-and-mapping-consumer/2017-09-10-sensographics-and-mapping-consumer.markdown %} ){:target="_blank"} describing a market-mapping study of beverages, based on consumer responses to a survey describing how the beverages made them feel.

[^3]:The sign of a question's score (e.g. positive or negative) is determined by the direction and magnitude of the variable’s contribution to the principal component and is arbitrary; the relative signs of loadings within a component, which indicate the pattern of correlations among variables, allow us to interpret the component's meaning.