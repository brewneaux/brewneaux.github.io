---
title: "Component Testing"
date: 2023-11-18T11:08:29-06:00
tags: []
draft: false
description: |
    How and why to component test everything.
    >>>
    
---

At work, I have a cohort, whom with I am convinced I share a brain.  We recently embarked on a project to redesign and rewrite one of the core pieces of the application stack that we work one - it manages a hands-free modelling pipeline, based on configurations.  A user of our app can make a configuration change, and this component reconciles those changes, runs the necessary background modelling tasks, and figures out when the model is ready to be used in production.

At its core, it is a state machine.  We designed it like that at the start. However, through time and added features, it wasn't implemented as a state machine, and became pretty unruly to debug or extend.  We were adding a lot of new features in our upcoming release that just couldn't be made to work in the existing code base - it was written too rigidly to accept that many things can happen from a singular "state", each taking their own time - and started a rewrite into a full state machine.  We're a C# and .NET shop, so we found [dotnet-state-machine/stateless](https://github.com/dotnet-state-machine/stateless), and it worked amazingly. 

While we were doing the rewrite, however, we wanted to make sure we were not adding new bugs or regressions.  The original codebase had only the bare minimum of automated tests, so we started with expanding them first.

Let's talk about testing layers, how they interact, and how we used them to ensure our rewrite was a success
<!--more-->

# Testing Layers

Software Testing is a notoriously difficult thing, and I think most engineers have struggled with it at some point - either technically, or cognitively.  We know it is necessary, to prove that what we have coded actually _works_, but it rarely is fun, and it is often tedious.  Making sure your tests cover everything you need to is an even bigger challenge - coverage metrics exist, but do not actually ensure that you are truly testing a particular line of code.

The best way I have found is to use a software testing layer approach.  You never ensure that any particular layer will 100% validate every edge case, but you cover as much as you can, expanding on the layers below.  I primarily work on non-user-facing software, so I have the layers defined like this - the names I use might be non-standard, but its how I like to think about them.  

### Unit Testing

Cover all of your functions.  Make sure you assert every single code path.  To make unit testing easier, make sure each function does _one_ thing, and does it well.  When writing unit tests, I tend to do a few cases for the more “normal” uses, and then add a few that give inputs that are wildly unexpected.  I spend about as much reading the code (that I potentially just wrote) as I do writing up the unit tests themselves.  

### Component Testing

A component test should (generally) be written without looking at the code, and instead thinking about it from the business use cases of the application.  For example, you aren’t testing specific functions within your app when you do a component test for an API endpoint, but you are instead verifying that (nearly) every possible way someone might use that endpoint works how you expect it.  

Some purists say that you should _only_ use the application features that exist to write component tests - that is, if you are component testing a CRUD app, you should _only_ use the read endpoints to ensure that the create or update endpoints work.  I tend not to agree.  The inputs should only be through the real ways data can get into your app (REST, message busses, etc), but you can - and in some places, should - assert data pulled straight from the data storage.  We tend to use DynamoDb for a lot of our backend storage, and most of our component test suites end up with rudimentary data access layers.  

Usually, for component tests, we tend to use mock versions of external dependencies.  For AWS, we tend to use the amazing https://github.com/getmoto/moto library.  For other APIs, we have adopted [MockServer](https://www.mock-server.com/).  Using non-real infrastructure makes it significantly easier to write these component tests - I tend to have moto and mockserver running in my local Docker almost constantly, and it allows me to just boot an app and get to testing.

 

### Functional and Integration Testing

This is where you make sure that multiple services all work together.  This doesn’t have to just be microservices - most, if not many, apps in the modern era rely on __something__ externally, either APIs or infrastructure.  These tests start the app like it would be in real life, and you write tests that will ensure that all the pieces work together.  In the same way as component tests, you might need to assert based on data in the data stores, not just application outputs.


## This is more of a parfait than an onion

Shrek jokes aside, these testing layers, combined with manual QA validation, are all meant to work together, not separately. As you move up the pyramid, each one should build on the last.  Unit tests make sure the code pieces all work.  Component tests make sure those pieces work as a whole. Functional/Integration testing makes sure the application works in the real world around it.  QA is there to make sure the application works when we get weird use-cases.
