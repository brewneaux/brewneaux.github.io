---
title: "That's Not CI/CD"
date: 2024-07-16T14:45:52-05:00
tags: []
draft: false
---

I just started a rew role at a new company, and I saw a pattern (that I have seen before) come up again, and I want to talk about it.

<!--more-->

# What does CI/CD even mean?

Probably a lot of different things to a lot of different people. But to me, CI/CD is two very different parts that share the same emphasis. It's automation, all the way down - a perfect CI/CD system would be completely automatic - a developer would check in code, and once someone reviewed it, it could get all the way to stage (the last level before prod) without human intervention.  Obviously, though, this isn't super feasible for most things - you won't get any kind of approval gates or anything. So the balance is to make it automated enough that its crazy easy, and you can move fast, but manual enough that the right approvals happen at the right time.

Breaking it down further - we have two pieces. CI - continuous integration - to me means that you can have completely automated tests that cover as many of your testing layers as possible (at bare minimum, unit tests and component tests) in such an automated way that you can literally _block_ pull requests and the like from getting merged to master without them all passing. The idea here is that code that gets to the main branch will have a REALLY high confidence of working properly - if the tests can be trusted, you know that when you hit "merge", its probably gunna work. This includes not only those tests, but also things like installation and even rollback testing, if thats a thing you need.  

CD - continuous delivery - is getting your software delivered to your environments automatically.  For pre-prod environments, this can just be easily trigger based, especially if you have testing that happens along the way, like if you have integration tests that ensure your service is working within the scope of it being deployed with its friends. In most organizations, though, there is an understandable hesitancy (and sometimes even legal requirements) to make sure that certain gates are being checked as it progresses, especially when it comes to deploying to production. The idea, then, is to make the deployment process - from built artifact to production - as easy and automated as you can. 

# A common anti-pattern I've seen

I have seen a similar pattern show up now, organically, at 3 different organizations I've worked with.

For the delivery of software, a DevOps team (usually) will create a series of repos of Terraform scripts, where each script deploys a single component.  They almost always have a repository-per-environment, with almost exact copies of the same modules, and in each of those, the versions of the deployed software are controlled via the Terraform.  

What this means in practice is that to get to any given environment, someone inherently must do a commit, updating the version, and get it approved via a pull request. I've seen it, also, where approval of these pull requests requires as little as 3, but as many as 5 approvals, from various teams. I have to imagine that those approvers very often do not know what they are looking at, and mostly blindly approve - creating a situation in which the approvals do almost nothing to actually add any benefit.

To me, this is not at all continuous.  If you have to make commits and involve humans to get your software deployed, there isn't a lot that can be automated to get the continuous nature of CD back. 

One of the benefits I see to CD is that it allows developers and teams to _try things_, especially in lower environments, without needing to do the overhead of convincing the powers that be that they are doing the correct things.  A lot of organizations like to enforce some standards for applications - such as run books or documentation or the following of various other practices - but that shouldn't matter for something going to a non-production environment.  

## A side note about ITIL

I don't love ITIL, don't get me wrong, but I do feel that Change Management is a good practice.  It gives a single point of documentation that things have changed within an environment, and allows all necessary parties to be aware of those changes. A good change management process should check all the boxes that are necessary for compliance, create that necessary document trail, but most importantly - not be disruptive.  Even most ITIL or ITSM trainings I've sat in on say that change management shouldn't be a burden or a blocker, but many organizations treat it as one.

Change management is extremely easy to automate, however.  You basically need to keep a changelog in your artifact, and have the various jobs that do the deployments use that change log and the information it knows about the change to just make the CM ticket itself.

# What I've liked

I've implemented automated deployments, including the change management piece, a few times over my career. I usually reach for Jenkins, but the underlying technology really doesn't matter.

Most of the deployments I've done involve Helm charts for Kubernetes, and some type of cloud deployment - usually CloudFormation or Terraform.  What we have found that works best is to abuse the packaged Helm chart and shove our cloud deployment scripts into it.  So for Cloudformation, we make a new folder in the Helm chart, put the Cloudformation scripts in there, and then when we run `helm package`, it includes that folder.

Also included in that package is 0 to a few "deploy hooks".  These hooks are custom logic that will get run at specific stages of the deployment so we can do any custom logic that might need to happen.  For instance, we have used a post-deploy hook to clean up old deployments that we don't even need anymore.

The deployment jobs take in a couple of parameters - one is the version, which can be left as the default to grab the latest semantic versioned package, and the other is a "promote" checkbox.  We store the deployment packages in a repository-per-environment, so in order to get to stage, it _must_ be deployed successfully to dev, with the promote checkbox checked.  This is our "gate" for moving software forward.  

At deploy time, we download that packaged artifact, and un-tar-gz it so we can have access to the various files we need.  If there is a pre-deploy hook, we run that.  Then we run the embedded CloudFormation or Terraform scripts, which are set up to take in some specific arguments, but can take in whatever they want - the per-environment settings are actually packaged within the artifact, and extracted by environment name.  Then we have an optional `pre-helm-hook` that runs before the Helm Install, and then we run the Helm Install, again, taking in a set up standard arguments but optionally including more.  Here, we use values files specific to the environment to control environment specific settings.  At the end, we run a `post-deploy` hook, if it exists, to do whatever else needs to do.

I probably have a lot more words here to say, but for now, I might leave this here.