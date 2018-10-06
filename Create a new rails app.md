# Create a new Rails 5 app with RSpec, Postgres and Bootstrap.

Features :

- Using postgres development, test and production
- Git for version control
- RSpec and Capybara for testing
- Bootstrap for views
- Deployment to Heroku

## Create the app

````bash
$ rails new <app_name> -d postgresql -T
$ cd <app_name>
````

## Do some Git 

Initiialise Git and do the initial commit.

````bash
$ git init							#Initialise Git repository
$ git add -A						#Add all new files
$ git commit -m "Initial commit"	#Commit all the new files
````



## Add Bootstrap

Add the Boostrap Gem to make thing look nice.

````ruby
gem 'bootstrap', '~> 4.0.0.beta'
````



Rename `app/assets/stylesheets/application.css` to `app/assets/stylesheets/application.scss`

Delete it's contents and add :

```ruby
@import "bootstrap";
```

##### Custom Bootstrap Variables

Custom Bootstrap Variables must be set or imported before Bootrap.

For instance :

```ruby
$body-bg: #eee;
$body-color: #212529;
@import "bootstrap";
```

Or imported via a seperate file such as `app/assets/stylesheets/custom_bootstrap_variables.scss` :

```ruby
@import "custom_bootstrap_variables";
@import "bootstrap";
```

A [full list of the variables](https://github.com/twbs/bootstrap-rubygem/blob/master/templates/project/_bootstrap-variables.scss) is available, copy/paste and uncomment any you wish to change.

##### Bootstrap JavaScript

Bootstrap JavaScript depends on jQuery. Bootstrap tooltips and popovers depend on [popper.js](https://popper.js.org/) for positioning. 

First add the `jquery-rails` Gem to `/Gemfile`

```ruby
gem 'jquery-rails'
```

Then add the following to `app/assets/javascript/application.js`

```
//= require jquery3
//= require popper
//= require bootstrap
```

Don't forget to install the Gems.

```bash
$ bundle install 
```



## Add RSpec

Add in some Gems. Edit **/Gemfile **with...

````ruby
group :development, :test do
  ...
  gem 'rspec-rails', '~> 3.5'
  gem 'shoulda-matchers'
  gem 'capybara'
  gem 'factory_bot' 
end
````

Install RSpec

````bash
$ bundle install 
$ rails g rspec:install
````

Sometimes `rails g spec:install` will hang. In which case use `spring stop` before installing it.

#### Capybara

In  **spec/rails_helper.rb** add...

```ruby
require 'capybara/rails'
```

to the bottom of the file and include...

```ruby
config.include Capybara::DSL
```

in the `RSpec.configure do |config|` block.

Lastly put the following into the bottom of your `spec/spec_helper.rb` file:

```
require 'capybara/rspec'
```



## Test the testing 

To do this we need something to test.

#### Add a static page

Create a controller and view for a static page. This will create the controller, view and spec files.

````bash
$ rails g controller static home
````

#### Edit the view

Edit **/app/views/static/home.html.erb** to contain 

````html
<p>Hello World!</p>
````

#### Update the default route

Edit **/config/routes.rb** 

```ruby
Rails.application.routes.draw do
  ...
  root 'static#home'
end
```

#### Create DB's

Rails expects DB's to exist.

````bash
$ rails db:create && rails db:migrate
````

#### Visit homepage

```bash
$ rails server
```

```bash
=> Booting Puma
=> Rails 5.0.2 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.8.2 (ruby 2.2.4-p230), codename: Sassy Salamander
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://localhost:3000
Use Ctrl-C to stop
```

Visit http://localhost:3000

Check that you see **Hello World!** 

#### Alternatively use Pow to serve the site

Assumes [Pow](http://pow.cx) and [Powder](https://github.com/powder-rb/powder) are installed.

Link the site to Pow. From within the project directory.

```bash
$ powder link
```

Ensure Pow is running.

```bash
$ powder start
```

Visit your site at [http://your-app-name.dev](http://your-app-name.dev)

#### Running RSpec

Run RSpec to ensure that everyhing is working correctly.

````bash
$ rspec
````

You will see a number of test failures. That is because RSpec creates dummy tests when it creates controller. Lets replace them.

Comment out everything in **/spec/controllers/static_controller_spec.rb** and **/spec/helpers/spec_helper.rb**

In **/views/static/home.html.erb_spec.rb** replace the contents with...

```ruby
require 'rails_helper'

describe 'the homepage' do
  before do
    visit '/'
  end

  it 'should say hello world!' do
    expect(page).to have_content "Hello World!"   
  end
end
```

Then run RSPec again and we should have success.

```bash	
$ rspec
```



#### Adding guard

Guard watches your application for changes and runs various tasks when it see's them. This includes running the tests and reloading the browser.

```ruby
gem "guard"
gem "guard-rspec"
gem "guard-livereload"
gem "rack-livereload"
```

Install the Gems.

```bash
$ bundle install 
```

Get things set up...

```bash
$ bundle exec guard init 				# Create a Guardfile	
$ bundle exec guard init livereload 	# Add livereload to Guardfile
$ bundle exec guard init rspec			# Add RSpec to the Guardfile
```

Check **/Guardfile** that the Livereload block is before the RSpec block. This means the browser update will happen before your RSpec tests run.

#### Configure Rack Livereload

This allows the Rack App (that Rails sits on) to automatically inject the required Javascript to reload the page. Meaning you don't need browser extensions. In **config/enviroments/development.rb** add the follwoing to the bottom of the `Rails.application.configure do` block.

```ruby
Rails.application.configure do
  ...
  config.middleware.insert_after ActionDispatch::Static, Rack::LiveReload
end
```

#### Running Guard

You run Guard with 

````bash
$ bundle exec guard 		
````

If you run into problems it could be that a guard or rails process is running in the background. 

#### Using Forman to run everything

Install `foreman`. This shouldn't be in your `Gemfile`

```bash
$ gem install foreman
```

Create `Procfile.dev` 

````Bash
$ touch Procfile.dev
````

With the following...

````yaml
rails: rails s
guard: bundle exec guard
````

Now we can start up our app *locally* using...

```bash
$ foreman start -f Procfile.dev
```

#### Create a startup script

````bash
$ touch bin/start
````

With the following...

```bash
foreman start -f Procfile.dev
```

And ensure it has the right permissions to run...

```bash
$ chmod +x bin/start
```

You can now start with 

```bash
$ bin/start
```

#### Getting better errors

Two more Gems to add to /Gemfile in the development group.

```ruby
gem "better_errors"
gem "binding_of_caller"
```

This gives us way better error reporting.



## Deploy to Heroku

Create a new Heroku app and delpoy

````bash
$ heroku create
$ git push heroku master
````

And visit the site

````bash
$ heroku open
````
