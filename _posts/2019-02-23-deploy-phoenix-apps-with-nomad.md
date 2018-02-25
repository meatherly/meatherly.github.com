---
layout: page
title: Deploy Phoenix Apps With Nomad
date: 2019-02-23 11:25 -0600
---

![](https://raw.githubusercontent.com/hashicorp/nomad/master/website/source/assets/images/logo-hashicorp.svg?sanitize=true)

Recently I've been playing around with deployments and I didn't realized there are as many options and tools out there as there are JS frameworks. So I started looking at [The Twelve-Factor App](https://12factor.net/) methodology and was like this is good but can I do this with Elixir and Phoenix? Turns out you can with [Distillery](https://hexdocs.pm/distillery) but it's just one component of deployments. Now how do I get the binary to the server and make sure it's configured right? With no surprise you have plenty of options to do that: Docker, Ansible, Puppet, Terraform, Nomad, Kubernetees and the list goes on. What makes it very confusing it they all seem to have tons of overlap. After more googling and clicking through links I came across the [12-Factor Apps and the HashiStack](https://www.youtube.com/watch?v=gf43TcWjBrE&t=1366s) dev talk by [Kelsey Hightower](https://twitter.com/kelseyhightower).

Before I move on I'll just say I'm not amazing at dev ops. I know enough to burn the house down. So with that, let's burn the house down :p

Things we'll be dealing with:
* hcl templating. A template engine made by hashicorp that's pretty straight forward to deal with.
* Vagrant. A tool to help to create an virtual environment to play with dev ops tools.
* Nomad.

btw can I just say I love Hashicorp because I feel like they get that there are fullstack developerz having todo ALL THE THINGS, which we know is impossible so they create these tools and document them really well for us. Hell, they even put [__Advanced Topic!__](https://www.nomadproject.io/docs/internals/consensus.html) at the top of docs so you know that you need to skim the contents or to hit the back button on your browser. LOL.

What is [Nomad](https://www.nomadproject.io/)? It's just a scheduler. The BEAM has scheduler in it for scheduling work for your all your processes. _AWWW YEAH_, with that, i'm signing off because you know everything about Nomad now. Nothing else to learn. Joking. But seriously a scheduler... Just like your OS has a scheduler in it scheduling resources for the applications you're using. If you're browsing the web then it's going to give more resources to the browser. If you're compiling code it's going to give resources to compiler to do that. I'm starting to learn that we deal with schedulers more than we realize. AWS is a scheduler, scheduling all the resources your application needs, Servers, Storages, Networks, and more.

![why though](https://i.imgur.com/73SL1o7.jpg){:height="250px"}

Nomad. y tho?
We've got the hottest tool combo out on the market right now for deploying and managing _ALL_ your applications, Docker + Kubernetees. True. But what I know about Elixir. It runs on a VM. My application is running on a VM which to Docker this just means it justs sees the VM running and doesn't know if my application is failing or running. Its like virtualization inception if you will. I've heard Java apps have the same going on. With Nomad you're not limited to just running things in Docker, you have all kinds of [drivers](https://www.nomadproject.io/docs/drivers/index.html)(look to the left under Drivers to see the list) to allow you to schedule work for.

What are we going to build here?
We're going to build a __non production ready__ environment to see some possibilities that Nomad has to offer to Elixir deployments. I'll be making this a series of posts so check back for more links at the bottom of the post to link you on the next post. We're going to deploy a Phoenix application called [hello_nomad]() using Nomad. It will only have 1 route that just shows the name of the instance of the app you hit on the page. It'll say "Hello from [instance name]".

Let's start of with a job. I'll let Hashicorp explain that:

> A Job is a specification provided by users that declares a workload for Nomad. A Job is a form of desired state; the user is expressing that the job should be running, but not where it should be run. The responsibility of Nomad is to make sure the actual state matches the user desired state. A Job is composed of one or more task groups.

With that let's create a job for nginx to load balance our work across our Phoenix apps. But wait Phoenix has this built in it by using processes in it to handler a large influx of requests. Yes, you're right but trust me you're going to want to see what nomad can do with templates. It has a very similar to Elixir developers, supervisors. Nomad has that built into it. It can watch for changes on your services and do actions based on the changes. It uses [Consul]() for service discovery and then allows you to make templates on those services to config other things in your infrastructure.

Let's take a look at the code for this:

``` hcl
template {
  data = <<EOF
  events { worker_connections 1024; }

  http {
    upstream hello_nomad {
      least_conn;
      {% raw %}{{range service "hello-nomad"}}
      server {{.Address}}:{{.Port}} weight=10 max_fails=3 fail_timeout=30s;
      {{else}}
      server 127.0.0.1:65535;
      {{end}}{% endraw %}
    }



    server {
      listen 8080;

      location / {
        proxy_pass http://hello_nomad;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
      }
    }
  }
  EOF
  destination   = "new/default.conf"
  change_mode   = "restart"
}
```



I'm using an inline template with some `{% raw %}{{ }}{% endraw %}` in it to tell Nomad to fill in for me from the data it gets from Consul. I'm saying for the service `hello-nomad` create a line like: `server 127.0.0.1:4000 weight=10 max_fails=3 fail_timeout=30s;` in the nginx upstream block. But if no services exist then fallback to `server 127.0.0.1:65535;` so that it just shows the infamous Bad gateway message. I'm also telling it where to place the config file with the `destination` variable and what the nomad should do to the task on change with the `change_mode` variable. Why is this cool? I can scale up or down as many instances of my application as I want and Nomad will change the nginx config file for me and reload it to update the changes. It's like a self healing infrastructure.