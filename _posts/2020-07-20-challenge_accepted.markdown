---
layout: post
title:      "Challenge Accepted"
date:       2020-07-20 01:51:03 -0400
permalink:  challenge_accepted
---

Over the course of the past few weeks, I’ve had the pleasure of acquainting myself with the simplicity of Sinatra in the most unorthodox of ways! I started my journey in the dry mountains of Colorado. I was living in a motel with naught but the mastery of software development on my mind. I was speeding through lessons with complete comprehension and for a while there, I felt like things were getting too easy. I was familiar with SQL from previous experiences, and Inactive /ActiveRecord made life even. Suddenly though, a windfall of change came about. First, money started to run low and Colorado is not the cheapest place to live, let me tell you. So I got a job; it’s a simple customer service. At this same time though, it was becoming time to retrieve my daughter from vacation in Georgia. Also, I was to make this trip alone with my wife between us getting off of work one Friday and needing to clock back in the following Monday. To make things even more complicated, my cohort was moving on to this new unit, Sinatra??

The challenge was very intimidating. However, I had three things in my arsenal that would assist me in the task of completing this 45-hour road trip from Ft. Collins, Co, to Savannah, Ga, to Houston, Tx, to Thibodaux, La:

I missed my daughter incredibly. She has been gone for a month and it was the longest that my wife and I had ever spent from her.

I’m a formidable opponent.

Sinatra ended because incredibly easy to understand and use. The idea of creating a web application with user authentication sounded daunting, but Sinatra’s user-friendly methods and versatility made building my first web application much more digestible than I would have imagined. Specifically, I wondered how my server would know which user is signed in. This process is actually quite simple when broken down with the tools offered by Sinatra:

**post '/login' do**   

#when a post request is made to  '/login' 

**@user = User.find_by_username(params[:user][:username]) **

#make this user the the user that is found in a database of users whose username matches the one passed in from             the  previous 'get' request. Before a username was created at signup it was validated for uniqueness. It's a good                     primary key to use

**if @user && @user.authenticate(params[:user][:password])**

#if a user is found in the previous line and the user's password that was also retrieved in get '/login' is that user's                   password...the password authentication is made available by line 3 in user.rb 'has_secure_password'. This takes                   whatever password the user creates, it interpolates random characters to make it even more unique and secure.                   When #athenticate is called in line three of this code, it triggers the authentication of the password by matching what          the user provided at get '/login' what was established as the user's password during signup despite the extra layer              of secure provided by password disgest

**session[:user_id] = @user.id**

#if the previous line was truthy, the user for this session is this particular user and no other one. The session hash               keeps track of which user is logged in preventing a user from continuously having to re-login everytime a page is                 requested

**redirect '/charts'**

#after making the current session belong to the user found in line 2, authenticated in line 3, and assigned to this                   session in line 4, redirect to the '/charts'. This will only show that user's information since the user is now logged-in

**else**

#if line 3 is falsey...

**erb :'sessions/new'**

#rerender erb:'sessions/new' which is called in get '/login' or retry signing up because there was an error. 

I’m excited to see how rails will make this process even simpler!

In the end, we all made it safely through the journey. Instead of traveling back to Colorado, we did a Grand Tour of sorts to be with family and friends while we power through this expansive phase in our lives. Since beginning Flatiron School, I’ve had significant growth as a software developer, artist, and entrepreneur. There’s no doubt in my mind that these things are all closely intertwined. I’ve realized that seizing opportunities is more of a challenging balancing act than anything.
