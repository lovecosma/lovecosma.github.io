---
layout: post
title:      "Rendering media with React-Redux using Rails Active Storage Part 1"
date:       2020-10-20 16:30:27 -0400
permalink:  user_authentication_for_react-redux_and_rails_api_react-redux_side
---


If you have ever wanted to include media such as audio files, images, or even videos in your React-Redux single-page application, then this is the blog post for you. Some key challenges to this feat:

* Handling file storage in Rails backend (Active Storage)
* Handling successful passage of file from client to backend API and vice versa
* Using rendering media with stored file

Before tackling this task, you should have a solid comprehension of asynchronous requesting to Rails API with React-Redux using middleware 'thunk'. This process is made relatively simple with a two additional dependencies:

* Rails Active Storage (addition to Rails app)
* Tone.js (addition to React app)

I will be demonstrating this process by making a simple audio sampler where a user can upload files and trigger them with buttons. 

https://media.tenor.co/images/3781fb49f9f50c4b7d5c9590dcf9e710/tenor.gif?riffsid=TVRRMk5EY3dNRGhmWmc9PTB6cvK4Lr2R4JqtQNexIFfoxiTO4izGG7YT2rvecv2NykJVJ4P_BOCdKNFR21S0hk0

To get started with this, we're going to need to get our apps organized and initialized. I like to put my client and backend directories in the same parent directory. We'll call this parent directory SimpleSamplerDemo. Inside that parent directory we will create a rails api by the name of  "sampler-backend"  and a react app called "sampler-client":

```
$ mkdir  SimpleSamplerDemo
$ cd SimpleSamplerDemo
$ rails new sampler-backend --api --database=postgresql  //Using postgresql for easy deployment later
$ create-react-app sampler-client
$ ls
=> 
sampler-backend       sampler-client
```


Next let's add what will need for CORS AJAX calls. Add gem 'rack-cors' to gem file and add the following information in cors.rb:

```
$ gem install 'rack-cors'
$ bundle
```
```
config/intitializers/cors.rb

# Add

Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```


Let's change our puma.rb file run rails on port 3001 since react will need port 3000

```

config/puma.rb

...
port        ENV.fetch("PORT") { 3001 }
...

```

Now let's add active storage to Rails. Active Storage will allow us to upload files to a cloud storage service like Amazon S3, Google Cloud Storage, or Microsoft Azure Storage and attach those files to Active Record objects. In this tutorial, I will only store the files locally using Active Storage. You can find details on this process [here](https://edgeguides.rubyonrails.org/active_storage_overview.html):

```
cd sampler-backend
rails db:create
rails active_storage:install
rails db:migrate
```

After running the above commands, your schema should now contain the necessary tables for storing blobs and attachments:

```
db/schema.rb

ActiveRecord::Schema.define(version: 2020_10_29_190857) do

  # These are extensions that must be enabled in order to support this database
  enable_extension "plpgsql"

  create_table "active_storage_attachments", force: :cascade do |t|
    t.string "name", null: false
    t.string "record_type", null: false
    t.bigint "record_id", null: false
    t.bigint "blob_id", null: false
    t.datetime "created_at", null: false
    t.index ["blob_id"], name: "index_active_storage_attachments_on_blob_id"
    t.index ["record_type", "record_id", "name", "blob_id"], name: "index_active_storage_attachments_uniqueness", unique: true
  end

  create_table "active_storage_blobs", force: :cascade do |t|
    t.string "key", null: false
    t.string "filename", null: false
    t.string "content_type"
    t.text "metadata"
    t.bigint "byte_size", null: false
    t.string "checksum", null: false
    t.datetime "created_at", null: false
    t.index ["key"], name: "index_active_storage_blobs_on_key", unique: true
  end


  add_foreign_key "active_storage_attachments", "active_storage_blobs", column: "blob_id"
end
```

Yay! Now your models will have the ability to have files attached to them. We can actually make a model with the ability to attach the file very easily:

```
$ rails g scaffold samples name url
```

This generates a model with a name and url attribute, both strings. With one simple adjustment to the model using a method available to us from Active Storage, we are able to attach an avatar to the object which will act as a bridge between the file and the object:

```
app/models/sample.rb

class Sample
has_one_attached :avatar
end 
```


Now we're almost done setting up the backend for file torage. The last adjustments we will need to make are in samples_controller.rb:

```
app/controllers/samples_controller.rb

...
def create

sample = Sample.create(name: sample_params[:name])  # Use the name from params to assign the sample's name upon initialization

    if(params[:file])  //Is the parameter for file empty?
			sample.avatar.attach(sample_params[:file])     # If not, #attach to the avatar the file object that is in params[:file]
      sample.url = url_for(sample.avatar)     # Save as the object url the url to the object's avatar using #url_for
    end 

    if sample.save    # If the object can be saved
      render json: sample, status: :created, location: sample //render the object as JSON
    else
      render json: sample.errors, status: :unprocessable_entity
    end

end 


def sample_params
      params.require(:sample).permit(:name, :file) # Permit the file to be passed into sample_params
 end

```

As a recap, we have completed the following tasks for our Simple Sampler App :

* Created parent directory and initialized backend api and client
* Installed gem 'rack-cors' 
* Changed puma.rb, cors.rb for AJAX calls from React
* Installed active storage and migrated the associated tables tables
* Generated scaffold for samples
* Adjusted sample.rb to allow attachments via avatar
* Adjusted controller action to accept files and attach them to avatar 


Theoretically, you should be able to run 'rails s' in your console and use an API tester like HTTPie or Postman to make a post request to your API (http://localhost:3001/samples) with params of name and a file. If you were to do this, you should receive a status code 200 ok in the API tester, and in your rails server, you will see that the file has been saved to the active storage databases.


