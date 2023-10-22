---
layout: post
title:  GRC 2 - Left or Right Brain?
date:   2023-07-19 15:01:35 +0300
image:  03.jpg
tags:   WebDesign
---
### Project Scope

GRC 2(Governance, Risk and Compliance) is the 1st and the only risk management tool (+link) in Amazon, with all 1,541,000 employees as its potential users.

GRC 2 is currently supporting Global Financial Risk Controls (GFRC) team, policy creators and owners across Amazon, CDO Infosec, AWS Security, AWS Managed Services, Internal Audit and risk teams. 

Amazon leadership (VP and directors) also rely heavily on GRC 2 to do periodicly reviews.

My Role: lead designer and researcher of the project.

### Context

This storyboard can help you understand the context and terminology.
![]({{ site.baseurl }}/images/grcstoryboard.jpeg)
*Terminology*

Business entity: restaurant

Process & application: cooking

Risk: fire

Control: fire alarm, emergency exit

Compliance: government standards

Review: Periodically check
### Problem Statement

##### What does GRC do?

GRC is a structured tool to align IT with business goals while managing <strong>risks</strong> and meeting all industry and government <strong>compliance</strong> requirements.

(Risk: natural disaster, info leak, manual errors, or any risks in the world.

Compliance: the act of following rules, laws, and regulations. It applies to legal and regulatory requirements set by industrial bodies and also for internal corporate policies.)


##### What is the user need?

Over 10 years, Amazonians have been using GRC 1.0 which looks like this:
![]({{ site.baseurl }}/images/OldGRC.jpg)
*Click to enlarge*

The biggest problems here include but not limited to:
<ul>
<li>No visibility on the aggregated status of users' personal items (processes/applications/controls/reviews/...) in GRC.</li>
<li>No risk library, which is the reason why controls exists. </li>
<li>UI is tech-heavy, and not friendly to non-tech users (including leadership). </li>
</ul>

Long story short, GRC 1.0 is nothing more than a recording tool of users' personal items, and cannot provide extra insights/convenience.


### Research: No one knows it all.

The scope of GRC is extremely broad, which covers both internal and external risks and auditing flows. Not too surprisingly, we soon found out: no one knows it all-most users only interact with a small part of GRC, and no teammate (PM or SDE) has the comprehensive understanding of all the user flows. Based on this finding, we decided to divide and conquer. Our plan pivoted to: start with one customer-GFRC(Global Financial Risk Controls) team on their financial auditing flow, and then spread the success and scope to other users.

After:
* 30 hours of interviews
* 10+ interviewees
* 4 weeks of research
* 2 Rounds of workshops with customers 

We were able to map out 4 user journeys for main personas:

![]({{ site.baseurl }}/images/ownerflow.jpg)

![]({{ site.baseurl }}/images/approverflow.jpg)

![]({{ site.baseurl }}/images/auditorflow.jpg)

![]({{ site.baseurl }}/images/creatorflow.jpg)

Based on these flow, we were able to move to the design phase.


### Design: Enterprise app is boring...but it doesn't have to be.

With the user flows in mind, it's fairly easy to map out the IA map...but, wait.

Before heading into the rational work and narrowing down the scope, I wanted to get everyone in the room (teammates and customers) excited, so I collected competitors in the same field and company to collect stakeholders' comments on what they like vs. not.

![]({{ site.baseurl }}/images/GRCMoodboard.jpg)
*Moodboard*

This practice allowed our stakeholders to think with their creative and emotional minds, and treat the app users as human on top of personas.
The wish list we collected include:
* Customers want a clear conclusion ("Am I at risk or not?") at the first glance.
* Customers like clean color palette (but not too serious).
* Timeline view will help the leadership and admins, on the visibility of their teams and scopes.

This wish list helped me quickly map out the IA and wireframes:

 <div class="row">
  <div class="column left"><img src="{{ site.baseurl }}/images/GRCIAmap.jpg" alt="IA Map"></div>
  <div class="column right"><img src="{{ site.baseurl }}/images/Dashboardwire.jpg" alt="Dashboard Wireframe"></div>
</div> 

Since stakeholders were deeply involved in the co-creating process, they were able to understand the IA map and wireframes easily, and had the confidence that their needs are well heard and received.

### Design: Commit to the hardest part - Risk Library

At this point, it's hard to ignore the elephant in the room: risk.

Risk is the reason why controls exist, why controls need periodic reviews, and lives in all the processes and applications. 
However, no one has successfully built any risk library in Amazon since it's founded. 

Surprise!

##### Why risk library is super hard to build 

After another 2 rounds of research workshops with risk managers, we found out the required conditions to successfully run a workshop are:
* The access to 10+ risk themes (confidential to every company).
* A standardized risk metrics that can be shared across company and applied to any risk (info security, natural disaster, financial risk, you name it.).
* Regular updates to all the risk themes and sub-categories (3 layers, hundreds of categories in total).
* A risk management team who audits all the risk descriptions and how they are connected to risk themes.

Even after the library is built, users will need the following to make it useful:
* Knowledge of all the risk themes and sub-categories to connect their own risks to;
* knowledge of risk metrics, and the objective judgement on risk ratings;
* connecting all the related processes/applications/controls to a risk.

##### How can GRC 2 benefit from a risk library?

Despite the challenges we would face while building a risk library, the library will become a game-changer to GRC once built.  
I visualized the benefit with the concept design as following:

![]({{ site.baseurl }}/images/GRCdashboard.jpg)

![]({{ site.baseurl }}/images/RiskList.jpg)

As shown above, GRC 2 users will be able to:
* get an aggregated view on risk score/review status, instead of having to track down single risks/reviews. 
* easily map and categorize all their risks, and deal with them based on priority level and ratings.
* find all the controls related to risks, which could help remediate them. 
* find out all the processes and apps risks live in, and flag them based on risk level.

This conclusion and visualized concept motivated our stakeholders to expand the business scope to the building a risk library, and adding it to the GRC network.

### Develop: Heads in Clouds, Feet on the Ground



### Wins

The biggest challenge for GRC has always been finding the simple solution/output 