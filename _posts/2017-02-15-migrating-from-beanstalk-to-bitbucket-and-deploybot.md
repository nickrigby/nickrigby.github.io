---
layout: post
title: "Migrating from Beanstalk to Bitbucket (and Deploybot)"
date: 2017-02-15 20:30:00 -0500
categories: general
tags: [bitbucket, beanstalk, deploybot, deployments] 
---

I recently migrated a ton of repositories from [Beanstalk](http://beanstalkapp.com) to [Bitbucket](https://bitbucket.org) (with [Deploybot](https://deploybot.com) for deployments), so I wanted to share my thoughts on the process.

## Beanstalk
First of all, [Beanstalk](http://beanstalkapp.com) is awesome. I've been using it for a little over seven years, and it has served me well over that time. It has an easy to use interface, code reviews, and above all the ability to deploy code — the feature that drew me to the product originally. However, as the number of repositories I managed grew, it didn't make sense to keep increasing my plan just for deployments. I started to migrate old repositories to [Bitbucket](https://bitbucket.org), but that created a problem. When you're working with a team of developers, managing repositories in different places becomes cumbersome. There had to be a better way, and fortunately there was.

## Bitbucket and Deploybot
[Bitbucket](https://bitbucket.org) is great. For a product that can cost as little as zero dollars for unlimited private repositories, it really doesn't get much better. Throw wikis, issues and pipelines into the mix, and you start to realize how much of an unbelievable offering it is. The interface is simple and intuitive, and I love the wiki feature (which Benastalk doesn't have), which comes in really handy when you are collaborating on a project. As far as managing remote repositories, it's as good as Beanstalk, actually even a little better (Sorry Beanstalk!).

_Note:_ Bitbucket has an import feature, which makes it a breeze to migrate repositories from other remotes.

### Deployments
Bitbucket doesn't have a simple deployment interface like Beanstalk. As far as I can tell deployments are possible via pipelines, but the pricing seems a little vague, and set up looks a little more work than I'd like. I really wanted a GUI interface similar to Beanstalk, which is where Deploybot comes in.

### Deploybot
[Deploybot](https://deploybot.com) is purely for deployments, and allows you to connect repositories from Bitbucket, Github and other third party sources. It's also from the same company that makes Beanstalk, so the interface is very similar.

Setting up deployments is very similar to Beanstalk, but is has even more features, which makes it even more useful. Here's some features I like, which Beanstalk doesn't have (at time of writing):

#### Server API connections
If you're using AWS or Digital Ocean, you can use API keys to authenticate with your servers. Can save you time if you set up deployments to the same server often e.g. in the case of a staging server.

#### Atomic Deployments
Atomic deployments allow you to deploy with zero downtime. Essentially, each deployment is created in a new "release" folder on the server, and then it is symlinked from the docroot on successful completion. [Read more about atomic deployments in Deploybot](https://deploybot.com/blog/deploy-complex-apps-with-atomic-sftp-deployments).

#### Build Tools
This is my favourite feature. Build tools allow you to spin up a [Docker](https://www.docker.com) container (on the Deploybot server) to run your build commands (Gulp, Grunt, Composer etc). You can use one of the provided Deploybot containers, or you can build your own. Having versioned compiled files in the past (purely for deployment purposes), this feature means compiled assets no longer need to be versioned, and you can avoid those annoying merge conflicts. Schweeeet! [Read more about Build Tools in Deploybot](http://support.deploybot.com/article/791-setting-up-and-using-build-tools).

## Summary
So far, I'm really liking this new setup. Having all repositories in one place (Bitbucket) makes it much easier when working within a team, and having the wiki feature for documenting projects is tremendously useful. Deployments with Deploybot are a breeze, and with the build tools feature, I finally have the fine-tuned workflow I always wanted.

Would love to know what repository and deployment tools you're using. Let me know in comments below.
