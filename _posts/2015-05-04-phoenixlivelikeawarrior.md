---
layout: post
title: "Basic web app with Phoenix"
description: ""
category:
tags: []
---
{% include JB/setup %}


In this blog post i'm going to building a basic web app with the [Phoenix framework](http://www.phoenixframework.org/v0.12.0).


This app will be a simple app that allows you to create users and then  create cars for that user. Simple right? Let's go!

You'll need to install [Elixir] and [Phoenix] you can do that here:
* Postgres: `brew install postgresql`
* Node & NPM: `brew install node`. You'll want NPM for all the JS goodies you get with Phoenix.
* Elixir: http://elixir-lang.org/install.html
* Phoenix:
  * `$ mix local.hex`
  * `$ mix archive.install https://github.com/phoenixframework/phoenix/releases/download/v0.12.0/phoenix_new-0.12.0.ez`


Alright let's create our simple app.

First we'll create our app with the `mix phoenix.new` command.

      $ mix phoenix.new simple_phoenix_app

When it asks `Install mix dependencies? [Yn]` say yes.

When it asks `Install brunch.io dependencies? [Yn]` say yes.

This will use NPM to install the

That's going to install all the things you need for your phoenix app.
I've never heard of Brunch before Phoenix but it's a really nice template generator. check it out here: [Brunch.io](http://brunch.io/)

Okay now we have our new Phoenix project. Let's run it.

    $ cd simple_phoenix_app
    $ mix phoenix.server

Now you should be able to open your browser and visit http://localhost:4000


Just like you would see in Rails. It's a simple welcome page. Yay!

Okay now it's time for me just to show you the coolest "batteries included" feature about this framework. Open the `simple_phoenix_app/web/templates/page/index.html.eex` file in your editor. Then make sure you have the editor and browser pointed at http://localhost:4000 and it's visible. Now at the top of the page add a `h1` to the jumbotron div. like:  

``` html
<div class="jumbotron">
  <h1>O Snap!</h1>
  <h2>Welcome to Phoenix!</h2>
  <p class="lead">Most frameworks make you choose between speed and a productive environment. <a href="http://phoenixframework.org">Phoenix</a> and <a href="http://elixir-lang.org">Elixir</a> give you both.</p>
</div>
```
Then hit save. OMG!! You catch that?! Yeah auto-reloading baked in!!

Alright let's create our first model. We'll use the scaffold like command for Phoenix.

    $ mix phoenix.gen.html User users name email

The command above is creating from the output:
* A `User` model
* A `users` table
* With the string attributes `name` and `email`
* The controller, view, and templates needed for requests
* And some tests for you.

Now let's follow the directions from the output.

Add `resources "/users", UserController` to the `simple_phoenix_app/web/router.ex` file.

    scope "/", SimplePhoenixApp do
      pipe_through :browser # Use the default browser stack

      get "/", PageController, :index
      resources "/users", UserController
    end

Before we migrate the DB we'll edit the DB connection info, or you can leave it if you have a postgres user with the password set to postgres, but I don't so i'm going to change mine to my username. I you change this stetting be sure to restart your server or else it won't pick up the changes.

      ### simple_phoenix_app/config/dev.exs
      ...
      # Configure your database
      config :simple_phoenix_app, SimplePhoenixApp.Repo,
        adapter: Ecto.Adapters.Postgres,
        username: "meatherly",
        database: "simple_phoenix_app_dev"

Now let's create the DB and Migrate.

    $ mix ecto.create
    $ mix ecto.migrate

Then let's add a link to our home page that's points to the users index route.

    ### simple_phoenix_app/web/templates/page/index.html.eex

``` html
<div class="jumbotron">
  <h2>Welcome to Phoenix!</h2>
  <p class="lead">Most frameworks make you choose between speed and a productive environment.</p>
  <%= link "Users", to: user_path(@conn, :index) %>
</div>
```

Now you should be able to click on the users link on the home page. *I didn't have to refresh the page either :)*

Boom! We have our basic user index page!

You can go ahead and create a user and all that fun stuff. *Try not to let your face melt with the speed. lol.*


Now lets create our Car model for our users. 
