---
title: "Git Workflows"
date: 2023-12-14T09:36:59-06:00
tags: []
draft: false
---

Git Workflows are so highly talked about on the internet - I would guess there are tens of thousands of blog posts, articles, and discussion threads on the topic.  Everyone seems to have their own opinions on why your workflow is slowing you down or why its problematic - but no one talks about the fact that, like most tools, it needs to work for _you_ and _your team_.  I feel the same way about agile and scrum - so many people like to tell you how to do it, and like to tell you that your way is bad, but that way might not work for you at all.

<!--more-->

# Use what works for you

Beyond anything else, use the Git workflow that works for you.  Trying to use someone elses when it doesn't fit your situation reminds me of trying to use a screwdriver that is too small - eventually it will work, but annoyingly.  Finding what works for you is going to take time, and its going to probably lead you to annoyance in the process, but it'll be worth it in the end when it feels like second nature.

Let's take an example of this blog.  I'm the only person who ever uses it, and the only one who ever commits.  I don't need pull requests and complicated branching strategies because the chances of me overwriting something that important are extremely slim.  My workflow for this blog is basically to just check out a feature branch when I make a new post, and use that branch until I'm done.  Even that is probably overkill, however. The blog is built on [Hugo](https://gohugo.io/), and I use a combination of the VSCode Markdown Preview and `hugo --buildDrafts server` to preview it.  I leave all of my posts in `draft: true` until I'm ready to go with them.  The chances of me "releasing to production" something that isn't read is super super slim, especially because "release to production" is just a commit to master that invokes a Github Workflow.

So for this blog, using any kind of complicated workflow is overkill and would just slow me down.  On the other hand, I do use some of the features of Git that let me work on multiple things and on multiple devices - I have branches for new posts before they are ready, so I can leave notes, I commit often to make sure I can get back to things. However, I also merge by hand straight to master, and will (not infrequently) commit directly to master when I'm working on small layout tweaks or the like.  That would never work with a team of people.

# What works for me and my team

For the vast majority of our team's work, we use what basically comes out to be trunk-based development.  All of our work is done in feature branches, and they get merged to master through required PRs.  Our repositories are set up so that basically only GitHub itself can commit to master.  We work on Microservices, and the way we release to production means that we can pretty much always fail forward and just release a bugfix.  It works great for our team, and takes only the smallest amount of cognitive overhead to think about.  Personally, I really like it because we don't need to remember how different releases work, we don't have to think about merging to a specified pre-release branch, and we don't have to keep track of multiple versions of a codebase in Git.  There is one HEAD, there will always be one HEAD, and we can remember where it is.


In previous posts, I've talked about how there I have a counterpart on my team that I work really closely (and really well) with.  We end up pretty often working on the same app, on the same overall feature, and coming about it from multiple sides - or, in the Component Testing posts, I was writing tests while he coded functionality.  


For that, we need a different slightly different approach.  My component tests contained cases that the original code didn't pass, and his new feature development didn't have tests in the original code.  

# Enter the megatron branch

We started using a thing we called "megatron" branches - these are branches for a specific feature that you work from as your "fake master" until you are done.  So for our application rewrite, we created a megatron branch of `megatron/refactor`.  That became the master that he and I both worked from.  Off of that, I then created a `feature/component-test-framework`, and started my development.  We did frequent, small Pull Requests from our feature branches to megatron, which meant we were reviewing things and finding potential bugs the whole way, without having to do a 200-file, multiple-thousand-line pull request at the end.  Overall, we probably did 12-15 pull requests from our feature branches to the megatron, and at the end, we were reasonably sure it would all "just work".  

When we went to merge megatron to master, we still did a pull request, and both he and I looked it over. This was more to make sure nothing looked overwritten, and that we didn't forget something glaring.  We did not do a line-by-line full on code review at this stage, because we trusted that your previous feature-to-megatron PRs were well reviewed.  

------------

I think, the moral of the story, at least for me, is don't let people on the internet (including me) tell you how you work, and more importantly, tell you that you are working wrong.  Finding what works for you is always going to be better - especially with something like Git, where the less you can think about it the better it does its job for you.
