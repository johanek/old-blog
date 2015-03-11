---
layout: post
title: "The path to peer review"
date: 2015-03-10 14:58
comments: true
categories:
---

As a sysadmin, I've never been part of a team that did peer review well. When you're doing a piece of work and you want to check you're doing the right things, how do you get feedback?

Do you send an email out saying *"can you look at this?"*

Do you have someone look at the screen of your work?

Do you have discussions about what you’re going to do?

Sometimes I've done these things, and they've been partly effective - but usually they just get a reply of *"Looks good to me!"*

Most of the sysadmins I've worked with will just make a change because they want to, or were asked to, and don’t tell anyone. Plenty of them want to go away and work on something and then deploy it and for everyone to use it and tell them they’re awesome.

I don’t want to work where people are expected to pull rabbits out of hats. I hate surprises. Having done all those individualistic stupid things myself, I want to work in a team where we work together on problems out in the open. When you involve others, they are more engaged and feel part of the decision making process. And their feedback makes you produce better work.

My current role now has the best culture of reviewing work that I've seen. But we had to create it for ourselves, and this is what we did.

## We put all our work in version control

When I started this role, the first project I worked on was to move our configs into git. Most of those configs are stored on shared NFS volumes which are available on all hosts. Previously people made config changes by making a copy of the file they wanted to change and then made the change to the copy. Once that is ready they would take a backup of the production copy of the file, and copy their new version into place.

Importing the files into repos was generally straightforward, but sometimes there was automatically generated content that needed excluding in `.gitignore`. To deploy the repo we added a post-update hook on the git server that would run `git pull` from the correct network path.

But we also wanted to catch when people would change things outside of git and do so without overwriting their changes. This would allow us to identify who was changing things and make sure they knew the new git based process. To do this we added a pre-receive hook on the server that would run `git status` in the destination path and look for changes.

That was the first step in changing the way we worked. It wasn't a big change, but it got everyone somewhat comfortable with using git. The synchronous nature of the deployment was a big plus too, because all the feedback about success or failure would happen in your shell after running `git push`

## Notification workflows

This was pretty awesome, and we got more and more of our configs into this system over time. The next thing we wanted to do was to produce notifications about what's changing so we could catch mistakes, or find bad changes that broke things, and just understanding who is making changes to what systems.

We did two things to achieve this - we made all these repos send email changelogs and diffs whenever someone pushed a change, and we created an IM bot that would publish details changes into a group chat root.

This was great to produce an feed for the audit trail of changes happening to the system that we knew about. But whenever you saw a breaking change, you thought - well I knew that was going to break something, I could stop that before it was deployed.

## Then we added Pull Requests

We knew we wanted to implement Pull Requests but we couldn't do this with the git server software we were originally using.

Since we had started using Confluence and JIRA for our intranet and issue tracking, we moved all our git repos to Stash, which is Atlassian's github like git server. This provided us with the functionality we wanted.

On a single repo, we enabled a Pull Request workflow - no one could commit to master any more and everyone had to create a new branch and raise a PR for the changes they wanted merged.

We wrote up some documentation on how to use git branching and raise PRs via the stash web interface, and explained to everyone how it all worked.

We chose a repo that was relatively new, so the workflow was part of the process of learning to work with the new software. It was also one that everyone in the team would have to work with, so that everyone was exposed to the workflow as soon as possible.

To deploy the approved changes, we used the Jenkins plugin for Stash to notify Jenkins when there were changes to the repository. Jenkins then ran a job that did the same thing as our previous post-update hooks - ran `git pull` in the correct network location.

Running the deployment asynchronously like this felt like we were losing something - if there was a problem you were sent an email & IM from Jenkins, but this felt less urgent than a bunch of red text in your terminal. But for the benefit of review, this was a price worth paying.

In the interim we had moved the checks for changes being made outside of git into our alerting system, so we could catch them earlier than when the next person went to make a change. This meant we didn't have to implement these checks as part of the git workflow - if there was still a problem we let the hook job fail, and it could be re-run from Jenkins once the cause was resolved.

Over time, we moved all the other repositories across to this workflow, starting with the repos with the highest number of risky changes. But, for a number of repos, we kept the old workflow with synchronous deployment hooks because all the changes being made were low risk and well established practises.

## Initially, slowing down is hard to do

