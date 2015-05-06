---
layout: post
title: "Basic web app with Phoenix"
description: ""
category:
tags: [phoenix elixir]
---
{% include JB/setup %}



## Overview

In this blog post i'm going to be building a basic web app with the [Phoenix framework](http://www.phoenixframework.org/v0.12.0). I want to pick up from where the getting started guide for phoenix left you. Let's create a nested model relationship app. I hope to also show my little neat tricks i've learned along the way. Just a heads up, I'm writing this blog like you've already gone through the getting started guide for elixir and you're ready for the next step. So if you're feeling a little lost please consult the [Overview page](http://www.phoenixframework.org/v0.12.0/docs/overview).


This will be a simple app that allows you to create users and then create cars for that user. Simple right? Let's go!


## Setup

Things you'll need:
* Postgres: `brew install postgresql`. This will be your DB service.
* Node & NPM: `brew install node`. You'll want NPM for all the JS goodies you get with Phoenix. Phoenix uses [Brunch.io](http://brunch.io/). It's the first time i've heard of it and it looks really cool!
* Elixir: [http://elixir-lang.org/install.html](http://elixir-lang.org/install.html)
* Phoenix:
  * `$ mix local.hex`
  * `$ mix archive.install https://github.com/phoenixframework/phoenix/releases/download/v0.12.0/phoenix_new-0.12.0.ez`

Alright let's create our simple app.

## Create a new Phoenix app


First we'll create our app with the `mix phoenix.new` command.
```
$ mix phoenix.new simple_phoenix_app
```

When it asks:
* `Install mix dependencies? [Yn]` say yes.
  * yes.
* `Install brunch.io dependencies? [Yn]`
  * yes.

Now you should have all your dependencies to create the app.

That's going to install all the things you need for your phoenix app.
I've never heard of Brunch before Phoenix but it's a really nice template generator. check it out here:
(http://brunch.io/)

Okay now we have our new Phoenix project. Let's run it.
```
$ cd simple_phoenix_app
$ mix phoenix.server
```
Now you should be able to open your browser and visit http://localhost:4000


Just like you would see in Rails. It's a simple welcome page. Yay!

Okay now it's time for me just to show you the coolest "batteries included" feature about this framework. Open the `simple_phoenix_app/web/templates/page/index.html.eex` file in your editor. Then make sure you have the editor and browser pointed at http://localhost:4000 and it's visible. Now at the top of the page add a `h1` to the jumbotron div. like:  

```html
<div class="jumbotron">
  <h1>O Snap!</h1>
  <h2>Welcome to Phoenix!</h2>
  <p class="lead">Most frameworks make you choose between speed and a productive environment. <a href="http://phoenixframework.org">Phoenix</a> and <a href="http://elixir-lang.org">Elixir</a> give you both.</p>
</div>
```
Then hit save. OMG!! You catch that?! Auto-reloading baked in!!

## Create user scaffolding

Alright let's create our first model. We'll use the scaffold like command for Phoenix.

    $ mix phoenix.gen.html User users name email

The command above is creating from the output:
* A `User` model
* A `users` table
* With the string attributes `name` and `email`
* The controller, view, and templates needed for requests
* And some tests for you.

Now let's follow the directions from the output.

Add `resources "/users", UserController` to the `router.ex` file.
```elixir
### simple_phoenix_app/web/router.ex
scope "/", SimplePhoenixApp do
  pipe_through :browser # Use the default browser stack

  get "/", PageController, :index
  resources "/users", UserController
end
```
Before we migrate the DB we'll edit the DB connection info, or you can leave it if you have a postgres user with the password set to postgres, but I don't so i'm going to change mine to my username. I you change this stetting be sure to restart your server or else it won't pick up the changes.
```elixir
### simple_phoenix_app/config/dev.exs
...
# Configure your database
config :simple_phoenix_app, SimplePhoenixApp.Repo,
  adapter: Ecto.Adapters.Postgres,
  username: "meatherly",
  database: "simple_phoenix_app_dev"
```

Now let's create the DB and Migrate.
```
$ mix ecto.create
$ mix ecto.migrate
```
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

## Create car model

Now lets create our Car model for our users. Well use the model generator because we're not creating a top level route.

    $ mix phoenix.gen.model Car cars name year:integer

We need to edit the migration first to make the referenced foreign key for the users and cars. To do that we'll need to add the `references` function to the user_id column.

```elixir
  ### simple_phoenix_app/priv/repo/migrations/*_create_car.exs
  defmodule SimplePhoenixApp.Repo.Migrations.CreateCar do
    use Ecto.Migration

    def change do
      create table(:cars) do
        add :user_id, references(:users)
        add :name, :string
        add :year, :integer

        timestamps
      end
    end
  end
```

Now let's migrate our DB to add the latest migration

`$ mix ecto.migrate`

## Adding a relationship between cars and users

We'll need to edit our car and user model and add the relationships to it

```elixir
### simple_phoenix_app/web/models/user.ex
###...
schema "users" do
  has_many :cars, SimplePhoenixApp.Car
  field :name, :string
  field :email, :string

  timestamps
end
###...
```

```elixir
### simple_phoenix_app/web/models/car.ex
###...
schema "cars" do
  belongs_to :user, SimplePhoenixApp.User
  field :name, :string
  field :year, :integer

  timestamps
end
###...
```

## Adding Car controller, view and templates

Now let's create our Controller, View, and Templates.

__Controller__
``` elixir
### simple_phoenix_app/web/controllers/car_controller.ex
defmodule SimplePhoenixApp.CarController do
  use SimplePhoenixApp.Web, :controller

  alias SimplePhoenixApp.Car

  plug :action
end
```

__View__
```elixir
### simple_phoenix_app/web/views/car_view.ex
defmodule SimplePhoenixApp.CarView do
  use SimplePhoenixApp.Web, :view

end
```

__Templates__
`simple_phoenix_app/web/templates/index.html.eex`
```html_ruby
<h2>Listing cars for <%= @user.name %></h2>

<table class="table">
  <thead>
    <tr>
      <th>Name</th>
      <th>Year</th>
    </tr>
  </thead>
  <tbody>
<%= for car <- @cars do %>
    <tr>
      <td><%= car.name %></td>
      <td><%= car.email %></td>
    </tr>
<% end %>
  </tbody>
</table>

<%= link "New car", to: user_car_path(@conn, :new, @user) %>
```

`simple_phoenix_app/web/templates/new.html.eex`
```html_ruby
<h2>New car for <%= @user.name %></h2>

<%= render "form.html", changeset: @changeset,
                        action: user_car_path(@conn, :create, @user) %>

<%= link "Back to cars", to: user_car_path(@conn, :index, @user) %>
```

`simple_phoenix_app/web/templates/form.html.eex`
```html_ruby
<%= form_for @changeset, @action, fn f -> %>
  <%= if f.errors != [] do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below:</p>
      <ul>
        <%= for {attr, message} <- f.errors do %>
          <li><%= humanize(attr) %> <%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="form-group">
    <label>Name</label>
    <%= text_input f, :name, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Year</label>
    <%= number_input f, :year, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit "Submit", class: "btn btn-primary" %>
  </div>
<% end %>
```

And one last thing let's add our cars route to the `router.ex` file. Just add the cars route inside a block for the users route.

```elixir
scope "/", SimplePhoenixApp do
  pipe_through :browser # Use the default browser stack

  get "/", PageController, :index
  resources "/users", UserController do
    resources "/cars", CarController
  end
end
```

Alright time to edit our CarController to add the needed plugs and actions. Let's start with a plug to find the user. Let's add the `find_user` function at the bottom of the controller. Then add the `plug :find_user` above the action plug since plugs are executed in the order they are defined.

```elixir
defmodule SimplePhoenixApp.CarController do
  #...
  plug :find_user
  plug :action
  #...
  defp find_user(conn, _) do
    user = Repo.get(SimplePhoenixApp.User, conn.params["user_id"])
    assign(conn, :user, user)
  end
end
```

Now let's add our index action to the controller

```elixir
def index(conn, _params) do
  user = conn.assigns.user
  cars = Repo.all assoc(user, :cars)

  render conn, cars: cars, user: user
end
```

You'll notice we get our user from the `conn.assigns` map that we made down in the `find_user` function. You can't really have instance variables flying around your controllers like you have in Ruby in Elixir. So we have to add assignments to the connection and pass that around.

Next lets add a link to the user's cars on the user's show page.

```html
<!-- simple_phoenix_app/web/templates/user/show.html.eex -->
<%= link "Back", to: user_path(@conn, :index) %>
| <%= link "Cars", to: user_car_path(@conn, :index, @user) %>
```

You can add this link to the bottom of the show page.

Now let's create a new user on our web app using the site. Navigate to [http://localhost:4000/users]() click on New user link.

Create the new user. Now submit the form and you should see your user. Click on the show button for that user and you should see the Cars link we made on the bottom of the page! YAY!

## Add new action for car controller

Now Let's add the new action to the car controller to render the new car form. And since we've made that find_user plug it's going to be smaller than you think :)

```elixir
#...
def new(conn, _) do
  changeset = Car.changeset(%Car{})
  render conn, changeset: changeset
end
#...
```
You might be wondering. Won't I need the user for the view since we have a `@user` variable in the view? Well no since we've added the user to the connections assignments that means it was sent down with the connection to the view and the view passed it along to the templates.

So pro tip: You can add things to the assigns to DRY up your controllers.

Okay now you can render the new car form from then car index page. YAY!

## Add create action for car controller

Now let's add the create action so we can submit the car form. This one will be a bit longer than the new action.

```elixir
def create(conn, %{"car" => car_params}) do

  changeset =
    build(conn.assigns.user, :cars)
    |> Car.changeset(car_params)

  if changeset.valid? do
    Repo.insert(changeset)

    conn
      |> put_flash(:info, "Car has been successfully created.")
      |> redirect(to: user_car_path(conn, :index, conn.assigns.user))
  else
    render(conn, "new.html", changeset: changeset)
  end
  render conn, changeset
end
```

Wow! right? lol. Let's talk about some of this. First off let's talk about that random build function just hanging out in there. Well I stumbled into this file `simple_phoenix_app/web/web.ex` and saw this where all the `use SimplePhoenixApp.Web, :view` are coming from. It has all the View, Model, Controller, and Router functions defined. In the controller one they `import Ecto.Model` which that has the build function. It's very similar to the ActiveRecord build method. You can read more about it [here](http://hexdocs.pm/ecto/). The rest of the action should look very similar to the UserController create action. So if the changeset isn't valid we're just rendering the new page again with the errors. Else we're sending them back the index page for the cars for the user.

Well what are you waiting for?! Try it out!!


## Ending notes

So how bout dem apples! we've built a simple Phoenix App I bet you might have a lot questions and I bet I don't have a lot of answers but if you head over to the IRC room #elixir-lang there are some great guys in there that would love to help. Even the creator of the Framework :)
