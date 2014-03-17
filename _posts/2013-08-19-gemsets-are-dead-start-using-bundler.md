---
layout: post
title: "Gemsets are dead. Start using Bundler."
category: ruby
tags: [rvm-gemsets, bundler, rvm, rails]
---
{% include JB/setup %}

Over the past year now at my job, we have been using RVM. We decided to go with it because we work on multiple projects with different ruby versions and the projects have different gem versions. Us being the noobs that we are, gemsets made sense logically to us. You don’t want to mix up your gem versions going from project to project right?

Well a year has past and I hate dealing with RVM gemsets. Having to make new .rvmc files for every project, making sure you’re in the right gemset before running any bundler commands if didn’t make a .rvmc file, and having to do a fresh bundle install every time for a new project gets pretty old (btw. V8 takes forever to install!). 

This frustration lead me to research if RVM gemsets are even worth the headache. I also remembered back in my Windows days, when I was playing with Rails, that I didn’t have RVM but I was able to use different gem versions on different projects. This lead me to this article by Steven Ball. My favorite part and pretty much the summary of the article is this:

>If you’re already using bundler to manage the gems for all of your projects (and you should) then you can just stop dealing with gemsets and let bundler sort it all out for you.

You should basically read the article if you have the time. Alright, moving on. What’s so cool about bundler is that it will save all the different versions of your gem once you pull them down with bundle install. Bundler even says this in their docs:

>If any of the needed gems are already installed, Bundler will use them. After installing any needed gems to your system, bundler writes a snapshot of all of the gems and versions that it installed to `Gemfile.lock`.

And bonus, if you have the gem already you won’t have to pull it down again when you’re in a different project! All you have to do is specify what gem version you want to use in your gemfile and bundler will use it. If you don’t specify one then it will just pull down the latest version. Basically what it all boils down to for me is that when dealing with a project you don’t need to worry about your gems versions not touching from project to project, you just need to worry about your ruby version you are using with your project. I mean look what Bundler says on their front page:

>Bundler maintains a consistent environment for ruby applications. It tracks an application’s code and the rubygems it needs to run, so that an application will always have the exact gems (and versions) that it needs to run.

Sounds like gemsets doesn’t it? I think people are still using gemsets because they didn’t realize bundler did this. Also Bundler was baked in to Rails 3. So before that people were using gemsets to manage their gem versions.

For me it’s time to say good bye to gemsets and finally start letting Bundler do it’s job! Don’t get me wrong, I’m not saying to get rid of RVM. You can still use RVM but now without the burden of gemsets. Just use it to manage your ruby versions.

---

References: 

* <http://bundler.io/v1.3/rationale.html>
* <http://rakeroutes.com/blog/how-to-use-bundler-instead-of-rvm-gemsets/>


