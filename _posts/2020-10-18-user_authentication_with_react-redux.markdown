---
layout: post
title:      "Setting up Rails API for User Authentication with React-Redux"
date:       2020-10-18 22:51:59 -0400
permalink:  user_authentication_with_react-redux
---


I will explain how to set up user authentication for a React-Redux app using a Ruby on Rails API.

To complete this, we will need gem 'bcrypt', gem 'rack-cors' as dependencies. You can uncomment them in the gem file or add them if they aren't there. Then run:

```
rails new user-auth-demo
bundle install
```

Before starting on the user model, we must configure  rack-cors which will allow us to access our server from our frontend. 

```
config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'http://localhost:3000' //Whatever domain is attempting to access. For me, React is running on localserver:3000
    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

Speaking of which, we must change Rails's server to port 3001 to accomodate React on port 3000. Change this in puma.rb.

```
/config/initializers/puma.rb

...
port        ENV.fetch("PORT") { 3001 } //Initial set to 3000. Setting to 3001.
...

```

Now we are ready to start building our user model. Run:

```
rails g scaffold email password_digest 
rails db:migrate

```

Add you should see:

```
create    db/migrate/20201019145923_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
      invoke  resource_route
       route    resources :users
      invoke  scaffold_controller
      create    app/controllers/users_controller.rb
      invoke    test_unit
      create      test/controllers/users_controller_test.rb

```

 To account for the password_digest we must add to the users model:

```
// users.rb
class User
has_secure_password
end 
```

We should be able to create a new user as long as we pass it the required values. For instance, if in the console we run:

```
rails c

User.create(email: "test@test.com", password_digest: "12345678", password_confirmation: "12345678")
	 
 => #<User id: 1, email: "test@test.com", password_digest: [FILTERED], password_confirmation: nil, created_at: "2020-10-19 15:08:27", updated_at: "2...
```

Now a user exists on the backend for user to log in. In the future, this user-creation process would be handled by something like a 'user#create' action in the users controller. 

```
rails g controller sessions
```


Adjust the sessions controller as such:

```
/app/controllers/sessions_controller.rb


class SessionsController < ApplicationController

    def create
        @user = User.find_by_email(user_params[:email])  
				
        if @user && @user.authenticate(user_params[:password]) 
				
        session[:user_id] = @user.id  
        render json: @user 
				
        else
				
            render json: { 
                status: 401,
                errors: ['no such user, please try again']
              }
							
        end
				
      end
			
      private
			
  def user_params
    params.require(:user).permit(:email, :password)
  end
	
end
```


 Essentially, the login action will find the user that matches the email passed through params. If the user exists and their password is correct., (#bcrypt), that user is the the user of this session. Then render that user as json and send it as a response. If the user does not exist or their password_digest coukldn't be authenticated, it will send an error message. While we're at it, let's add a destroy action for logging out:
 
 ```
 /app/controllers/sessions_controller.rb
 
 ...
 
 def destroy
 
    session.clear
    redirect_to root_path
		
  end
	```
 
 Next, let's set up the login/logout route:
 
 
 ```
 /config/routes.rb
 
 Rails.application.routes.draw do
  resources :users
  post "/login", to: 'sessions#create'
	 post "/logout", to: 'sessions#destroy'
end
```
 
 
If we run in the terminal:

```
rails routes
=>
login POST   /login(.:format)                                                                         sessions#create
logout POST   /logout(.:format)                                                                        sessions#destroy
users GET    /users(.:format)                                                                         users#index
POST   /users(.:format)                                                                         users#create
user GET    /users/:id(.:format)                                                                     users#show
PATCH  /users/:id(.:format)                                                                     users#update
PUT    /users/:id(.:format)                                                                     users#update
DELETE /users/:id(.:format)                                                                     users#destroy
```


Everything we need to create a user and login is now set up and ready. We can start configuring our front end. The frontend will make fetch requests to this API with the React-Redux framework. In my next blog, I will be continbuing this concept.