The hardest thing to adapt to was that we were changing the perceived pace that people were working. Everyone was very used to being able to make the change they wanted immediately, and close that issue in JIRA straight away, and that's how they judged how long a piece of work takes to do. But that doesn't take into account the time spent troubleshooting and doing unplanned work.

What we were doing was moving more of the work up front before deployment, where you can fix problems with less disruption. But making that adjustment to the way you work can be really hard because you perceive the process to be slower.

Everyone in our team is a good sport and willing to give things a go - but as much as we could try to explain the benefit of slowing down, it takes time actually doing it to see how much better it is.

Sometimes it's necessary to move faster, and we manage that simply - if someone needs a review now, they ask someone and explain why it's urgent. Similarly if I'm busy I'll say something like *"I'm probably not going to be able to review everything in my queue today. Is there anything that needs looking at now?"*

## In time the benefits are demonstrated

By using Pull Requests we created an asynchronous feedback system. In JIRA, you propose what you're going to do. Then you implement it how you think it should be done and create a PR. Then when a reviewer is available they'll provide feedback or approve the change. You keep making updates to your PR and JIRA until the change is approved or declined.

Given enough time, everyone experienced all of the following benefits of that feedback:

**Mistakes are caught before they are deployed**

This was what we set out to do! There were breaking changes made before, and there are less of them now.

**Learn about dependencies between components**

A common type of feedback is *"How is this change going to affect X?"* - sometimes the requestor has already considered that and can explain the impact and any steps they took to deal with X. But if you haven't considered it, you need to research it. Then you've learnt more about how things are connected and have a greater appreciation of how the system works as a whole.

**Consistent style and approaches**

Everyone has their own preferred way of doing things - with a PR we can say this is the style we want to use, and enforce it. The tidier you keep your configs and code, the more respect others will have for them.

It's massively helpful to get told there's an existing function that achieves what you're trying to do, or there's existing examples of approaches to solving a problem. This can help you learn better techniques, and avoid duplicating code.

**Identify risky changes**

With changes to fragile systems, you're never confident about hitting deploy even if the change looks good. Until it's been put in production and suffered the full range of uses there is risk. So this has allowed us to schedule deploying changes for times where the impact will be lower, or deploy the change to a subset of users.

It's also stopped push and run scenarios - we avoid merges after 5pm, they can always wait for tomorrow morning!

**Forced to explain what you’re trying to do**

Much of the review process is not about identifying problems with configs and code, but simply being aware of and understanding the changes that are taking place. This in invaluable to me as a reviewer.

So, when raising a Pull Request, it's expected that each commit has a reference to the JIRA issue related to the change. The Pull Request can have a comment about what is changing and any, and that can be very helpful but the explation of why the change is taking place must be in the JIRA.

This way, when looking back at the change history we can also reference the motivations for making that change and see the bigger picture beyond the commit message.

## People want to have their work reviewed

Somewhere along the way, getting your work reviewed became desirable. You realise the earlier you put your work out there for review, the earlier you get feedback and the less likely you are to spend time doing the wrong thing.

We still have a number of repos that anyone can just change by pushing to master, there's no controls because these changes are considered safe. But people *choose* to create branches and PRs for their changes to these repos, because they *want* to have their work reviewed!

## Change takes time ...

Going from no version control to making this cultural change in the way we approached our work took about 2.5 years.

Throughout this period there was no grand plan that we were heading to a certain scenario. At each stage we could only see a small way forward, and had some things we were looking to do better, so we experimented and tried things. Then, for the things that worked, we made sure everyone kept doing it that way.

At the start, I had no idea that peer review of sysadmin work would be done via code review. That didn't feel intuitive to the way we were working in the past. But as we've moved more things into git and we've seen the benefits, we want to manage everything the same way which drives moving more things into git.

We moved slowly because there was other work going on and we needed to get people comfortable with the new way of working and tooling before we could ask more of them. It takes time to change, and many of our team have benefitted from seeing that incremental process take place.

But it's been gratifying that the newest team members who have joined since we established PRs have said *"I was uncertain about it at first, but now I get it. It really works."*

## ... and involves lots of hard work

To get to where we are has meant spending a lot of time setting up new repositories and isolating files that are managed by humans and automatically generated, migrating repositories to stash, creating deploy hooks, explaining how git works, and most of making sure you're providing useful reviews and that they happen regularly.

It's one thing to start setting up a couple of repos like this - but to fully establish the change you need to do lots of boring work where you make sure *everything* is migrated, even the more difficult cases. It's really important you're using a single system for everything!
