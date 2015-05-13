---
layout: post
title: "Phoenix app with home made authentication"
description: "Rolling in the deep with Authentication and Phoenix"
category:
tags: [phoenix authentication comeonin plug]
---
{% include JB/setup %}


## Overview

Ever wondered how hard it is to implement authentication with Phoenix? Ever wondered how hard it is to roll your own authentication in Phoenix? Well in this blog post we'll find out!

We'll be making an app called Phitter. An where users can post their thoughts that are 255 characters in length for all to see. In order to do this we'll need 2 models: User and Pheets. We'll learn how to make a Plug for authentication.

Alright let's go!

### Create a user model

If you've forgotten the generator for model like I have many times, you can just run `mix help | grep phoenix` in your shell.

```
$ mix phoenix.gen.model User users username encrypted_password first_name last_name
```

Alright let's make sure we set up our validations for our changeset. We want to make sure that the username is unique.

First off since we're going to be using the User model so much let's go ahead and add it to our `web.ex` file under the controller function.

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

`pitter/web/templates/layout/application.html.eex`

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Hello Phitter!</title>
    <link rel="stylesheet" href="<%= static_path(@conn, "/css/app.css") %>">
  </head>

  <body>
    <div class="container">
      <p class="alert alert-info"><%= get_flash(@conn, :info) %></p>
      <p class="alert alert-danger"><%= get_flash(@conn, :error) %></p>

      <%= @inner %>

    </div> <!-- /container -->
    <script src="<%= static_path(@conn, "/js/app.js") %>"></script>
    <script>require("web/static/js/app")</script>
  </body>
</html>
```


`phitter/web/templates/registration/new.html.eex`

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
    <label>Name</label>
    <%= text_input f, :name, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Username</label>
    <%= text_input f, :username, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Password</label>
    <%= text_input f, :password, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Password Confirmation</label>
    <%= text_input f, :password_confirmation, class: "form-control", placeholder: 'hello' %>
  </div>

  <div class="form-group">
    <%= submit "Submit", class: "btn btn-primary" %>
  </div>
<% end %>
```


Let's add our Pheet model for the user to create Pheets.

```
mix phoenix.gen.html Pheet pheets body
```

before we migrate let's add the user relationship so we know who said what. 
