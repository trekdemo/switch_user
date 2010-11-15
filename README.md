switch_user
===========

Inspired from [hobo][0], switch_user provides a convenient way to switch current user that speeds up your development.

Use Case
--------

switch_user is very useful in such use cases

1. switch current users in development so that you don't waste your time to logout, login and input email (login) or password any more.

2. reproduce the user specified error in production. Sometimes the error is only raised for specified user, which is difficult to reproduce for developers, switch_user can help you reproduce it by login as that user.

Example
-------

Visit here: [http://switch-user-example.heroku.com/][1], switch the current user in the select box.

Install
-------

Add in Gemfile.

    gem "switch_user"

Usage
-----

Add following code into your layout page.

erb

    <%= switch_user_select %>

or haml

    = switch_user_select
    
If there are too many users (in production), the switch_user_select is not a good choice, you should call the switch user request by yourself.

    <%= link_to user.login, "/switch_user?scope_id=user_#{user.id}" %>
    <%= link_to admin.login, "/switch_user?scope_id=admin_#{admin.id}" %>
    
    = link_to user.login, "/switch_user?scope_id=user_#{user.id}"
    = link_to admin.login, "/switch_user?scope_id=admin_#{admin.id}"

If you use it in a Rails 2 project, you have to add a route manually.

    # config/routes.rb
    map.switch_user '/switch_user', :controller => 'switch_user', :action => 'set_current_user'

Configuration
-------------

By default, you can switch between Guest and all users in users table, you don't need to do anything. The following is the default configuration.

    SwitchUser.setup do |config|
      # provider may be :devise or :authlogic
      config.provider = :devise

      # available_users is a hash, 
      # key is the model name of user (:user, :admin, or any name you use), 
      # value is a block that return the users that can be switched.
      config.available_users = { :user => lambda { User.all } }

      # available_users_identifiers is a hash
      # keys in this hash should match a key in the available_users
      # hash
      # value is the name of the identifying column to find by,
      # defaults to id
      # this hash is to allow you to specify a different column to
      # expose for instance a username on a User model instead of id
      config.available_users_identifiers = { :user => "id" }

      config.available_users_identifiers
      # what field should be displayed on select box
      config.display_field = :email

      # controller_guard is a block, 
      # if it returns true, the request will continue, 
      # else the request will be refused and returns "Permission Denied"
      # if you switch from "admin" to user, the current_user param is "admin"
      config.controller_guard = lambda { |current_user, request| Rails.env.development? }

      # view_guard is a block, 
      # if it returns true, the switch user select box will be shown, 
      # else the select box will not be shown
      # if you switch from admin to "user", the current_user param is "user"
      config.view_guard == lambda { |current_user, request| Rails.env.development? }

      # redirect_path is a block, it returns which page will be redirected 
      # after switching a user.
      config.redirect_path = lambda { |request, params| '/' }
    end

If the default configuration can't meet your requirement, you can define your customized configuration in <code>config/initializaers/switch_user.rb</code>

If you want to switch both available users and available admins

    config.available_users = { :user => lambda { User.available }, :admin => lambda { Admin.available } }
    
If you want to display the login field in switch user select box

    config.display_field = :login
    
If you only allow switching from admin to user in production environment

    config.controller_guard = lambda { |current_user, request| Rails.env == "production" and current_user.admin? }
    
If you only want to display switch user select box for admins in production environment

    config.view_guard = lambda { |current_user, request| Rails.env == "production" and current_user and current_user.admin? }
    
If you want to redirect user to "/dashboard" page

    config.redirect_path = lambda { |request, params| "/dashboard" }
    

Copyright © 2010 Richard Huang (flyerhzm@gmail.com), released under the MIT license

[0]: https://github.com/tablatom/hobo
[1]: http://switch-user-example.heroku.com/

