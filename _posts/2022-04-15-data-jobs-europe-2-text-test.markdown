---
layout: post
title: 'Test Post'
date: '2022-04-15Z12:00:00.000'
author: Method Matters
tags: 


---

<div style="width:1000px;">
</div>

# Introduction

In the previous post [LINK], my partner for this project, [Jesse Blum](https://www.linkedin.com/in/jblum1/){:target="_blank"} and I analyzed metadata from over 6,500 job descriptions for data roles in seven European countries. In this post, we've applied text analysis to those job postings to better understand the technologies and skills that employers are looking for in data scientists, data engineers, data analysts, and machine learning engineers. 

## Executive Summary 

Performing text analysis improved our overall understanding of data from more than 6500 job openings for data roles in seven European countries, compared with only considering job ad metadata. In this post we present results from text analyses that show that: 
- Data analysts are expected to have skills in reporting, dashboarding, data analysis, and office suite software. 
- Data scientists are expected to know more about data science, statistics, mathematics, and making predictions. 
- Data engineers are expected to industrialize organisations' cloud and data architecture and infrastructure. 
- Machine learning engineers are expected to use artificial intelligence and deep learning frameworks such as TensorFlow and Pytorch.

The results of this analysis complements the results we presented last time, showing that employers have distinct visions of the (mostly technical & software-related) skillsets that data analysts, data scientists, data engineers, and machine learning engineers should possess. 

# The Data

The data come from a web scraping program developed by Jesse and myself. Every 2 weeks, we scraped job advertisements from a major job portal website, extracting all jobs posted within the previous 2-week period for the following job titles: Data Engineer, Data Analyst, Data Scientist and Machine Learning Engineer for the following countries: the United Kingdom, Ireland, Germany, France, the Netherlands, Belgium and Luxembourg. We started data collection in mid-August and finished at the end of December 2021.

We started data collection mid-August and finished by the end of December, 2021, ending up with 6,590 job descriptions scraped. All the data and code used to for this analysis are available on [Github](<Link>). Feedback welcome! 

# Results

Our dataset includes job descriptions for data roles across four languages (English, French, Dutch and German). We wanted to see if there were any differences in word usage among the different roles (data scientist, data engineer, machine learning engineer and data analyst), and therefore conducted language-specific analyses to contrast and compare the roles according to the words used to describe the job openings.

## Word Clouds

Our first set of analyses uses a great [R function](<link to CRAN or other package docs>) to create comparison clouds. This type of analysis allows us to compare the frequency of words across groups of documents, and highlight words that appear more in a given group versus the others. 

Jesse and I are more comfortable in English, French, and Dutch than German, so we limited our analysis to those three languages. However, there were far fewer Dutch job descriptions than for the other two, so its resulting comparison cloud was not particularly informative. Below, we focus on the English and French wordclouds and what they reveal about employers' expectations for the different roles.


PICTURE HERE





Overall, we found that there were clear differences between the roles in the language used in the job advertisements. Furthermore, these differences were consistent across the English and French language job ads. 

The following table summarizes the comparison:

| **Role**                   | **French (N = 1349) Job Descriptions**                                                                                                                                                                                                                                                                                                                                                                                                 | **English (N = 3869) Job Descriptions**                                                                                                                                                                                                                                                                                                                                                   |
|:--------------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| Data analysts              | <ul><li>foo</li></ul> | <ul><li> - Expected to have skills in reporting, dashboarding, data analysis and office suite  </li><li> - More interaction with other stakeholders throughout the larger organization </li><li> - More emphasis on identifying insights (which need to be communicated to others in order to inform decision making). </li></ul>                                                         |
| Data scientists            | <ul><li> - Relatively few unique skills </li><li> - Expected to know data science and statistics (statistique), and to build models (modèle) and make predictions (prédiction). </li></ul>                                                                                                                                                                                                                                             | <ul><li> - Relatively few unique skills </li><li> - Expected to know about data science, statistics, mathematics and making predictions. </li></ul>                                                                                                                                                                                                                                       |
| Data engineers             | <ul><li> - Greater expectation to work with cloud platforms (plateforme, cloud, Azure), big data technologies (Scala and Spark), data pipelines, etl and data storage (stockage) </li><li> - Somewhat surprisingly, data engineers, compared to the other roles, are expected to work with agile methodology. </li></ul>                                                                                                               | <ul><li> - Greater expectation to work with cloud and data platforms, etl (data transfer & storage) and data pipelines, databases, data architecture and infrastructure, Spark and SQL.  </li><li> - Essentially, the technologies and databases that go along with storing and transferring data from one place to another are under the responsibility of the data engineer. </li></ul> |
| Machine learning engineers | <ul><li> - Greater expectation to use machine learning (apprentissage automatique), artificial intelligence (intelligence artificielle), and tools for deep learning algorithms / neural networks (réseau de neurones artificiels) like TensorFlow and Pytorch. </li></ul>                                                                                                                                                             | <ul><li> - Greater expectation to use artificial intelligence and deep learning frameworks such as TensorFlow and Pytorch </li><li> - Greater expectation to know more about software engineering and computer science </li><li> - Interestingly, the text of the English job ads reveals that machine learning engineers are being asked to work on computer vision problems. </li></ul> |
   |

************

**Role**|**French (N = 1349) Job Descriptions**|**English (N = 3869) Job Descriptions**
:-----:|:-----:|:-----:
Data analysts|<ul><li> - Expected to know about data analysis (analyse), reporting (reporting, tableau de bord), and data visualization (visualisation).  </li><li> - Likely more work with stakeholders in the business (métier).  </li><li> - In contrast to the English job description texts, data analysts are expected to know more about SQL (in English this word appeared more frequently in data engineering job descriptions). </li></ul>|<ul><li> - Expected to have skills in reporting, dashboarding, data analysis and office suite  </li><li> - More interaction with other stakeholders throughout the larger organization </li><li> - More emphasis on identifying insights (which need to be communicated to others in order to inform decision making). </li></ul>
Data scientists|<ul><li> - Relatively few unique skills </li><li> - Expected to know data science and statistics (statistique), and to build models (modèle) and make predictions (prédiction). </li></ul>|<ul><li> - Relatively few unique skills </li><li> - Expected to know about data science, statistics, mathematics and making predictions. </li></ul>
Data engineers|<ul><li> - Greater expectation to work with cloud platforms (plateforme, cloud, Azure), big data technologies (Scala and Spark), data pipelines, etl and data storage (stockage) </li><li> - Somewhat surprisingly, data engineers, compared to the other roles, are expected to work with agile methodology. </li></ul>|<ul><li> - Greater expectation to work with cloud and data platforms, etl (data transfer & storage) and data pipelines, databases, data architecture and infrastructure, Spark and SQL.  </li><li> - Essentially, the technologies and databases that go along with storing and transferring data from one place to another are under the responsibility of the data engineer. </li></ul>
Machine learning engineers|<ul><li> - Greater expectation to use machine learning (apprentissage automatique), artificial intelligence (intelligence artificielle), and tools for deep learning algorithms / neural networks (réseau de neurones artificiels) like TensorFlow and Pytorch. </li></ul>|<ul><li> - Greater expectation to use artificial intelligence and deep learning frameworks such as TensorFlow and Pytorch </li><li> - Greater expectation to know more about software engineering and computer science </li><li> - Interestingly, the text of the English job ads reveals that machine learning engineers are being asked to work on computer vision problems. </li></ul>

### to do
Strikingly few terms that are unique to the data scientist role, suggesting large overlaps with the other profiles, and potentially hinting at some confusion as to what exactly the unique characteristics of data scientists are. 

However, there were some interesting differences among the different roles. For example, the French machine learning engineer ads were more likely to include “innovation”, and their data science ads were more likely to include “model” (modèle). On the other hand, English data engineer ads were more likely to highlight databases, and English data analyst ads had more emphasis on “stakeholder”.