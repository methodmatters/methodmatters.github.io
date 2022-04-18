---
layout: post
title: 'Text Analysis of Job Descriptions for Data Scientists, Data Engineers, Machine Learning Engineers and Data Analysts'
date: '2022-04-18 09:00:00.000'
author: Method Matters
tags: 
- Europe
- data
- data jobs
- cluster analysis
- data visualization
- Python
- R
- text analysis
- NLP
- natural language processing
- wordclouds
- word clouds
- text analysis
- O*NET
- Quanteda
- SkillsML
- seaborn
- cluster map
- heatmap
- job descriptions
- labor market
- recruitment
- data scientist
- data analyst
- data engineer
- machine learning engineer

---


# Introduction

In the [previous post]{{ site.baseurl }}({% link _posts/2022-01-23-data-jobs-europe.markdown %} ){:target="_blank"}, the intrepid [Jesse Blum](https://www.linkedin.com/in/jblum1/){:target="_blank"} and I analyzed metadata from over 6,500 job descriptions for data roles in seven European countries. In this post, we'll apply text analysis to those job postings to better understand the technologies and skills that employers are looking for in data scientists, data engineers, data analysts, and machine learning engineers. 

In this post we present results from text analyses that show that: 
- **Data analysts** are expected to have skills in reporting, dashboarding, data analysis, and office suite software. 
- **Data scientists** are expected to know more about data science, statistics, mathematics, and making predictions. 
- **Data engineers** are expected to industrialize organizrations' cloud and data architecture and infrastructure. 
- **Machine learning** engineers are expected to use artificial intelligence and deep learning frameworks such as TensorFlow and Pytorch.

The results of this analysis complement and extend the results we presented last time, showing that employers have distinct visions of the (mostly technical & software-related) skillsets that data analysts, data scientists, data engineers, and machine learning engineers should possess. 

# The Data

The data come from a web scraping program developed by Jesse and myself. Every 2 weeks, we scraped job advertisements from a major job portal website, extracting all jobs posted within the previous 2-week period for the following job titles: Data Engineer, Data Analyst, Data Scientist and Machine Learning Engineer for the following countries: the United Kingdom, Ireland, Germany, France, the Netherlands, Belgium and Luxembourg. 

We started data collection mid-August and finished by the end of December, 2021, ending up with 6,590 job descriptions scraped. All the data and code used for this analysis are available on [Github](https://github.com/methodmatters/data-jobs-europe-2){:target="_blank"}. Feedback welcome! 

# Results

Our dataset includes job descriptions for data roles across four languages (English, French, Dutch and German). We wanted to see if there were any differences in word usage among the different roles (data scientist, data engineer, machine learning engineer and data analyst), and therefore conducted language-specific analyses to contrast and compare the roles according to the words used to describe the job openings.

## Word Clouds

Our first set of analyses uses a great [R function](https://search.r-project.org/CRAN/refmans/wordcloud/html/comparison.cloud.html){:target="_blank"} to create comparison clouds. This type of analysis allows us to compare the frequency of words across groups of documents, and highlight words that appear more in a given group versus the others. 

Jesse and I are more comfortable in English, French, and Dutch than German, so we limited our analysis to those three languages. However, there were far fewer Dutch job descriptions than for the other two, so the resulting Dutch comparison cloud was not particularly informative. Below, we focus on the English and French wordclouds and what they reveal about employers' expectations for the different roles.

The French word cloud looks like this:

![French comparison cloud]({{site.baseurl}}/assets/img/2022-04-18-data-jobs-europe-2-text/comparison_cloud_fr_data_jobs.png)

The English word cloud looks like this:

![English comparison cloud]({{site.baseurl}}/assets/img/2022-04-18-data-jobs-europe-2-text/comparison_cloud_en_data_jobs.png)


Overall, we found that there were clear differences between the roles in the language used in the job advertisements. Furthermore, these differences were largely consistent across the English and French language job ads. 

The following table summarizes the comparison:

<style>


    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        width: 85px;
        vertical-align: middle;
    }
    .wide {
        width: 100%; 
    }

</style>
<div style="width:1000px;overflow-x: scroll;">
<table>
    <tr>
        <td><b>Role</b></td>
        <td><b>French (N = 1,349) Job Descriptions</b></td>
        <td><b>English (N = 3,869) Job Descriptions</b></td>
    </tr>
    <tr>
        <td><b>Data analysts</b></td>
        <td><ul><li> Expected to know about <b>data analysis</b> (<i>analyse</i>), <b>reporting</b> (<i>reporting</i>, <i>tableau de bord</i>), and <b>data visualization</b> (<i>visualisation</i>).   </li><li> Likely work more with <b>stakeholders</b> in the business (<i>métier</i>).   </li><li> In contrast to the English job description texts, data analysts are expected to know more about <b>SQL</b> (in English this word appeared more frequently in data engineering job descriptions). </li></ul></td>
        <td><ul><li> Expected to have skills in <b>reporting, dashboarding, data analysis</b> and <b>office suite.</b>   </li><li> More interaction with other <b>stakeholders</b> throughout the larger organization.  </li><li> More emphasis on identifying <b>insights</b> (which need to be communicated to others in order to inform decision making). </li></ul></td>
    </tr>
    <tr>
        <td><b>Data scientists</b></td>
        <td><ul><li> Relatively few unique skills.  </li><li> Expected to know <b>data science</b> and <b>statistics</b> (<i>statistique</i>), and to build <b>models</b> (<i>modèle</i>) and make <b>predictions</b> (<i>prédiction</i>). </li></ul></td>
        <td><ul><li> Relatively few unique skills.  </li><li> Expected to know about <b>data science, statistics, mathematics</b> and making <b>predictions</b>. </li></ul></td>
    </tr>
    <tr>
        <td><b>Data engineers</b></td>
        <td><ul><li> Greater expectation to work with <b>cloud platforms</b> (<i>plateforme, cloud, Azure</i>), <b>big data technologies</b> (<b>Scala</b> and <b>Spark</b>), <b>data pipelines</b>, <b>etl</b> and <b>data storage</b> (<i>stockage</i>).  </li><li> Somewhat surprisingly, data engineers, compared to the other roles, are expected to work with <b>agile</b> methodology. </li></ul></td>
        <td><ul><li> Greater expectation to work with <b>cloud</b> and <b>data platforms</b>, <b>etl</b> (<b>data transfer</b> &amp; <b>storage</b>) and <b>data pipelines</b>, <b>databases</b>, <b>data architecture</b> and <b>infrastructure</b>, <b>Spark</b> and <b>SQL</b>.   </li><li> Essentially, the technologies and databases that go along with storing and transferring data from one place to another are under the responsibility of the data engineer. </li></ul></td>
    </tr>
    <tr>
        <td><b>Machine learning engineers</b></td>
        <td><ul><li> Greater expectation to use <b>machine learning</b> (<i>apprentissage automatique</i>), <b>artificial intelligence</b> (<i>intelligence artificielle</i>), and tools for <b>deep learning algorithms</b> / <b>neural networks</b> (<i>réseau de neurones artificiels</i>) like <b>TensorFlow</b> and <b>Pytorch</b>. </li></ul></td>
        <td><ul><li> Greater expectation to use <b>artificial intelligence</b> and <b>deep learning</b> frameworks such as <b>TensorFlow</b> and <b>Pytorch</b>.  </li><li> Greater expectation to know more about <b>software engineering</b> and <b>computer science</b>.  </li><li> Interestingly, the text of the English job ads reveals that machine learning engineers are being asked to work on <b>computer vision</b> problems. </li></ul></td>
    </tr>
</table>
</div>

Some other observations that we found noteworthy:

- There are strikingly few terms that are unique to the data scientist role, suggesting large overlaps with the other profiles. As recently as a couple of years ago, the roles of data engineer and machine learning engineer were much less prevalent and many of the responsibilities  currently assigned to these roles fell under the purview of data scientists. With the growth of other data roles and a resulting divvying up of data work, it seems as though organizations are not entirely clear as to what exactly the unique characteristics of data scientists are. 

- While the conclusions from the wordclouds were virtually identical across languages, there were some notable differences among the different roles between English and French. For example, the French machine learning engineer ads were more likely to include *innovation* than the English ones, perhaps suggesting that this work is taking place in R&D or innovation centers of larger companies. The French job descriptions for data engineers were more likely to mention *agile* methodology, and the French job descriptions for data analysts were more likely to mention *SQL* (in English, this technology was more prevalent for the data engineer job ads). 


- Finally, it was interesting to note that many of the terms used in French job descriptions are actually English words. For example, *cloud*, *reporting*, and *deep learning* could all be translated into French, but they're usually left in English. Other jargon surrounding data professions, however, has well-established French equivalents. For instance, *tableau de bord* is the French equivalent of *dashboard*, *intelligence artificielle* is the French equivalent of *artificial intelligence*, and *apprentissage automatique* is the French equivalent of *machine learning*. So if you're trying to understand the tech industry in France, it's perhaps worth brushing up on your English vocabulary!

## Using Skills-ML to Extract Skills from Job Ads

The [Skills ML library](http://dataatwork.org/skills-ml/){:target="_blank"} is a great tool for extracting high-level skills from job descriptions. The Skills ML library uses a dictionary-based word search approach to scan through text and identify skills from the [ONET](https://en.wikipedia.org/wiki/Occupational_Information_Network){:target="_blank"} [skill ontology](https://www.onetcenter.org/content.html){:target="_blank"}, allowing for the extraction of important high-level skills mapped by labor market experts. This approach is more comprehensive than simply counting words (as we did with the comparison clouds above), and it takes into account the fact that some words are synonyms or represent the same skill or technology (e.g."database", "data warehouse", "data lake", etc. can be grouped under a higher-level term such as "data storage"). Because the ONET skills are only available in English, this analysis was conducted only on the English-language job descriptions. 

### Most Common Skills

As the following figure shows, *Python* was the most common skill represented in the English-language job descriptions. Other top skills include *R*, *programming*, *mathematics*, *Tableau*, *visualization*, *writing*, *Git*, and *physics*. However, this analysis collapses all the skills across the four data roles. We saw in the wordcloud analysis above and in the [previous analysis]{{ site.baseurl }}({% link _posts/2022-01-23-data-jobs-europe.markdown %} ){:target="_blank"} of job keywords that the desired skillsets can look quite different between the different data profiles. 


![Most common ONET skills]({{site.baseurl}}/assets/img/2022-04-18-data-jobs-europe-2-text/top_skills_extracted_onet_better_title.png)


### Clustering Skills and Roles

In order to get a sense of how the extracted skills differed across the data roles, we made a [cluster map](https://seaborn.pydata.org/generated/seaborn.clustermap.html){:target="_blank"} using the Python [Seaborn](https://seaborn.pydata.org/index.html){:target="_blank"} library. Specifically, we calculated the percentage of job ads per role that contained each skill, filtering on skills that appeared in more than 50 job ads. These percentages were converted to z-scores, such that higher numbers indicate that a given skill is mentioned more often for a given role compared to the others. This final matrix was then passed to the cluster map algorithm, which performs a simultaneous clustering of both the job roles and of the extracted skills.

The results of this analysis showed that there are clear clusters of skillsets required for different types of data-related roles.  

![Heatmap ONET skills]({{site.baseurl}}/assets/img/2022-04-18-data-jobs-europe-2-text/heatmap_onet_vlag_better_title.png)


In the clustering diagram, shades of <span style="color:red">red</span> indicate a *higher* prevalence of a given skill for a given role compared to the others, while shades of <span style="color:blue">blue</span> indicate a *lower* prevalence of a given skill for a given role compared to the others. 

Along the horizontal axis, individual skills are clustered together in logical ways. For instance, at the right side of the chart, *Microsoft Office* is grouped together with *Microsoft Excel* and *Google Analytics*. 

On the vertical axis, roles cluster into three separate groups according to their required skills: 

- **Data analysts** are in their own cluster at the top of the graph, with skills that are most different from the other roles. In particular, job ads for data analysts are more likely to mention office-suite software (e.g. *Microsoft Office & Excel, Google Analytics*), data visualization / dashboarding tools (e.g. *Tableau*), and sales management & tracking tools (e.g. *Salesforce*). Job ads for data analysts are less likely to mention programming tools such as *Git, Python, programming languages*, etc.
- **Data engineers** are grouped in an overall cluster with data scientists and machine learning engineers, but have a separate branch from the other two. The job ads for data engineers were comparatively more likely to mention data tools (*Oracle, noSQL, MySQL, MongoDB, PostgreSQL*, and *Apache Spark*). This suggests that data engineers are expected to play greater roles in the development and maintenance of an organization's data infrastructure, compared to the other three roles.
- **Data scientists** and **machine learning engineers** are placed together in the same cluster at the bottom of the graph. These two roles overlap in terms of computer science skills like *programming*, *Python*, and *Git*, and in domain knowledge in scientific fields such as *biology* and *physics*. Furthermore, job ads for these two roles are also less likely to require office suite software capabilities (e.g. *Excel*), visualization (e.g. *Tableau*), and business tools (e.g. *Salesforce*) that are most characteristic of data analysts. However, there are some differences between data scientists and machine learning engineers. Data scientists' job ads have higher prevalence of *mathematics*, *chemistry* and *R*, while job ads for machine learning engineers have higher prevalence of operating systems (e.g. *Unix* and *Linux*), and programming languages (e.g. *Javascript* and *C*).

# The Added Value of Analyzing Job Description Texts

Overall, the above analysis serves as a useful extension of the Metadata analysis we described [in our previous post]{{ site.baseurl }}({% link _posts/2022-01-23-data-jobs-europe.markdown %} ){:target="_blank"}. Here, we first presented comparison clouds showing the relative frequency of words that were unique to a given role compared to the others. We made separate word clouds for the texts of the English and French job ads, respectively, and found that the main conclusions from these visualizations were the same. Interesting findings from this analysis included:

- *Data analysts* are expected to work with dashboarding, data analysis and Office tools like Excel. Of all of the profiles, job descriptions for data analysts were more likely to mention contact with the business, interacting with stakeholders and generating and communicating insights. 

- *Data scientists*, in contrast, had relatively few unique words in their job descriptions. Compared to the other roles, they are expected to know about statistics, mathematics and making predictions from models. Our sense was that, given the recent growth of other data roles such as data engineers and machine learning engineers, there is some degree of ambiguity regarding the distinct characteristics that data scientists should have compared to the other roles.

- The job ads for *data engineers* had a long list of data storage and transfer technologies that were unique to this role. Data engineers are expected to master many different types of databases and cloud platforms in order to move data around and store it in a proper way.

- Finally, job ads for *machine learning engineers* were more likely to contain mentions of artificial intelligence and deep learning frameworks like Pytorch and TensorFlow, applied to domains such as computer vision.

We also extracted skills from the English language job descriptions using the ONET skill classification. As in our previous analysis of skill keywords, Python was the most frequently-appearing skill. We then made a clustermap to see how the extracted skills differed across the roles. 

In this analysis, the data analysts role had least in common with the others. Data analysts in particular were more likely to use office tools (*Excel*, *Google Analytics*), visualization tools (e.g. *Tableau*) and business software (e.g. *Salesforce*), and less likely to use programming tools and languages (e.g. *Git* and *Python*). 

Data Engineers also had their own specialties, being particularly likely to work with a wider variety of data storage, big data, and query technologies (e.g. many flavors of *SQL*, *Apache Spark* etc.) This analysis shows that data analysts and data engineers have very different skillsets, with data analysts being more focused on office and business software, and data engineers being more focused on programming and databases. This highlights the importance of having both roles on a team in order to have a well-rounded skillset, and the unlikeliness of having one person being equally good at both skillsets (the [long-sought after](https://hdsr.mitpress.mit.edu/pub/t37qjoi7/release/4){:target="_blank"} but [rarely-found](https://www.infoworld.com/article/3429185/stop-searching-for-that-data-science-unicorn.html){:target="_blank"} ["unicorn"](https://pubsonline.informs.org/do/10.1287/LYTX.2019.04.02/full/){:target="_blank"} [profile](https://www.forbes.com/sites/cognitiveworld/2019/09/11/the-full-stack-data-scientist-myth-unicorn-or-new-normal/){:target="_blank"}).

# The End

This is the final post that we’ll make of the analysis of these job description data. All of the data and code for these analyses [are available](https://github.com/methodmatters/data-jobs-europe){:target="_blank"} [on Github](https://github.com/methodmatters/data-jobs-europe-2){:target="_blank"}, and we encourage you to explore them further! 

This exercise was very meta for us, challenging ourselves across data analysis, data science, data engineering. Both the [metadata analysis]{{ site.baseurl }}({% link _posts/2022-01-23-data-jobs-europe.markdown %} ){:target="_blank"} presented previously and the current text analysis helped us clarify our thinking about the market for data profiles in Europe, and we hope to have expanded your understanding of the data professions and the skills that unite and differentiate them. The job market is evolving quickly, as are the technologies and tools that data professionals are being asked to master. Our analysis of European job descriptions offers a snapshot of the current job market, and we are excited to see what the future brings as European companies' and institutions' data efforts mature and as the market continues to evolve!
