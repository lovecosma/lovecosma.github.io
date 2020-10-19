---
layout: post
title:      "Setting up Rails API for User Authentication with React-Redux"
date:       2020-10-18 22:51:59 -0400
permalink:  user_authentication_with_react-redux
---


I will explain how I achieved user authentication using Ruby on Rails api and a React-redux application. I will then demonstrate how to keep a user logged in through a page refresh. 

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
rails g scaffold email password_digest passwrod_confirmation
rails db:migrate
=>
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


This will create a scaffold for the users table that accepts the listed attributes. The user model will need to accept  a password digest in order to add layers of security to the authentication process. This is being made available by gem 'bcrypt'. To account for this, we must add to the users model:

```
// users.rb
class User
has_secure_password
end 
```

We should be able to create a new user as long as we pass it the required values. For instance, if we run:

```
//console

rails c
user = User.create(email: "test@test.com", password_digest: "12345678", password_confirmation: "12345678")
	 
	 
 => #<User id: 1, email: "test@test.com", password_digest: [FILTERED], password_confirmation: nil, created_at: "2020-10-19 15:08:27", updated_at: "2...
```

Now a user exists on the backend. If we run our server, that user's information is now being served on the API. Now we must create a session controller to keep track of the session. Run:

```
rails g controller sessions
```

Set up some helper methods in the application controller:

```
class ApplicationController < ActionController::Base
skip_before_action :verify_authenticity_token
helper_method :login!, :logged_in?, :current_user,     :authorized_user?, :logout!, :set_user
    
def login!
      session[:user_id] = @user.id
end
def logged_in?
      !!session[:user_id]
end
def current_user
      @current_user = User.find(session[:user_id]) if session[:user_id]
end
def authorized_user?
       @user == current_user
end
def logout!
       session.clear
end
def set_user
    @user = User.find_by(id: session[:user_id])
end
end

```

Adjust the sessions controller as such:

```
/app/controllers/sessions_controller.rb

class SessionsController < ApplicationController

    def create
		
        @user = User.find_by(email: session_params[:email]) 
      
        if @user && @user.authenticate(session_params[:password])
          login!
          render json: {
            status: 200,
            logged_in: true,
            user: @user
          }
        else
          render json: { 
            status: 401,
            errors: ['no such user, please try again']
          }
        end
    end
		
		
    def destroy
          logout!
          render json: {
            status: 200,
            logged_out: true
          }
    end
		
    private
    def session_params
          params.require(:user).permit(:email, :password)
    end
    end
		
		```
		
		
	Params will be comming from the user login page in the React frame work. Almost done setting up the Rails side. Lastly, we must create the routes to allow us to make requests:
	
	```
	/config/routes.rb
	
	Rails.application.routes.draw do
  post '/login',    to: 'sessions#create'
  post '/logout',   to: 'sessions#destroy'
  resources :users
  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
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


Everything we need to create a user and login is now set up and ready. We can start configuring our front end. In my next blog, I will be continbuing this concept and connecting the readct-redux front end.


