---
layout: post
title: 'Inside the 2025 Data Labor Market in Europe: <br /> <i> What 8,086 job ads tell us about the roles, technologies and skills that really matter </i>'
date: '2025-09-21T07:00:00.000-01:00'
author: Method Matters
tags:

- Europe
- data
- data jobs
- cluster analysis
- data visualization
- text analysis
- Gemini
- LLMs
- large language models
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
- business analyst
- devops engineer
- cloud engineer
- statistician
- recruiters

---


There are many blog posts and think-pieces about the state of data and the data labor market in the US, but far fewer such writings focused on Europe. The goal of this blog post is to present a data-driven analysis of the job market for data profiles in Europe in 2025, based on an analysis of  **8,086** European job listings posted between April 13 to May 9, 2025. This blog post is based on a talk given by myself and [Cesar Legendre](https://www.linkedin.com/in/cesarlegendre){:target="_blank"} at the [2025 meeting](https://rssb.be/wp-content/uploads/2025/04/RSSB-Statistics-Data-Science-and-AI-2025.pdf){:target="_blank"} of the Royal Statistical Society of Belgium in Brussels. 

The main conclusions of our analysis are:

- **Employers are increasingly looking for “hybrid” profiles** who can take on tasks that are related to multiple job titles (e.g. both data scientist and data engineer)
- **The hot job title of 2025 is “data analyst”**, though employers are also looking for engineering talent across different sub-disciplines (e.g. cloud, develops and data engineers)
- **Data jobs are located in many different data-rich sectors**, including IT, finance, healthcare, consulting and manufacturing
- Five years after the COVID-19 pandemic began, **employers are expecting data employees to return to the office** en masse
- Finally, across the data job profiles, the **top requested *tools* are related to coding, cloud services, and data analysis**, while the **top requested *skills* focus on problem solving, soft skills and project management**

For all of the details and more, read on below!


# Data

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
<table>
  <thead>
    <tr>
      <th>Data set</th>
      <th>8,086 data-related job ads, scraped from a major online job board</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Period</td>
      <td>13 April – 9 May 2025</td>
    </tr>
    <tr>
      <td>Job titles</td>
      <td>Data/ML/AI/Cloud/DevOps Engineers, Analysts (Data & Business), Data Scientists, Statisticians, Financial Analysts</td>
    </tr>
    <tr>
      <td>Key variables</td>
      <td>Job title, country, sector, way of working, tools & skills<sup>1</sup></td>
    </tr>
  </tbody>
</table>

<sup>1</sup> Sector, way of working, tools and skills were extracted based on open-coding of the job description text by LLMs (Gemini from Google).

# Results

## More “Hybrid” Job Titles

One of the first things that popped up to us in this analysis was that fully **11% of job titles encompassed multiple roles**. Some examples (reproduced exactly as listed in the job ads):

- *ML Engineer / Data Scientist*
- *BI Business Analyst / Data Engineer (Business-Intelligence-Consultant)*
- *AI Engineer / Data Analyst*
- *IT Business Analyst / IT Consultant (Data Engineer)*
- *SAP Team Leader SAP BI / Data Science (Data Engineer)*

This trend is a reversal from what we saw in [previous years]{{ site.baseurl}}({% link _posts/2023-09-10-data-jobs-belgium.markdown %} ){:target="_blank"}, where a single job title (e.g. data engineer) would split into separate sub-roles (e.g. devops engineer and cloud engineer) as time went on. In 2025, companies are looking for data profiles who can handle a larger technical scope.

In order to better understand which roles were being sought after in a single hire, we conducted a cluster analysis of the co-occurrence of job titles in our dataset:

**_Figure 1: Clustering of Roles Based on Co-Occurrence in Job Ads_**   
![cluster_role_co_occurrence]({{site.baseurl}}/assets/img/2025-09-21-data-jobs-europe-2025-part-1/cluster_role_co_occurrence.jpg)

This analysis shows that the job titles fall into three primary groups: 

- The first (on the left-hand side) is **business analyst**, which appears in a cluster by itself. This makes sense, in that business analyst is a broad and relatively generalist job title, covering dashboarding as well as other types of general business analysis.
- We next have a cluster consisting of **IT engineering profiles**: *devops* and *cloud engineer*. These profiles both work to deploy data solutions and move data around within an organization.
- Finally, we have the third group (on the right-hand side) whose **members all work with data** in some capacity. The data engineer job title is more on the *storage and transfer* side, and it appears in its own sub-cluster. The remaining titles (in orange in the graph above) relate to *analysis* or *model-building* in some capacity.

## Most Popular Data Job Titles in 2025

The plot below shows the most frequently occurring job-titles in our jobs dataset:

**_Figure 2: Most Popular Job Titles_**
![top_job_titles_250]({{site.baseurl}}/assets/img/2025-09-21-data-jobs-europe-2025-part-1/top_job_titles_250.png)

We see three main takeaways from this analysis 

- **The hottest data job title in 2025 in Europe is business analyst**; these profiles work on dashboards, business intelligence, and other general business topics linked with data. It makes sense that such a generalist title would be most frequent in the data.
- Second, **the engineering titles** (cloud, devops & data engineers) **are very sought after right now.** These three titles are more-or-less equally mentioned, and together account for 53% of all job titles in our data.
- **Data scientist is much less popular than in previous years**. Specifically, in our [2021 analysis]{{ site.baseurl}}({% link _posts/2022-01-23-data-jobs-europe.markdown %} ){:target="_blank"}, there were more openings for data scientists than for data engineers. In our [2023 analysis](https://methodmatters.github.io/data-jobs-belgium/){:target="_blank"}, there were around 2 data engineer job ads for every data scientist ad. In 2025, there are 3 data engineer job ads ford every data scientist ad, indicating the continued relative decline of data science in comparison to data engineering.

<!-- 

(https://methodmatters.github.io/data-jobs-europe/)
{{ site.baseurl}}({% link _posts/2022-01-23-data-jobs-europe.markdown %} )

(https://methodmatters.github.io/data-jobs-belgium/)
{{ site.baseurl}}({% link _posts/2023-09-10-data-jobs-belgium.markdown %} )

 -->
## Sectors


Our analysis indicates that companies in data-rich sectors are the most interested in hiring data profiles. Indeed, in these domains, the potential added value from data and data analysis is substantial:


**_Figure 3: Number of Data Jobs Per Company Sector_**
![company_sector_250]({{site.baseurl}}/assets/img/2025-09-21-data-jobs-europe-2025-part-1/company_sector_250.png)

- **IT and cybersecurity** rank number one. Looking more closely at these job descriptions, we find jobs that sit in the IT silos of organizations. These employees are expected to work almost exclusively with IT tools, databases, SQL, cloud technologies.
- The second most popular sector is **financial services**. Here, we see many banks and insurance companies represented, and the jobs are a mix between more generalist profiles (e.g. data scientist / engineer) and specialist profiles such as financial analyst.
- **Healthcare & life sciences** comes next. In this group of organizations we find medtech companies working with data-intensive machines like EEG tools, along with hospitals, national cancer research centers, and SAAS companies targeting the healthcare industry.
- There is healthy demand, as in [previous]{{ site.baseurl}}({% link _posts/2022-01-23-data-jobs-europe.markdown %} ){:target="_blank"} [years]{{ site.baseurl}}({% link _posts/2023-09-10-data-jobs-belgium.markdown %} ){:target="_blank"}, in the **consulting** sectors. These job ads are all focused on finding people to go perform data work at various (nearly always unnamed) clients.
- Finally, we see a fair mount of jobs in the **manufacturing** sector. A great many of these positions focus on working with data from industrial processes in factories, whether in storing, analyzing and visualizing such data or on optimizing industrial processes based on data captured in factories.

It is safe to say that, in 2025, a great many sectors are looking for data profiles, and that people with data skills are sought after in many different types of organizations!

## Way of Working

One of the many ways that the COVID 19 pandemic changed our lives was in the shift to remote and hybrid ways of working. 

In 2025, however, it seems pretty clear that companies in Europe are expecting data workers to return to the office. This is a big change from a [similar analysis in 2023]{{ site.baseurl}}({% link _posts/2023-09-10-data-jobs-belgium.markdown %} ){:target="_blank"}. In 2023, there were more hybrid jobs advertised than there were onsite jobs. In the current data the trend has reversed itself, fairly heavily, in the opposite direction.

**_Figure 4: Number of Data Jobs Per “Way of Working”_**
![way_of_working_250]({{site.baseurl}}/assets/img/2025-09-21-data-jobs-europe-2025-part-1/way_of_working_250.png)

## Private vs. Public Organizations

The graph below shows the split in job advertisements between private and public organizations. 

It is clear that the vast majority of data jobs in our analysis are in the private sector. However, there are a non-trivial amount of jobs in the public sector as well. In our data, public entities looking for data profiles include public transport companies (e.g. national rail and regional transportation networks), national research centers, finance ministries and tax services, along with national, regional and city governments. 

**_Figure 5: Number of Data Jobs in the Private vs. Public Sectors_**
![public_private_sector_250]({{site.baseurl}}/assets/img/2025-09-21-data-jobs-europe-2025-part-1/public_private_sector_250.png)

# Top Tools & Skills

Now let’s dig in a bit more to the substance of the job advertisements, in particular into what qualities prospective employers are looking for. For this analysis, we relied on LLMs to examine the text of each job description and to extract the tools and skills that were expected of job applicants.

## Tools

The graph below shows the top requested tools. The following themes strike us as interesting: 

- **Coding** (Python, SQL, etc.) and **development** (e.g. VSCode) **tools** are the number one requested toolset in this analysis. Despite the fact that LLMs are making some coding tasks easier, data professionals are still expected to know how to code!
- In contrast to [previous]{{ site.baseurl}}({% link _posts/2022-01-23-data-jobs-europe.markdown %} ){:target="_blank"} [years](https://methodmatters.github.io/data-jobs-belgium/){:target="_blank"}, **cloud tooling** and topics (e.g. containerization, Azure, Cloud computing, AWS, GCP, etc.) occurs in many places on our list. This increased focus is new, and indicates that the cloud is increasingly where companies work with data and deploy data solutions.
- Tools related to **data analysis** also make the list, whether on the BI side, or tools used for more complicated analysis like data science, statistics, and modeling.

**_Figure 6: Top Tools for Data Jobs_**
![top_tools_250]({{site.baseurl}}/assets/img/2025-09-21-data-jobs-europe-2025-part-1/top_tools_250.png)

## Skills

The graph below shows the top-requested skills in our job ad data. The following points stick out to us: 

- The number one skill in this analysis is **problem solving** and **analytical thinking**. At a high-level, working with data entails a never-ending series of problem solving and analytical thinking exercises. It makes sense that these skills take the top spot!
- The second and third most-requested skills here are **soft skills** – namely *collaboration* and *teamwork* and *communication* and *interpersonal* *skills*. This represents a big change from [previous]{{ site.baseurl}}({% link _posts/2022-01-23-data-jobs-europe.markdown %} ){:target="_blank"} [years]{{ site.baseurl}}({% link _posts/2023-09-10-data-jobs-belgium.markdown %} ){:target="_blank"}, where technical skills were ranked much more highly than soft skills. There seems to be an increasing realization that, in the data professions, being able to communicate to others what you’ve done and why you’ve done it is critical.
- Finally, **project management** seems to be a sought-after skill – having the ability to “get things done” is important, not always easy in an organizational setting, and not necessarily intuitive for many technical profiles.

**_Figure 7: Top Skills for Data Jobs_**
![top_skills_250]({{site.baseurl}}/assets/img/2025-09-21-data-jobs-europe-2025-part-1/top_skills_250.png)

One final thing we’ll take a look at in this blog post is to examine whether some of the job titles place more or less accent on the different skills in our analysis

## Clustering Job Titles & Skills

In order to understand how the job titles (e.g. data analyst, scientist, data analyst, etc.) differ by the requested skills, we made the below clustermap. In this map, *lighter* colors indicate *higher* prevalence of a given skill for a given job title compared to the others, while *darker* colors indicate a *lower* prevalence of a given tool for a given job title compared to the others.

The graph below gives an overview of the differences in employer expectations in regards to skills for the different data job titles. There are too many data points to go over them all in detail, but the following results struck us as the most interesting: 

- **Business analysts** need the most *communication* and *interpersonal skills*, likely because they often work more closely with the business and therefore have a greater need to explain their work to non-experts.
- **AI engineers** are expected to be constantly learning, no doubt because this sub-domain moves so rapidly that knowledge quickly becomes out-of-date.
- **DevOps** need to be good at *operations* and *support* to others – their job is to keep processes running, and when the processes fail to get them back up again.
- **Financial analysts** need *specific domain knowledge* more than other data profiles. Indeed, the financial data space is one where domain knowledge is critical, given the regulation surrounding specific types of modelling applications (e.g. credit risk scoring) in the banking sector.
- Finally, **data analysts** are asked to have a *problem-solving* and *analytical thinking mindset*, compared to the other roles.
   
**_Figure 8: Clustering of Skills and Roles_**    
![heatmap_skills_roles_250]({{site.baseurl}}/assets/img/2025-09-21-data-jobs-europe-2025-part-1/heatmap_skills_roles_250_20250921.jpg) 
    

# A Data Career in Europe in 2025

So, what does this analysis suggest about how to think about a data career in 2025 in Europe? In our thinking, there are two primary types of work that data professionals could consider pursuing:

### Focus on Moving the Data:

- Job titles include **data** and **cloud** **engineering**, along with **devops**
- These titles dominate the data jobs ads in our analysis
- *This expertise is needed because* A) data need to be in good shape to be used effectively, B) the task is enormous (often due to past under-investment) and C) few companies are where they need to be to take advantage of the data they have.

### Focus on Understanding the Data:

- Job titles include **business analyst** / **data analyst** / **data scientist**
- There is strong demand for these job titles in our analysis
- *This expertise is needed because* A) understanding data is pre-requisite to act on it B) the ability to analyze, visualize and communicate clearly about data is a rare and valuable talent and C) other engineering and IT profiles are less good at this (in our experience)

# Summary and Conclusion

Our main takeaways from our analysis of 8,086 European data job ads are as follows:

- **Employers are increasingly looking for “hybrid” profiles** who can take on a broader set of responsibilities (e.g. handle both data engineering and data science work)
- The **bulk of the data job titles** can be **clustered into two main groups** with different emphases:
    - *Data analysis* in some shape or form (notably data analyst, along with data scientist, statistician, etc.)
    - *Moving data* from one place to another (e.g. engineering in some capacity, notably cloud, develops and data engineers)
- **Data jobs are to be found in many different sectors**, and present in both private and public organizations. In short, data opportunities exist in a great many places!
- Across the data profiles, **employers are looking for candidates who are familiar with *tools* related to coding, cloud services, and data analysis**
- Across the data profiles, **employers seek candidates who have problem-solving and project management skills, and who balance their technical know-how with soft skills**

### *Coming Up Next*

How has the perception and practice of the field of data science evolved over the past decade? An [early and influential article](https://hbr.org/2012/10/data-scientist-the-sexiest-job-of-the-21st-century){:target="_blank"} in the Harvard Business Review from 2012 made a great deal of promises and predictions about what the field of data science would deliver. The next blog post will discuss how those predictions have held up over the past 13 years. *Stay tuned!*