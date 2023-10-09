---
layout: post
title:  GRC 2 - Left or Right Brain?
date:   2023-07-19 15:01:35 +0300
image:  03.jpg
tags:   WebDesign
---
### Project Scope

GRC 2(Governance, Risk and Compliance) is the 1st and the only risk management tool in Amazon, with all 1,541,000 employees as its potential users.

GRC 2's existing users include Global Financial Risk Controls (GFRC) team, policy creators and owners across Amazon, CDO Infosec, AWS Security, AWS Managed Services, Internal Audit and risk teams. 

Amazon leadership (VP and directors) also rely heavily on GRC 2 to do periodicly reviews.

My Role: the only designer and lead researcher of the project.



### Problem Statement

##### What does GRC do?

GRC is a structured tool to align IT with business goals while managing risks and meeting all industry and government compliance requirements.

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

### Design: Commit to the hard part - Risk Library

At this point, it's hard to ignore the elephant in the room: risk.

Risk is the reason why controls exist, why controls need periodic reviews, and lives in all the processes and applications. 
However, no one has successfully built any risk library in Amazon since it's founded. 

Surprise!

##### Why risk library is super hard to build 

To successfully run a risk library, we will need:
* The permission to 10+ risk themes (confidential to every company)
* A standardized risk metrics that can be shared across company and applied to any risk. (info security, natural disaster, financial risk, etc.)
* A risk management team who audits all the risks description and how they are connected to   

### Develop: Heads in Clouds, Feet on the Ground

You, a bobsleder!? That I'd like to see! No! The kind with looting and maybe starting a few fires! Good news, everyone! There's a report on TV with some very bad news! When I was first asked to make a film about my nephew, Hubert Farnsworth, I thought "Why should I?" Then later, Leela made the film. But if I did make it, you can bet there would have been more topless women on motorcycles. Roll film!

Eeeee! Now say "nuclear wessels"! Why did you bring us here? Yeah, and if you were the pope they'd be all, "Straighten your pope hat." And "Put on your good vestments." That's the ONLY thing about being a slave.

### Wins

The biggest challenge for GRC has always been finding the simple solution/output 