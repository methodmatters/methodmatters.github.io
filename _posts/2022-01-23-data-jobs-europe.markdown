---
layout: post
title: 'The European data labor market: Where are the data jobs and what are companies looking for when hiring Data Scientists, Data Engineers, Machine Learning Engineers and Data Analysts?'
date: '2022-01-23Z12:00:00.000'
author: Method Matters
tags: 
- Europe
- data
- data jobs
- cluster analysis
- data visualization
- Python
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

<div style="width:1000px;">
</div>

# Introduction

It’s 2022 and the market for data talent in Europe is definitely hot. Along with newer digitally-native companies who have been focused on data since day one, an increasing number of established organizations are launching digital and data initiatives, with all prospective employers attempting to attract candidates from a relatively small (but growing) talent pool. As any data professional can attest, there are plenty of open jobs and unsolicited contacts from recruiters arrive regularly.

Despite there being plenty of jobs available, data-related job advertisements often appear unrefined. Many are laundry lists of technologies, programming languages, or software tools combined in sometimes surprising ways (e.g. “The ideal candidate will master both Tableau and Tensorflow”), while others talk in vague generalities about what the role will entail (e.g. “You will develop custom data models and algorithms”). 

To better understand the European data job market, the intrepid [Jesse Blum](https://www.linkedin.com/in/jblum1/){:target="_blank"} and I scraped 5 months of relevant ads from 7 European countries on a major online job site for the following roles: *Data Scientist*, *Data Engineer*, *Data Analyst*, and *Machine Learning Engineer*. Our goal was twofold: 
1. To understand the relative demand for the different profiles  
1. To understand what types of skills employers are looking for in data professionals, and how these skills might differ across different roles. 

The results were very interesting -- with 6,590 job ads scraped, we were able to see that there is a comparatively low demand for Machine Learning Engineers compared to the other profiles; the cities with the most data job openings are London, Paris, and Berlin; the sectors with the most open roles are Business Services and IT; the companies doing the most hiring are either relatively small or very large; and the most in-demand skill is Python (unless you're a data analyst). Furthermore, each data role has a distinct profile of unique skills and as we suspected, other qualities such as empathy and good communication skills are sorely missing from data job advertisements.

# The Data

The data come from a scraping program developed by Jesse and myself. Every 2 weeks, we scraped job advertisements from a major job portal website. We extracted all jobs posted within the previous 2-week period for the following job titles: Data Engineer, Data Analyst, Data Scientist and Machine Learning Engineer for the following countries: the United Kingdom, Ireland, Germany, France, the Netherlands, Belgium and Luxembourg. We started data collection in mid-August and finished at the end of December 2021.

You can find the data and scripts for this analysis on Github [here](https://github.com/methodmatters/data-jobs-europe){:target="_blank"}. Comments, suggestions and improvements are welcome!

# What Roles are Most In Demand and Where Are They Located?

*Data scientist* was the most-popular job title in our dataset, followed closely by *data engineer* and *data analyst*. *Machine learning engineer*, in contrast, is a niche role right now in the European countries we examined.

![role frequency]({{site.baseurl}}/assets/img/2022-01-23-data-jobs-europe/job_title_freq_250.png)

## Cities

The city with the most jobs was *London*, followed closely by *Paris*. The next three most popular cities for data jobs were *Berlin*, *Dublin* and *Amsterdam*. Interestingly, “Remote” was sometimes listed explicitly as the city of employment, but only for 105 jobs. Many other jobs offered varying degrees of flexibility in terms of location, but it appears that Europe has not embraced remote work at this point. Indeed, [European regulations](https://www.linkedin.com/pulse/dissecting-eu-rules-social-security-remote-working-daida-hadzic/){:target="_blank"} seem to make it difficult for an employee to live in one country but work in a different one (even within the EU).


![top cities]({{site.baseurl}}/assets/img/2022-01-23-data-jobs-europe/jobs_by_city_250.png) 



## Role Breakdown for Top Cities

We wondered whether specific cities had roles that were more concentrated for specific job titles. We therefore broke out the number of jobs separately for each of the four data roles for the top 7 cities.

![role frequency top cities]({{site.baseurl}}/assets/img/2022-01-23-data-jobs-europe/roles_by_cities.png) 


Not all cities share the same demand profile. *London* and *Dublin* had more openings for data scientists and data analysts, while *Paris* and *Berlin* had comparatively more openings for data engineers. *Brussels* (where Jesse & I work) has more-or-less equal numbers of roles for data scientists, engineers, and analysts. Finally, *Berlin* stands out for the relatively large number of open positions for machine learning engineers (essentially the same amount of demand as for data analysts and scientists).

# What Type of Companies Are Recruiting?

## Company Sector 

When looking at the sectors that were most common across the dataset, *Business Services* and *Information Technology* had by far the most number of job openings.[^1]  


![company sector]({{site.baseurl}}/assets/img/2022-01-23-data-jobs-europe/jobs_per_sector_250.png) 


The roles in the *Business Services* sector appear split between two different types of companies. On the one hand, there are traditional business consultancy companies looking to hire data consultants. On the other hand, there are recruitment companies who are seeking candidates for third-party companies (some of which might also be consultancies!). 

For the *Information Technology* sector, roles appear to mostly be for smaller tech companies (often software-as-a-service). These companies are often focused on a specific application and industry, such as health care, marketing, automotive, etc.

## Company Size

Demand for data talent was particularly strong among two different types of companies - smaller ones with 500 employees or fewer, and very large ones with more than 10,000 employees.[^2]   

![company size]({{site.baseurl}}/assets/img/2022-01-23-data-jobs-europe/jobs_per_company_size_250.png) 


Smaller companies appear to be mixes of small consultancy firms and tech start-ups. Larger employers include banks, insurance companies, government agencies, state-run postal services, pharmaceutical companies, chemical companies and major retailers.

# Desirable Qualities in Candidates

## Overall 


Skill keywords were embedded in the scraped job descriptions. These were chosen by the employers from a predetermined list of skills (in English) maintained by the job portal website. These keywords represent the skills that the employers chose as being important for candidates to possess. 87% of the job advertisements contained at least one skill keyword, and there were 341 separate skills listed in total across all job postings. These are the 25 most-frequently listed skills across all job roles:

![top skills]({{site.baseurl}}/assets/img/2022-01-23-data-jobs-europe/top_skills.png) 


The most frequently listed skill is *Python*, present in more than half of the job advertisements. Python’s presence outranked its nearest competitor, *SQL*, at a rate of about 3:2. Beyond programming languages, among the other top-mentioned skill types are *language proficiency* (e.g. English, German, Dutch, French), proficiency with *visualization tools* (e.g. Power BI, Tableau), *big data technologies* (e.g. Spark, Hadoop), *cloud technologies* (e.g. AWS), and *statistical analysis software* (e.g. R, SAS). 

While this analysis gives a sense of the skills common among all of the data roles, it masks differences that are present among them. 

## By Role

The cluster map below shows how the roles (e.g. data scientist, data analyst, etc.) differ by their associated skills (e.g. software development, language skills, cloud platforms). We first grouped together separate skills that reflected a common underlying dimension (e.g. French, English, Dutch, etc. were all grouped together under the title “Language Skills”). We then calculated the percentage of job ads per role that contained each skill, filtering on skills that appeared in more than 50 job ads. These percentages were converted to z-scores per skill, such that higher numbers indicate that a given skill is mentioned more often for a given role compared to the others. This final matrix was then passed to the cluster map algorithm, which performs a simultaneous clustering of both the job roles and of the requested skills.[^3]


![roles & skills cluster map]({{site.baseurl}}/assets/img/2022-01-23-data-jobs-europe/roles_skills_clustermap.png) 


In this map, *lighter colors* indicate *higher* prevalence of a given skill for a given role compared to the others, while *darker colors* indicate a *lower* prevalence of a given skill for a given role compared to the others. 

Individual skills cluster together in logical ways - e.g. at the left side of the chart, Python is grouped with software development, C, natural language processing and Tensorflow; while on the right side of the plot, SQL is linked with Oracle and Databases. 

Furthermore, we also see how the roles cluster into two separate groups according to their required skills. 

*Data analysts* and *data scientists* are placed together in the same cluster at the top of the graph. These two roles overlap in terms of relational databases, SAS, communication skills and presentation skills. 

*Data engineers* and *machine learning engineers* are placed together in the same cluster at the bottom of the graph. These two roles overlap in terms of Kubernetes, shell scripting, APIs, Linux, Jira, Docker, Go, and Java. 

The cluster map also indicates which skills are particularly descriptive of certain roles but not others. 

- **Data analysts**, in contrast to the other roles, are expected to know about Microsoft Office, Google Ads, and Business Intelligence (including Power BI and Tableau, two Business Intelligence dashboarding tools). They are also expected to know relatively more about data analysis (logical given the role title, but presumably data scientists and machine learning engineers should also be skilled at data analysis), IT (oddly enough), and marketing topics.

- **Data scientists**, in contrast to the other roles, are expected to know more about MATLAB, leadership, data mining, and R.

- **Data engineers**, in contrast to the other roles, are expected to know more about Kafka and Terraform, Scala, Ansible and node.JS, Redshift, Big data, HTML5, Elasticsearch, MongoDB, NoSQL, .NET, Hadoop, Apache, Spark, and Cloud Platforms (i.e. AWS, Azure, GCP). 

- **Machine learning engineers**, in contrast to the other roles, are expected to know more about software development, C, natural language processing, and TensorFlow. 

## What’s Missing?

With the rare exception, the above skills are nearly all technical or software related. While the technology side of the data profession is foundational, in our view focusing on technology to the exclusion of other characteristics is a common but ultimately ineffective and even counterproductive way to choose which data professional to hire.

We are [not the first](https://analyticshour.io/2021/10/05/177-design-thinking-empathy-and-the-analyst-with-hilary-parker/){:target="_blank"} [to emphasize](https://jnolis.com/blog/hiring_1/){:target="_blank"} [the importance of](https://www.kdnuggets.com/2021/07/5-lessons-mckinsey-taught-better-data-scientist.html){:target="_blank"} [these skills](https://www.oreilly.com/library/view/the-care-and/9781492053972/){:target="_blank"}, but we felt that we couldn’t talk about job descriptions for data profiles without highlighting two skills which we think are particularly important: empathy and communication skills.

### Empathy

[Hilary Parker](https://hilaryparker.com/){:target="_blank"} has argued for the value of empathy in data and software development work. Her argument is that it is hard to deliver a good data solution if you do not understand your stakeholders. This requires empathy, by which we mean being able to understand other peoples’ points of view, needs and problems. A succinct articulation of this point comes from the classic book the [Pragmatic Programmer](https://en.wikipedia.org/wiki/The_Pragmatic_Programmer){:target="_blank"}, originally written for software developers but which contains many pearls of wisdom that are applicable to the data professions: “Our job is to help people understand what they want.” 

In our experience, empathy is critically important to all four roles because very often stakeholders in the data space don’t know exactly what they want, and have a hard time expressing their needs, prioritizing them, and interpreting results. Data initiatives are new in many companies and therefore most business stakeholders will not have much experience with data analytics, data science, or data engineering. Furthermore, data initiatives are often fundamentally threatening to the status quo. While existing leaders and managers within organizations can provide broad direction (though success on this front varies wildly), they often lack the knowledge necessary to define successful data projects. This can lead to friction when data professionals and organizational stakeholders need to initiate projects and collaborate to deliver business value. Empathy on the part of the data professional (along with a minimum amount of goodwill and collaborative spirit on the part of the stakeholder) can go a long way towards bridging this gap. 

## What’s Under-Valued?

### Communication Skills

Data topics are complex by their very nature, and this is true across all of the roles examined here. It is almost guaranteed that most business stakeholders will not understand all of the details of the work performed by the data professional. Nevertheless, stakeholders need to accept and use solutions developed by data professionals, and they will have high-level questions about dashboarding solutions, statistical or machine-learning models, and data pipelines that are built by data people. Having the ability to explain at a high level to many different types of stakeholders (executives, business partners, IT colleagues, other data professionals) what one does (whether dashboards, statistical/machine-learning models, or data pipelines), is important. Advocating and convincing others to take a sensible technical approach to achieving the desired business outcome is even more so.

We have experienced the difficulties surrounding communication about data topics in multiple ways: interactions with business stakeholders when defining projects or processes, interactions with other data professionals when working together across roles (e.g. data analysts, scientists, and engineers), and even when working with other colleagues who share our exact same expertise and job title.[^4] While it is unreasonable to expect every data professional to be a master communicator, excluding communication skills from the job requirements and selection criteria is asking for trouble in the long-run.

We are encouraged that, in our data, there are signs that companies are recognizing the importance of communication skills. After all, communication skills are listed as a keyword in more than 600 of the job descriptions in our dataset. However, that’s only one-fifth of the number of Python requests. The cluster map revealed that communication skills seem to be most prevalent in job ads for data analysts and data scientists, while we would argue that communication is important in all four of the data roles we examined.[^5]

# Summary & Conclusion

## Main Takeaways

Our analysis of more than 6,500 job ads shines a light on the data labor market in seven European countries and shows that:

- There are far fewer openings for machine learning engineers as compared to data scientists, data engineers, and data analysts.
- In London and Dublin, demand is higher for data scientists and data analysts vs. the other data roles, while in Paris and Berlin demand is higher for data engineers.
- Python is the most desired skill, appearing in more than half of the job descriptions.

Employers appear to have a specific vision on skills needed for each of the data roles: 

- Data analysts are expected to use Microsoft Office, Google Ads and marketing, and business intelligence tools.
- Data scientists are expected to use statistical software (MATLAB & R) and data mining tools.
- Data engineers are expected to use data pipelining technologies and cloud platforms.
- Machine learning engineers are expected apply software engineering strategies to natural language processing and deep-learning frameworks.

Finally, we noticed that job advertisements are overly focused on specific tools. This is akin to hiring managers looking for quality plumbers by asking for skills with a particular brand of wrench. This narrow focus has the potential to exclude the candidacies of well-rounded people from consideration for data jobs. 

## Where to Go From Here?

Despite the many [barriers to implementing data initiatives](https://www.bruegel.org/2021/11/what-is-holding-back-artificial-intelligence-adoption-in-europe/){:target="_blank"} (including the scarcity of talent), many European organizations are scaling up their data efforts and their recruitment of data profiles. We are encouraged by the increasing maturity we see in the data labor market, and while the recruitment process is often chaotic for both candidates and employers, we have the subjective impression that it’s getting easier for job seekers to find roles and for companies to find attractive candidates.[^6] Our analysis here suggests that employers are perhaps overly focused on technologies and tools, and one concrete recommendation we can make is for hiring managers to not neglect softer skills (like empathy and communciation) and to seek out well-rounded candidates. For additional thoughts on recruitment of data profiles, we direct interested readers to [this mature take](https://www.oreilly.com/library/view/the-care-and/9781492053972/){:target="_blank"} on recruiting data scientists and [this interesting blog post](https://www.getdbt.com/data-teams/hiring-data-engineer/){:target="_blank"} on hiring data engineers.


# Coming Up Next Time

In the next post, we’ll return to these data and do a deep dive into the text of the job advertisements to understand more precisely what companies are looking for when recruiting data analysts, data scientists, data engineers and machine learning engineers. 

Keep an eye out for that post in the coming weeks!


---

[^1]: 2/3rds of the job descriptions contained information on the company sector.

[^2]: 76% of the job descriptions contained information on the company size.

[^3]: This is conceptually similar to a [cluster analysis of musical genres]{{ site.baseurl }}({% link _posts/2020-11-23-music-song-key-mode-genre-clustering.markdown %} ){:target="_blank"} I did a while back. Big ups to the [Seaborn package](https://seaborn.pydata.org/generated/seaborn.clustermap.html){:target="_blank"} for making this analysis so easy. 

[^4]: It’s sometimes surprisingly hard for data scientists to communicate with other data scientists!  

[^5]: [Some have advocated](https://www.mckinsey.com/business-functions/mckinsey-analytics/our-insights/analytics-translator){:target="_blank"} for a specialized bridging role (e.g. “Analytics Translator”) to mediate technical data professionals and business stakeholders. The Analytics Translator “translates” the technical matter to the business stakeholder, and “translates” the business requirements to the data professional. This is a nice idea in theory, [but in practice](https://www.kdnuggets.com/2021/07/5-lessons-mckinsey-taught-better-data-scientist.html){:target="_blank"} can lead to a game of “telephone” where much information is lost when being passed from person-to-person.

[^6]: Our experience is personal in that we both changed jobs since starting this project!