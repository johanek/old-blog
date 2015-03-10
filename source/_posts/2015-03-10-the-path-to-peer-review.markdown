---
layout: post
title: "The path to peer review"
date: 2015-03-10 14:58
comments: true
categories: 
---

# The Path to Peer Review

How do you get feedback on your work? Send an email out saying ‘can you look at this?’ Have someone look at the screen of your work? Have discussions about what you’re going to do?

Sometimes this happens and sometimes this is effective. I see people who just do something because they want to, or were asked to, and don’t tell anyone. I see people who want to go away and work on something and then deploy it and for everyone to use it and tell them they’re awesome

I don’t want to work somewhere people are pulling rabbits out of hats. I hate surprises. Having done all those individualistic stupid things myself, I want to work in a team where we work together on problems out in the open because when others are involved they are more engaged and you produce better work.

## We put all our work in version control

*something something shared filesystems*

The first project I worked on here was to start moving our configs into git. Previously people made config changes to production by making a copy of the file they wanted to change and then made the change to the copy. Once that is ready they would take a backup of the production copy of the file, and copy their new version into place.

Importing the files into repos was straightforward, then to deploy the repo we added a post-update hook on the git server that would run `git pull` from the correct network path.

But we also wanted to catch when people would change things outside of git without overwriting their changes, and this allowed us to identify who was changing things and make sure they knew the new git based process. To do this we added a pre-receive hook on the server that would run `git status` in the destination path and look for changes.

That was the first step in changing the way we worked - from before modifying a file in production and saving it, to modifying your local copy, committing and pushing your changes. This didn't change the way people were working massively, but it got everyone somewhat comfortable with using git. The synchronous nature of the deployment was a big plus too, because all the feedback about success or failure would happen in your shell after running `git push`

## Notification workflows

This was pretty awesome, and we got more and more of our configs into this system over time. The next thing we wanted to do with the setup is to produce notifications about what's changing so we could catch mistakes, or find bad changes that broke things, and just understanding who is making changes to what systems.

We did two things to achieve this - we made all these repos send email changelogs and diffs whenever someone pushed a change, and we created an IM bot that would publish details of commits and URLs to view diffs.

This was great to produce an feed for the audit trail of changes happening to the system that we knew about. But whenever you saw a breaking change, you thought - well I knew that was going to break something, I could stop that before it was deployed.


## Then we added Pull Requests 

We knew we wanted to implement Pull Requests but we couldn't do this with the git server software we were originally using.

Since we had started using Confluence and JIRA for our intranet and issue tracking, we moved all our git repos to Stash, which is Atlassian's github like git server. This provided us with the functionality we wanted.

On a single repo, we enabled a Pull Request workflow - no one could commit to master any more (except the reviewers) and everyone had to create a new branch and raise a PR for the changes they wanted merged. 

We wrote up some documentation on how to use git branching and raise PRs via the stash web interface, and explained to everyone how it all worked.

We chose a repo that was relatively new, so the workflow was part of the process of learning to work with the new software. It was also one that everyone in the team would have to work with, so that everyone was exposed to the workflow as soon as possible.

To deploy the approved changes, we used the Jenkins plugin for Stash to notify Jenkins when there were changes to the repository. Jenkins then ran a job that did the same as our previous post-update hooks - ran `git pull` in the correct network location.

Running the deployment asynchronously like this felt like we were losing something - if there was a problem you were sent an email by Jenkins and IMed a link to the build log, but this felt less urgent than a bunch of red text in your terminal. But for the benefit of review, this was a price worth paying.

In the interim we had moved the checks for changes being made outside of git into our alerting system, so we could catch them earlier than when the next person went to make a change. This meant we didn't have to implement these checks as part of the git workflow - if there was still a problem we let the hook job fail, and it could be re-run from Jenkins once the cause was resolved.

Something something moved more repos something kept synchronous deploys for the low risk stuff.


## Initially, slowing down is hard to do
The hardest thing to adapt to was that we were changing the pace that people were working. Everyone was very used to being able to make the change they wanted immediately, and close that issue in JIRA straight away. What we were doing was slowing down the rate of change deliberately, but making that adjustment to the way you work can be really hard. Everyone in the team is a good sport and willing to give things a go - but as much as we could try to explain the benefit of slowing down, it takes time actually doing it to see how much better it is.


Sometimes it's necessary to move faster though, and we manage that simply - if someone needs a review now, they ask someone and explain why it's urgent. Similarly if I'm busy I'll say something like *"I'm probably not going to be able to review everything in my queue today. Is there anything that needs looking at now?"*


## In time the benefits are demonstrated

Catch mistakes before they happen
More consistency in style and approaches
Can identify risky changes and think about how we stage the deployment
Forced to explain what you’re trying to do to someone else

## People want to get their work reviewed

Somewhere along the way, it became accepted t... people choose to... 

When you find something being managed manually, and it's not in git, and it's different for each instance... You wish it was being done properly. And it becomes a priority to fix those things.

The result of that is you see things like - we still have a number of repos that anyone can just change by pushing to master, there's no controls because these changes aren't causing outages. But people *choose* to create branches and PRs for their changes to these repos, because they *want* to have their work reviewed!

It's been gratifying to see that the newest team members who have joined since we established PRs have said *"I was uncertain about it at first, but now I get it. It really works."*


## Change takes time and involves lots of hard work

In all this there was no grand plan that we were heading to a certain scenario. At each stage we could only see a small way forward, and had some things we were looking to do better, so we experimented and tried things. Then, for the things that worked, we made sure everyone was doing it that way.

At the start, I had no idea that peer review of sysadmin work would be done via code review. That didn't feel intuitive to the way we were working in the past. But as we've moved more things into git, ... got the benefit, want to manage everything that way which drives moving more things into git.

We moved slowly because there was other work going on and we needed to get people comfortable with the new way of working and tooling before we could ask more of them.


## The process enjoys many of the same benefits as change control, without the bullshit

ITIL defines the change management process this way:

*The goal of the change management process is to ensure that standardized methods and procedures are used for efficient and prompt handling of all changes, in order to minimize the impact of change-related incidents upon service quality, and consequently improve the day-to-day operations of the organization.*

ISO 20000 defines the objective of change management (part 1, 9.2) as:

*To ensure all changes are assessed, approved, implemented and reviewed in a controlled manner.*