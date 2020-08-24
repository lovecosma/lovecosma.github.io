---
layout: post
title:      "The Last Error"
date:       2020-08-24 00:36:31 +0000
permalink:  the_last_error
---

I just finished my first project using Rails... *Whew*  It actually wasn't very hard at all. Sinatra prepared me perfectly for comprehension of the way Rails works. Active Record is also making it very easy to manage databases. One interesting feature of Rails is the ability to make associations between tables/objects. In Digital Me, the numerology chart collector that I continued working on from Sinatra for this most recent project, being able to associate objects is a key feature.  Also, having the ability to validate data is very important. A useful as they are, these features gave me a headache when in the early stages of developing my application. Here's why:


I have three types of objects. A User, a Chart, and a *Number Object* (a number object consists of 7 different subclasses, but all 7 have the same basic features ex: Birthday Number, Life Path Number).


User -< Charts >- Number Objects


A user has a collection of charts. Each chart is associated with one of each of the subclasses of number objects. However, one number object can have multiple charts that belong to it (ex: multiple charts can have a Birthday Number 4). To make things more interesting, A user has many number objects through the charts it has, and a number object has many users through the charts that belong to it. 


```
#app/models/User.rb

class User

  has_many :charts
  has_many :birthdays, through: :charts
	validates :username, presence: true, uniqueness: true
  validates :password_digest, presence: true
  has_secure_password

end 
```

```
#app/models/chart.rb

class Chart

  belongs_to :user
  belongs_to :birthday
	validates :first_name, presence: true, format: { with: /\A[a-zA-Z]+\z/,
    message: "only allows letters" }
  validates :middle_name, presence: true, format: { with: /\A[a-zA-Z]+\z/,
    message: "only allows letters" }
  validates :last_name, presence: true, format: { with: /\A[a-zA-Z]+\z/,
    message: "only allows letters" }
  validates :birthdate, presence: true
	
end 
```

```

#app/models/birthday.rb
class Birthday

has_many :charts
has_many :users, through: :charts
validates_presence_of :number


end 
```



(I'm only including the birthday number in this example, just know that the same process is repeated with the other 6 number objects) The charts act as a join table between the users and the number objects. This means the charts table will have to have spots to hold the Ids of the number objects and users to which it belongs.  Notice as well, the validations. The chart must have a first name, middle name, last name, and birthdate. These numbers are used to determine a person's 7 number objects. There's a huge problem though. A chart needs to belong to 1 of each of the 7 number objects, the name is needed to calculate which object the chart belongs to. 


```
???
```


For a while back there, this one was actually really tricky. It was one of the last bugs I had to fix before finishing up the project and it was stumping me. See, each number object needs to have a number out of 0,1,2,3,4,5,6,7,8,9,11,22,33. Which number a chart is assigned to cannot be determined until a valid full name and birthdate are submitted. In order for a valid full name and birthdate to be submitted completely, the chart must have a number that is dependent on that info. 



I created the number 44. When the create chart form is submitted with a valid full name and birthdate, number 44 is assigned to each number object of that chart. This will allow the chart to be manually validated with valid? and persisted in the database as valid. No charts number could ever possibly be assigned to 44 otherwise. Then, another round of assignments takes place. First, #save then checks to make sure the info was saved. If so, the actual values are assigned using the valid full name and birthdate and the chart is saved with these new values. Now the data is persisted to the charts table is being validated and the three model types are able to keep their associations.


```
def create
    @chart = Chart.new
    @chart = current_user.charts.build(chart_params)
      @chart.birthday = Birthday.find_by(number: 44)
      @chart.life_path = LifePath.find_by(number: 44)
      @chart.soul_urge = SoulUrge.find_by(number: 44)
      @chart.soul_urge_challenge = SoulUrgeChallenge.find_by(number: 44)
      @chart.expression = Expression.find_by(number: 44)
      @chart.expression_challenge = ExpressionChallenge.find_by(number: 44)
      @chart.personality = Personality.find_by(number: 44)
      @chart.personality_challenge = PersonalityChallenge.find_by(number: 44)
      @chart.valid?
    if @chart.save
      @chart.birthday = Birthday.find_by(number: @chart.birthday_number)
      @chart.life_path = LifePath.find_by(number: @chart.life_path_number)
      @chart.soul_urge = SoulUrge.find_by(number: @chart.soul_urge_number)
      @chart.soul_urge_challenge = SoulUrgeChallenge.find_by(number: @chart.soul_urge_challenge_number)
      @chart.expression = Expression.find_by(number: @chart.expression_number)
      @chart.expression_challenge = ExpressionChallenge.find_by(number: @chart.expression_challenge_number)
      @chart.personality = Personality.find_by(number: @chart.personality_number)
      @chart.personality_challenge = PersonalityChallenge.find_by(number: @chart.personality_challenge_number)
      @chart.save!
      redirect_to user_chart_path(current_user, @chart)
    else
      render :new
    end

  end
```



This was a cool solution for me since my birthday number and life path number are both 4. I was also born in the 4th month on the 13 which reduces to 4. It only seemed right to imprint my numbers on every chart that is made even if only momentarily. This kind of logic was responsible for much of the whimsy throughout the site. I had fun making the project and look forward to what else Flatiron has in store.
