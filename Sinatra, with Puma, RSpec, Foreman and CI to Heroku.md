#Sinatra, with Puma, RSpec, Foreman and Circle CI to Heroku

A Tutorial on how to get continuous deployment with Circle CI set up.

### Initial setup

Make a new folder, add some initial files...

```bash
$ mkdir test_app
$ cd test_app
$ touch Gemfile
$ touch app.rb
```

In `Gemfile` add

````Ruby
#Gemfile
source 'https://rubygems.org'
ruby '2.4.2'
gem 'sinatra'
````

In `app.rb` add :

````ruby	
# app.rb
require 'sinatra'

get '/' do
  "Hello World"
end
````

Install Gems :

````bash
$ bundle install
````

To test : 

````bash	
$ ruby app.rb
````

And visit [http://localhost:4567](http://localhost:4567)

Initialise Git and Commit files

````bash	
$ git init
$ git add -A
$ git commit -m "Initial Commit"
````

### Deploy to Heroku

Create a `config.ru` file. Heroku requires this.

````ruby	
# config.ru
require 'sinatra'
require './app'

run Sinatra::Application
````

Add to Git

```bash
$ git add -A
$ git commit -m 'Add config.ru'
```

Create a Heroku App and deploy

````bash
$ heroku create
$ git push heroku master
````

Visit the live app

```bash
$ heroku open
```

### Set up Puma, foreman and code reloading

Add some Gems to `Gemfile`

````ruby
...

group :test, :development do
  gem 'puma'
  gem 'rerun'
end

````

And install them...

```bash
$ bundle install
```

Then install `foreman`. This shouldn't be in your `Gemfile`

```bash
$ gem install foreman
```

Create `Procfil.dev` and add...

```yaml
webserver: bundle exec rerun -- rackup -s puma -p 8000
```

This will start up our app using `rerun` (which will reload the app on code changes) using the `Puma` web server. The same one that Heroku uses.

Now we can start up our app *locally* using...

```bash
$ foreman start -f Procfile.dev
```

And visit our app at [http://localhost:8000](http://localhost:8000)

To make it a little easier create a bash script in `bin/start`

```bash
s Procfile
```

And ensure it has the right permissions to run...

```bash
touch$ chmod +x bin/start
```

You can now start with 

```bash
$ bin/start
```

Heroku requires a Procfile if you want to specify which web server and/or services/workers it uses.

Create `Procfile` with

```yaml
web: bundle exec rackup config.ru -p $PORT 
```

Commit everything to Git

```bash
$ git add -A
$ git commit -m 'Add Procfiles, foreman and rerun'
```



### Add RSpec and Capybara for testing

We need to add a couple more Gems to the development / test group.

```Ruby
group :test, :development do
  ...
  gem "rspec"
  gem 'rack-test'
  gem 'capybara'
  gem 'rspec_junit_formatter' # required for Circle CI
end
```

`rspec_junit_formatter` isn't required for local testing but is when we use Circle CI to run automated tests. So we'll add it here. Don't forget to `$ bundle install`.

Initialise RSpec :

```bash
$ rspec --init
```

This will create a couple of files : `.rspec` and `spec/spec_helper.rb`

You don't need to do anything with `.rspec` ([unless you want to](https://github.com/rspec/rspec/wiki)). In `spec/spec_helper.rb` add the following to the top...

```ruby
# require our applications
require_relative '../app'

# Gems required to run the testing.
require 'sinatra'
require 'capybara/dsl'
require 'rack/test'

# Set up the testing enviroment
set :environment, :test
set :run, false
set :raise_errors, true

# Module to contain our app
module RSpecMixin
 def app() Sinatra::Application end
end

RSpec.configure do |config|
  include Rack::Test::Methods   # Ensure we use the Rack Test Methods
  config.include RSpecMixin     # Include our App.
end
```

It up to you how much of the rest of the config information already in `spec/spec_helper.rb` you decide to use.

#### Lets write a test

Add `spec/app_spec.rb`

```ruby	
describe 'Our test app' do  

  context 'Basic functionailty' do
    before(:all) do
       get '/'
    end
    
    it 'Responds to an initial get call on /' do 
      expect(last_response).to be_ok
    end

    it 'Says Hello' do
      expect(last_response.body).to include("Hello") 
    end
  end
end


```

Run all tests with...

```Bash
$ rspec
```

or a specific test (we currently only have one) with...

```bash	
$ rspec spec/app_spec.rb
```

Commit everything to Git.

### Push to Github

Circle CI pulls everything in from Github so we need to get that set up. First create a new repositiry on Github. Then...

```bash
git remote add origin https://github.com/YOUR_USER_NAME/WHATEVER_YOU_CALLED_YOUR_REPO.git
git push -u origin master
```

### Config Circle CI

In [Circle CI](https://circleci.com/dashboard) Click **Projects** and **Add Project**. Assuming you signed up for Circle CI with you Github account, you'll see a list of your repos. 

Click Setup Project for the repo you just created.

In the config window. Choose Platform 1.0 and Ruby for the language. Then click **Start Building**. Circle CI infers all your project settings and then does it stuff.

You should have a succesful build.

Whenever you make a change to your app and push it to Github. Circle CI sees the change and starts a build.

#### Config deployment to Heroku

To actually do the deployment to Heroku we need to add the Heroku API Key to the Circle CI project settings. 

Get your Heroku API key from your [Heroku Account Page](https://dashboard.heroku.com/account)

Paste that in to your [Heroku API Key](https://circleci.com/account/heroku) settings in Circle CI.

Next create a new file `circle.yml` in your project with the following :

```Yams
deployment:
  prod:
    branch: master
    heroku:
      appname: YOUR_HEROKU_APP_NAME
```

Where `YOUR_HEROKU_APP_NAME` is the name Heroku gave your app when it was created. 

You can find it with :

````bash
$ heroku info
````

Now. Add that to Git.

```bash
git commit -am 'Added Circle CI deployment config'
```

Lastly make a change so we know the deployment works.

In `app.rb` change it to...

```ruby
require 'sinatra'

get '/' do
  "Hello World. Looks like continuous deployment is set up."
end
```

Add it to Git and push

```bash
$ git commit -am 'Updated Hello World text.'
$ git push
```

This will push the commit to Github and trigger Circle CI.

You can monitor your build in the Circle CI Dashboard. And when the build is complete visit your live site using `$ heroku open`

Boom!!! You have continuious deployment set up.