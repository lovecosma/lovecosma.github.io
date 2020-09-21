---
layout: post
title:      "Making a SPA with a Rails API and Javascript's fetch()"
date:       2020-09-21 01:21:33 +0000
permalink:  making_a_spa_with_a_rails_api_and_javascripts_fetch
---



My most recent project, Tarot Spread Explorer, is a single page web application that allows users to interact with Tarot spreads in a web browser. A single page application is a web application or website that interacts with the web browser by dynamically rewriting the current web page with new data from the webserver, instead of the default method of the browser loading entire new pages. Since single-page applications don’t update the entire page but only required content, they significantly improve a website’s speed.  This way users can have excellent interactive parallax scrolling and amazing transitions and effects to achieve an in-depth experience without bogging down the speed of the server with excess page requests. In Tarot Explorer, users are able to create new tarot spreads and save them to a list of all tarot spreads ever made by all other users of the applications. This process is handled by asynchronously sending HTTP requests to my local server from the frontend using Javascript's **fetch()**. 

To start off, we're going to need to run a server with data for us to request via json. Luckily, Rails makes this very easy by offering an --api tag to append when creating a new application. In my case, from my terminal I ran:

```
rails new tarot-explorer-backend --api
```

This command creates a new rails application without any views and with the capability to render request as JSON and serve that on their respective routes. For instance:

```
rails g scaffold cards name_short name value value_int meaning_up meaning_rev desc card_type image_url 
```


This will create a table :cards with :name, :value, and :card_type attribute. 

```
rails db:migrate
```

This command will create a Card model and cards_controller which comprise the 'MC' in our typical 'MVC' design pattern. In  cards_controller.rb, we will find our usually CRUD controller actions, except, instead of rendering erb HTML documents, we're rendering the data associated with the model in JSON.

```
class CardsController < ApplicationController
 before_action :set_card, only: [:show, :update, :destroy]

  # GET /cards
  def index
    @cards = Card.all

    render json: @cards
  end

  # GET /cards/1
  def show
    render json: @card
  end

  # POST /cards
  def create
    @card = Card.new(card_params)

    if @card.save
      render json: @card, status: :created, location: @card
    else
      render json: @card.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /cards/1
  def update
    if @card.update(card_params)
      render json: @card
    else
      render json: @card.errors, status: :unprocessable_entity
    end
  end

  # DELETE /cards/1
  def destroy
    @card.destroy
  end
```


In the case of Tarot Explorer, I actually used a previous API I built to seed in data for the deck of cards. Since I did that, when I visited http://localhost:3000/cards for the first time, I saw the JSON for each Card object in my database.

```
{
id: 1,
name_short: "ar01",
name: "The Magician",
value: "1",
value_int: "1",
meaning_up: "Skill, diplomacy, address, subtlety; sickness, pain, loss, disaster, snares of enemies; self-confidence, will;       the Querent, if male.",
meaning_rev: "Physician, Magus, mental disease, disgrace, disquiet.",
desc: "A youthful figure in the robe of a magician, having the countenance of divine Apollo, with smile of confidence and     shining eyes. Above his head is the mysterious sign of the Holy Spirit, the sign of life, like an endless cord, forming the         figure 8 in a horizontal position . About his waist is a serpent-cincture, the serpent appearing to devour its own tail. This     is familiar to most as a conventional symbol of eternity, but here it indicates more especially the eternity of attainment       in the spirit. In the Magician's right hand is a wand raised towards heaven, while the left hand is pointing to the earth.          This dual sign is known in very high grades of the Instituted Mysteries; it shews the descent of grace, virtue and light,          drawn from things above and derived to things below. The suggestion throughout is therefore the possession and              communication of the Powers and Gifts of the Spirit. On the table in front of the Magician are the symbols of the four          Tarot suits, signifying the elements of natural life, which lie like counters before the adept, and he adapts them as he            wills. Beneath are roses and lilies, the flos campi and lilium convallium, changed into garden flowers, to shew the culture    of aspiration. This card signifies the divine motive in man, reflecting God, the will in the liberation of its union with that          which is above. It is also the unity of individual being on all planes, and in a very high sense it is thought, in the fixation          thereof. With further reference to what I have called the sign of life and its connexion with the number 8, it may be                remembered that Christian Gnosticism speaks of rebirth in Christ as a change "unto the Ogdoad." The mystic number is    termed Jerusalem above, the Land flowing with Milk and Honey, the Holy Spirit and the Land of the Lord. According to         Martinism, 8 is the number of Christ.",
card_type: "major",
created_at: "2020-09-20T23:09:09.315Z",
updated_at: "2020-09-20T23:09:09.603Z",
suit: null,
image_url: "https://i.ibb.co/F8S1fMw/rwmagician.jpg",
}...
```


Before we move on to the frontend, we must make sure we enable CORS so that fetch will be able to make requests to the server. Add gem, 'rack-cors' to the gemfile for the project. Then in the terminal, run 

```
bundle install
```

Now go to your initializers folder in your rails app, then to the cors.rb file and uncomment this portion of code:


```
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'example.com'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```


Replace 'example.com' with a  " * " to allows all requests for development purposes. Later when the site is being hosted, we would put the domain name here.

```
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins ' * '

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```



So, now we have information that is easy to work with being served at a URL that can be accessed with an HTTP request. 
Now we can start building a fetch request from the front end. First, let's build the file structure for the frontend. To make Tarot Explorer, from the terminal I ran:

```
mkdir tarot-explorer-frontend
cd tarot-explorer-frontend
touch index.js
touch index.html
```

This creates an HTML document for the user to see and a javascript file to run fetch(). Next, we have to connect thee two files. Go to index.html in your IDE, and add basic HTML5 structure. Then add a script to connect index.js.

```
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    
		
		
	<script src="./index.js"></script>
  </body>
</html>
```


Almost there! Now in index.js we will add fetch() to send a get request to http://localhost:3000/cards. GET requests only require the URL of the data as an argument. fetch() returns a response object, so we will need to parse that for JSON. After retrieving that response in JSON, we will print the end result in the console. If there was an error with the request, the window will alert us with an error.


```
fetch(http://localhost:3000/cards)
.then(resp => resp.json())
.then(json => console.log()
.catch(error => alert(error)
```

Assuming no errors, the console will display all cards in JSON in the console

```
[
0: {id: 1, name_short: "ar01", name: "The Magician", value: "1", value_int: "1", …}
1: {id: 2, name_short: "ar02", name: "The High Priestess", value: "2", value_int: "2", …}
2: {id: 3, name_short: "ar03", name: "The Empress", value: "3", value_int: "3", …}
3: {id: 4, name_short: "ar04", name: "The Emperor", value: "4", value_int: "4", …}
4: {id: 5, name_short: "ar05", name: "The Hierophant", value: "5", value_int: "5", …}
5: {id: 6, name_short: "ar06", name: "The Lovers", value: "6", value_int: "6", …}
6: {id: 7, name_short: "ar07", name: "The Chariot", value: "7", value_int: "7", …}
7: {id: 8, name_short: "ar08", name: "Fortitude", value: "8", value_int: "8", …}
8: {id: 9, name_short: "ar09", name: "The Hermit", value: "9", value_int: "9", …}
9: {id: 10, name_short: "ar10", name: "Wheel Of Fortune", value: "10", value_int: "10", …}
...
]
```

We have successfully completed a fetch() request to a Rails API backend. The fetch request returns an array of JS objects that can be manipulated. More fetch() requests can be made for CRUD. These are listed in full in the cors.rb file.












 
