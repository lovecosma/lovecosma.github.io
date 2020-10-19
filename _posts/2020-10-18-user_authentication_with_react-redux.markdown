---
layout: post
title:      "User Authentication with React-Redux"
date:       2020-10-19 02:51:57 +0000
permalink:  user_authentication_with_react-redux
---


I will explain how I achieved user authentication using Ruby on Rails api and a React-redux application. I will then demonstrate how to keep a user logged in through a page refresh. 

First, you must set up a user model in Rails. This can be achieved by running 

To complete this, we will need gem 'bcrypt', gem 'rack-cors', 

```
rails new user-auth-demo
bundle install
rails g scaffold user first_name last_name email password_digest 
```

This will create a scaffold for the users table that accepts the listed attributes. The user model will need to accept  a password digest in order to add layers of security to the authentication process. This is being made available by gem 'bcrypt'. To account for this, we must add to the users model:

```
// users.rb
class User
has_secure_password
end 
```

Now, we should be able to create a new user as long as we pass it the required values. For instance, if we run:

```
rails c
user = User.create(first_name: "Matthew", last_name: "Doe", email: "test@test.com", password: "12345678" )
user => {UserObject}
```
