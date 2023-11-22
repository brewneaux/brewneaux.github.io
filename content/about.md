+++
title = 'About'
date = 2023-11-21T17:13:17-06:00
draft = false
+++

My name is Jon Bruno.  I am a senior software engineer, acting as an architect for a contact center SaaS company.  We work in C# (ASP.NET primarily) and Python, orchestrating our deployments with Helm and Kubernetes and Jenkins, and running our infrastructure in AWS with a combination of CloudFormation and Troposphere.

I very much just fell into the software world.  While I was attending Columbia College Chicago for a BA in Photojournalism, I had a job working as a call center representative for an online retailer.  Over time, I moved from just being a phone rep to what they called a Credit Specialist, where I was responsible for approving orders based on both their fraud likelyhood and their credit worthiness.  At the time, we were just using some human brain heuristics to determine fraud, and I got fairly decent at it - to the point that I became known as "the fraud guy" within the company.  I started to focus more on finding larger fraud trends to stop them more effectively, and was doing so using prebuilt reports, that quickly became not good enough for what we wanted to do.


## My start

I received limited access to our MySQL Data Warehouse, and taught myself SQL to be able to more effectively identify these trends.  After a year or two of that, and asking for new reports to be made out of fully formed SQL queries that I was handing our Analytics Team, I was poached to join the team.  There, I helped with a big migration from some custom built tooling to run ETL jobs to running jobs written in Perl - chosen only because it was already installed with the tools we needed, and we weren't allowed to install any other languages.

While on the Analytics Team, I was tasked with rewriting a core product - one that found the links between orders by things like partial address matches, email addresses, phone numbers, etc.  Originally written as a convoluted series of shell scripts and a C program doing a depth-first search, that ran once a week and took well over 36 hours to run.  It just wasn't fast enough.  So it was re-written in PHP (which I taught myself, with the guidance of a great mentor), using MySQL as a graph database to be able to do the same matching progress as an order is placed, rather than as an offline, once-a-week process.  

## Am I an engineer now?

Eventually, I moved to a new position at a different company, as a Data Engineer in a business intelligence team.  They were using MSSQL and SSIS to run all of their ETLs for a multi-tenant environment, with really limited monitoring and almost no flexibility - any new features were added just by nesting stored procedures inside stored procedures, which meant that no one knew how anything worked anymore.  I embarked on a project to write us a new ETL framework in Python (at the time, all of the now-standard tooling like Airflow didn't run on Windows).  It allowed us to run a stored procedure with any number of either manually or automatically specified Stored Procedure arguments, or external applications, all guided by configuration.  We built it all out so that the application, and all of the SQL it may have needed (table definitions, stored procedures, static data) were deployed via Chocolatey, and soon had fully-automated deployments on a weekly basis - a process that previously took a developer all day to accomplish now was the click of a button.

From there, very similar to my last position, I also rebuilt a large-scale ETL.  This time, it was a cross-tenant aggregation process operating on something like 150 million records, performing complex aggregations on about 20 columns.  It was much too large to run daily on our MSSQL warehouse, so it was rewritten in an Apache Spark job, made of 4 distinct tasks, orchestrated by Apache Airflow.  

## Crap, we got acquired

We did, and my role changed a little bit.  With the acquisition came a small change in what our larger R&D team was focused on, and with that, most of the data engineering work I had been doing was put into maintenance mode.  Now, the focus became working on our next generation of some of our software stacks.  I fell more and more into a C#/.NET/ASP.NET role, and quickly became a lead engineer for those things.  In the last 4 years as that role, I have gotten extremely comfortable with the stack and learned to love it.  

For the last 2 years, I have been acting as an architect for our product, as well as a tech lead. 