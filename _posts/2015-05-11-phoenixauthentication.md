---
layout: post
title: "Phoenix app with home made authentication"
description: "Rolling in the deep with Authentication and Phoenix"
category:
tags: [phoenix authentication comeonin plug]
---
{% include JB/setup %}


## Overview

Ever wonder how hard it is to implement authentication with Phoenix? Ever wondered how hard it is to roll your own authentication in Phoenix? Well in this blog post we'll find out!

We'll be making an app called Phitter. An where users can post their thoughts that are 255 characters in length for all to see. In order to do this we'll need 2 models: User and Pheets. We'll learn how to make a Plug for authentication.

Before I get started just wanted to let you guys in on a few things:

I want to thank all those who've helped me make this possible by helping me with all my questions:
 * [ericmj](https://github.com/ericmj)
 * [chrismccord](https://github.com/chrismccord)
 * [crododile](https://github.com/crododile)
 * [jpiked](https://twitter.com/jpiked)

I got my first pull request merged into Ecto!! https://github.com/elixir-lang/ecto/pull/597#issuecomment-102262215 Yay!

And lastly, If you didn't know my wife and I just got our referral for a little girl from South Korea. We're doing some fund raising to support our adoption so wanted to invite you guys to Show off your smarts and support adoption! http://buff.ly/1Q0caW0

Alright let's go!

### What we're going to do

* Make a user model. User will have a username and encrypted_password.
* We'll make 2 virtual fields: password and password_confirmation.
* * Make a custom validation to validate those fields.
* We'll make a registration controller for allowing user's to sign up.
* We'll make a session controller to validate and control user session.
* We'll use the [comeonin](https://github.com/elixircnx/comeonin) package for encrypting passwords.
* We'll make a Pheet model. This will just have 2 fields. user_id and body

### Make a new project

```
mix phoenix.new phitter
```

### Edit the application.html.eex layout

Let's make a few changes to the application layout before we move on to making our user model.


```
pitter/web/templates/layout/application.html.eex
```

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Phitter</title>
    <link rel="stylesheet" href="<%= static_path(@conn, "/css/app.css") %>">
  </head>

  <body>
    <div class="container">
      <p class="alert alert-info"><%= get_flash(@conn, :info) %></p>
      <p class="alert alert-danger"><%= get_flash(@conn, :error) %></p>
    </div>
    <div class="text-center">
      <h1>Phitter</h1>
    </div>
    <div class="container">
      <%= @inner %>
    </div> <!-- /container -->
    <script src="<%= static_path(@conn, "/js/app.js") %>"></script>
    <script>require("web/static/js/app")</script>
  </body>
</html>
```

### Create a user model

If you've forgotten the generator for model like I have many times, you can just run `mix help | grep phoenix` in your shell.

Let's create a user model with these fields: username:string, encrypted_password:string

```
$ mix phoenix.gen.model User users username encrypted_password
```

Since we're not store the `password` and `password_confirmation` we'll have to make them virtual attributes using Ecto. We'll also want to make sure we validate those fields. We'll need a custom validation since ecto doesn't have a `validates :confirmation` yet.

``` elixir
defmodule Phitter.User do
  use Phitter.Web, :model

  schema "users" do
    field :username, :string
    field :encrypted_password, :string
    field :password, :string, virtual: true
    field :password_confirmation, :string, virtual: true
    timestamps
  end

  @required_fields ~w(username password password_confirmation)
  @optional_fields ~w()

  @doc """
  Creates a changeset based on the `model` and `params`.

  If `params` are nil, an invalid changeset is returned
  with no validation performed.
  """
  def changeset(model, params \\ nil) do
    model
    |> cast(params, @required_fields, @optional_fields)
    |> validate_unique(:username, on: Phitter.Repo, downcase: true)
    |> validate_length(:password, min: 1)
    |> validate_length(:password_confirmation, min: 1)
    |> validate_confirmation(:password)
  end

  def validate_confirmation(changeset, field) do
    value = get_field(changeset, field)
    confirmation_value = get_field(changeset, :"#{field}_confirmation")
    if value != confirmation_value, do: add_error(changeset, :"#{field}_confirmation", "does not match"), else: changeset
  end
end
```

You see our custom validation function in our user model called `validate_confirmation`. You'll notice we either call the `add_error/3` (which returns the changeset with errors added to a field) or return the changeset. All validation functions must return a changeset. I found that out the hard way :P

We also validate the username field and make sure it's unique.

### Adding User to the web.ex file

Since we're going to be using the User model so much let's go ahead and add it to our `web.ex` file under the controller function.

``` elixir
### phitter/web/web.ex
def controller do
  quote do
    use Phoenix.Controller
    #...
    alias Phitter.User
    #...
  end
end
```

### Registration Controller

We'll implement the  `RegistrationController` step by step. Let's Go!

Let's create our registrations controller to allow the user to create an account.

``` elixir
## phitter/web/controllers/registration_controller.ex
defmodule Phitter.RegistrationController do
  use Phitter.Web, :controller

  plug :action

  def new(conn, _params) do
    changeset = User.changeset(%User{})
    render conn, changeset: changeset
  end

end
```

We'll also need to go ahead an make the view and templates needed for the registration controller.

``` elixir
### phitter/web/views/registration_view.ex
defmodule Phitter.RegistrationView do
  use Phitter.Web, :view
end
```

Let's make our registration form.

`phitter/web/templates/registration/new.html.eex`

``` html
<h3>Registration</h3>
<%= form_for @changeset, registration_path(@conn, :create), fn f -> %>
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
    <label>Username</label>
    <%= text_input f, :username, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Password</label>
    <%= password_input f, :password, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Password Confirmation</label>
    <%= password_input f, :password_confirmation, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit "Register", class: "btn btn-primary" %>
    <%#= link("Login", to: session_path(@conn, :new), class: "btn btn-success pull-right") %>
  </div>
<% end %>
```

*(we've commented out the session_path link because we haven't implemented that yet but we will soon.)*

Let's add the controller actions for the RegistrationController.

``` elixir
# phitter/web/router.ex

scope "/", Phitter do
  pipe_through :browser # Use the default browser stack
  get "/registration", RegistrationController, :new
  post "/registration", RegistrationController, :create

  get "/pages", PageController, :index
end
```


Check out the new action by going to [http://localhost:4000/registration](http://localhost:4000/registration)

You should be able to see your new registration form! Yay! Now let's add the `create/2` action for our registration controller.

### RegistrationController create/2

Next let's add our `comeonin` package for encryption.

``` elixir
## phitter/mix.exs
##....
defp deps do
  [{:phoenix, "~> 0.12"},
   {:phoenix_ecto, "~> 0.3"},
   {:postgrex, ">= 0.0.0"},
   {:phoenix_live_reload, "~> 0.3"},
   {:cowboy, "~> 1.0"},
   {:comeonin, "~> 0.10"}]
end
```

Now let's implement the RegistrationController's create action. This will create the user and store their ecto model in the session if all their validations pass or it will render the `new/2` action if the validations fail.


``` elixir
# phitter/web/controllers/registration_controller.ex

defmodule Phitter.RegistrationController do
  #...
  alias Phitter.Password

  plug :scrub_params, "user" when action in [:create]
  plug :action

  #...

  def create(conn, %{"user" => user_params}) do
    changeset = User.changeset(%User{}, user_params)
    if changeset.valid? do
      new_user = Password.generate_password_and_store(changeset)

      conn
        |> put_flash(:info, "Successfully registered and logged in")
        |> put_session(:current_user, new_user)
        |> redirect(to: page_path(conn, :index))
    else
      render conn, "new.html", changeset: changeset
    end
  end
end
```

You'll notice that we're aliasing the `Phitter.Password` and that's something we haven't implemented yet. Instead of putting all the registration functions in the user model we're going to put them in a different module. This isn't probably isn't needed because we only have one model with passwords but I just wanted to show you how easy it is to split up your functionality with Functional Programming. Let's go create our `Phitter.Password` module.

```elixir
# phitter/lib/phitter/password.ex

defmodule Phitter.Password do
  alias Phitter.Repo
  import Ecto.Changeset, only: [put_change: 3]
  import Comeonin.Bcrypt, only: [hashpwsalt: 1]

  @doc """
    Generates a password for the user changeset and stores it to the changeset as encrypted_password.
  """

  def generate_password(changeset) do
    put_change(changeset, :encrypted_password, hashpwsalt(changeset.params["password"]))
  end

  @doc """
    Generates the password for the changeset and then stores it to the database.
  """
  def generate_password_and_store_user(changeset) do
    changeset
      |> generate_password
      |> Repo.insert
  end
end
```

Alright, our `RegistrationController` should be finished now. We can visit the registration form and test it now. After you create a user it should redirect to the pages index page with a flash message letting you know that you've logged in. If something goes wrong it should render the new page again with errors. We have it redirecting to the pages controller but we'll change that to the `PheetController` soon.

### Creating the SessionController

Now that we can sign up let's go ahead and make it where we can sign in. To do that we'll need to make the session controller. This controller will have 3 actions: `new/2`, `create/2`, and `delete/2`. First let's create a view for our `SessionController`.

```elixir
# phitter/web/views/session_view.ex

defmodule Phitter.SessionView do
  use Phitter.Web, :view
end
```

Let's add `SessionController`

```elixir
# phitter/web/controllers/session_controller.ex

defmodule Phitter.SessionController do
  use Phitter.Web, :controller

  plug :scrub_params, "user" when action in [:create]
  plug :action

  def new(conn, _params) do
    render conn, changeset: User.changeset(%User{})
  end

  def create(conn, %{"user" => user_params}) do
    user = if is_nil(user_params["username"]) do
      nil
    else
      Repo.get_by(User, username: user_params["username"])
    end

    user
      |> sign_in(user_params["password"], conn)
  end

  def delete(conn, _) do
    delete_session(conn, :current_user)
      |> put_flash(:info, 'You have been logged out')
      |> redirect(to: session_path(conn, :new))
  end

  defp sign_in(user, password, conn) when is_nil(user) do
    conn
      |> put_flash(:error, 'Could not find a user with that username.')
      |> render "new.html", changeset: User.changeset(%User{})
  end

  defp sign_in(user, password, conn) when is_map(user) do
    cond do
      Comeonin.Bcrypt.checkpw(password, user.encrypted_password) ->
        conn
          |> put_session(:current_user, user)
          |> put_flash(:info, 'You are now signed in.')
          |> redirect(to: page_path(conn, :index))
      true ->
        conn
          |> put_flash(:error, 'Username or password are incorrect.')
          |> render "new.html", changeset: User.changeset(%User{})
    end
  end
end

```

```elixir
#phitter/web/router.ex

#...

scope "/", Phitter do
  pipe_through :browser # Use the default browser stack

  get "/", SessionController, :new
  post "/login", SessionController, :create
  get "/logout", SessionController, :delete
  get "/registration", RegistrationController, :new
  post "/registration", RegistrationController, :create

  get "/pages", PageController, :index
end

```

last let's add our session template for the new action.

```html
<h3>Login</h3>
<%= form_for @changeset, session_path(@conn, :create), fn f -> %>
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
    <label>Username</label>
    <%= text_input f, :username, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Password</label>
    <%= password_input f, :password, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit "Login", class: "btn btn-primary" %>
    <%= link("Sign Up", to: registration_path(@conn, :new), class: "btn btn-success pull-right") %>
  </div>
<% end %>
```

We also uncomment the session link in our registration form at `phitter/web/templates/registrations/new/html.eex` form.

What we have here is the `sign_in` function that will check if the user is nil or it will try to log in the user if the map is given from `Repo.get/2`. The `cond` condition looks odd but basically it's checking the password submitted by the user and if that is true it will log in. If that doesn't work then it will default the the `true ->` value and tell the person they can't login with that username and password combination.

We then store the user model to the session like we did in the `RegistrationController` so we can access the user in with the `current_user` value in the session. We're redirecting to the page's controller for now but once we implement the Pheet controller and model then all this will work together a little better. With that, let's do it!

### Creating the Pheet model

You can either create the Migration, Model, Controller, View, And Templates manually or create them with `mix phoenix.gen.html` and edit them to your needs. I'm going to use `mix phoenix.gen.html`

```
mix phoenix.gen.html Pheet pheets body
```

Let's edit the Migration and Model's to set up the relationship between the user and pheet.


```elixir
defmodule Phitter.Repo.Migrations.CreatePheet do
  use Ecto.Migration

  def change do
    create table(:pheets) do
      add :body, :string
      add :user_id, references(:users)


      timestamps
    end
    create index(:pheets, [:user_id])
  end
end
```
We've just added the `:user_id` reference column. Now migrate your database.

Now let's edit the models now. I'm not going to add

``` elixir
# phitter/web/models/user.ex

#...
schema "users" do
  has_many :pheets, Phitter.Pheet
  field :username, :string
  field :encrypted_password, :string
  field :password, :string, virtual: true
  field :password_confirmation, :string, virtual: true
  timestamps
end
#...
```

Now we'll be able to tell who said what in our pheets page.

``` elixir
# phitter/web/models/pheet.ex

defmodule Phitter.Pheet do
  use Phitter.Web, :model

  schema "pheets" do
    belongs_to :user, Phitter.User
    field :body, :string

    timestamps
  end

  @required_fields ~w(body user_id)
  @optional_fields ~w()

  @doc """
  Creates a changeset based on the `model` and `params`.

  If `params` are nil, an invalid changeset is returned
  with no validation performed.
  """
  def changeset(model, params \\ nil) do
    model
    |> cast(params, @required_fields, @optional_fields)
  end
end
```

We've added the relationship between user and pheet. We've got `user_id` and `body` in the required_fields so that they'll throw errors if they are nil. *(If you haven't noticed by now. The required_fields is basically validates presence true in Rails ActiveRecord validations)*

Now that we have that let's add our controllers and views for Pheets

### Creating Pheet Controller

The controller is pretty simple. To make this app easy you'll only be aloud to create Pheets. This way we'll only need 3 actions `index/2`, `new/2`, and `create/2`.

We'll also play around with Ecto.Query. Specifically the `preload` and `order_by` functions. Take a look:

``` elixir
defmodule Phitter.PheetController do
  use Phitter.Web, :controller

  alias Phitter.Pheet

  plug Phitter.Plug.Authenticate
  plug :scrub_params, "pheet" when action in [:create, :update]
  plug :action

  def index(conn, _params) do
    pheets = Repo.all from p in Pheet,
      order_by: [desc: p.updated_at],
      preload: [:user]

    render(conn, "index.html", pheets: pheets)
  end

  def new(conn, _params) do
    changeset = Pheet.changeset(%Pheet{})
    render(conn, "new.html", changeset: changeset)
  end

  def create(conn, %{"pheet" => pheet_params}) do
    new_pheet = build(conn.assigns.current_user, :pheets)
    changeset = Pheet.changeset(new_pheet, pheet_params)

    if changeset.valid? do
      Repo.insert(changeset)

      conn
      |> put_flash(:info, "Pheet created successfully.")
      |> redirect(to: pheet_path(conn, :index))
    else
      render(conn, "new.html", changeset: changeset)
    end
  end
end
```

Now I'll let you go ahead and create your `PheetView` and I'll post the views below. *(I need to find out how to make it default to a ApplicationView if you don't need one. I feel odd just creating blank view modules. Feel free to comment if you know anything about that.)*


### Pheet new.html.eex, form.html.eex, index.html.eex

The templates are pretty simple. Just add the user to the index and we're good.


new.html.eex

``` html
<h2>New pheet</h2>

<%= render "form.html", changeset: @changeset,
                        action: pheet_path(@conn, :create) %>

<%= link "Back", to: pheet_path(@conn, :index) %>
```

form.html.eex

``` html
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
    <label>Body</label>
    <%= text_input f, :body, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit "Submit", class: "btn btn-primary" %>
  </div>
<% end %>

```

index.html.eex

``` html

<div>
  Hello, <%= @current_user.username %> <%= link "Logout", to: session_path(@conn, :delete) %>
</div>

<div class="text-center">
  <%= link "New pheet", to: pheet_path(@conn, :new), class: "btn btn-primary " %>
</div>



<div id="pheets-wrapper">
  <%= for pheet <- @pheets do %>
      <div class="pheet">
        <div class="pheet-author">
          <%= pheet.body %>
        </div>
        <div class="pheet-author">
          - <%= pheet.user.username %>
        </div>
      </div>
  <% end %>
</div>

```
