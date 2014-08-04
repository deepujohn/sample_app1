##Sample App
To create
````console
rails _4.0.8_ new sample_app -T
````
Gemfile
````ruby
source 'https://rubygems.org'
ruby '2.1.2'
#ruby-gemset=railstutorial_rails_4_0

gem 'rails', '4.0.8'

group :development, :test do
  gem 'sqlite3', '1.3.8'
  gem 'rspec-rails', '2.13.1'
end

group :test do
  gem 'selenium-webdriver', '2.35.1'
  gem 'capybara', '2.1.0'
end

gem 'sass-rails', '4.0.1'
gem 'uglifier', '2.1.1'
gem 'coffee-rails', '4.0.1'
gem 'jquery-rails', '3.0.4'
gem 'turbolinks', '1.1.1'
gem 'jbuilder', '1.0.2'

group :doc do
  gem 'sdoc', '0.3.20', require: false
end

group :production do
  gem 'pg', '0.15.1'
  gem 'rails_12factor', '0.0.2'
end
````
Now
````console
bundle update
````
Modify config/initializers/secret_token.rb
````ruby
require 'securerandom'

def secure_token
  token_file = Rails.root.join('.secret')
  if File.exist?(token_file)
    # Use the existing token.
    File.read(token_file).chomp
  else
    # Generate a new token and store it in token_file.
    token = SecureRandom.hex(64)
    File.write(token_file, token)
    token
  end
end

SampleApp::Application.config.secret_key_base = secure_token

````
Now install rspec
````console
rails generate rspec:install
````
Push the app to git and for version control
````console
git checkout -b static-pages
````
###Generate a controller

We can generate a controller with no test framework
````console
 rails generate controller StaticPages home help --no-test-framework
````
Notice that CamelCase plural is used for controller name.

###Testing our app

For testing our app we can generate one rspec integration test
````console
rails generate integration_test static_pages
````
To test our home page we can modify our spec/requests/static_pages_spec.rb

````ruby
require 'spec_helper'

describe "Static pages" do

  describe "Home page" do

    it "should have the content 'Sample App'" do
      visit '/static_pages/home'
      expect(page).to have_content('Sample App')
    end
  end
end
````
Now add capybara DSL to spec/spec_helper.rb
````ruby
RSpec.configure do |config|
  .
  .
  .
  config.include Capybara::DSL
end

````
For testing our app we can use

````console
bundle exec rspec spec/requests/static_pages_spec.rb
````
To pass test we can modify our home view as 
````html
<h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="http://railstutorial.org/">Ruby on Rails Tutorial</a>
  sample application.
</p>
````
Now for testing help add this to static_pages_spec.rb
````ruby
 describe "Help page" do

    it "should have the content 'Help'" do
      visit '/static_pages/help'
      expect(page).to have_content('Help')
    end
  end
````
*Try yourself pass this!*

Now after that add this 
````ruby 
 describe "About page" do

    it "should have the content 'About Us'" do
      visit '/static_pages/about'
      expect(page).to have_content('About Us')
    end
  end
````
###For testing title 
Now for testing title our static_pages_spec.rb should be
````ruby 
require 'spec_helper'

describe "Static pages" do

  describe "Home page" do

    it "should have the content 'Sample App'" do
      visit '/static_pages/home'
      expect(page).to have_content('Sample App')
    end

    it "should have the title 'Home'" do
      visit '/static_pages/home'
      expect(page).to have_title("Ruby on Rails Tutorial Sample App | Home")
    end
  end

  describe "Help page" do

    it "should have the content 'Help'" do
      visit '/static_pages/help'
      expect(page).to have_content('Help')
    end

    it "should have the title 'Help'" do
      visit '/static_pages/help'
      expect(page).to have_title("Ruby on Rails Tutorial Sample App | Help")
    end
  end

  describe "About page" do

    it "should have the content 'About Us'" do
      visit '/static_pages/about'
      expect(page).to have_content('About Us')
    end

    it "should have the title 'About Us'" do
      visit '/static_pages/about'
      expect(page).to have_title("Ruby on Rails Tutorial Sample App | About Us")
    end
  end
end
````

To resolve this problem add this to application.html.erb

````html.erb
<title>Ruby on Rails Tutorial Sample App | <%= yield(:title) %></title>
````
and to all views like sample shown for home

````html.erb
<% provide(:title, 'Home') %>
````    
We can use 

````console
rspec spec/requests/static_pages_spec.rb
````
or
````console
rspec spec
````
alone if our rvm version is 1.11.x or greater

###Automated tests with guard

First of all add guard to gem file
````ruby
 group :development, :test do
  ..........................
  ..........................
  gem 'guard-rspec', '2.5.0'
 end 
````
Then initialize Guard so that it works with RSpec:
````console
bundle exec guard init rspec
````
Now goto Guardfile and replace whole by
````ruby
require 'active_support/inflector'

# A sample Guardfile
# More info at https://github.com/guard/guard#readme

guard 'rspec' , all_after_pass: false do
  watch(%r{^spec/.+_spec\.rb$})
  watch(%r{^lib/(.+)\.rb$})     { |m| "spec/lib/#{m[1]}_spec.rb" }
  watch('spec/spec_helper.rb')  { "spec" }

  # Rails example
  watch(%r{^app/(.+)\.rb$})                           { |m| "spec/#{m[1]}_spec.rb" }
  watch(%r{^app/(.*)(\.erb|\.haml)$})                 { |m| "spec/#{m[1]}#{m[2]}_spec.rb" }
  watch(%r{^app/controllers/(.+)_(controller)\.rb$})  { |m| ["spec/routing/#{m[1]}_routing_spec.rb", "spec/#{m[2]}s/#{m[1]}_#{m[2]}_spec.rb", "spec/acceptance/#{m[1]}_spec.rb"] }
  watch(%r{^spec/support/(.+)\.rb$})                  { "spec" }
  watch('config/routes.rb')                           { "spec/routing" }
   # Custom Rails Tutorial specs
  watch(%r{^app/controllers/(.+)_(controller)\.rb$}) do |m|
    ["spec/routing/#{m[1]}_routing_spec.rb",
     "spec/#{m[2]}s/#{m[1]}_#{m[2]}_spec.rb",
     "spec/acceptance/#{m[1]}_spec.rb",
     (m[1][/_pages/] ? "spec/requests/#{m[1]}_spec.rb" :
                       "spec/requests/#{m[1].singularize}_pages_spec.rb")]
  end
  watch(%r{^app/views/(.+)/}) do |m|
    (m[1][/_pages/] ? "spec/requests/#{m[1]}_spec.rb" :
                      "spec/requests/#{m[1].singularize}_pages_spec.rb")
  end
  watch(%r{^app/controllers/sessions_controller\.rb$}) do |m|
    "spec/requests/authentication_pages_spec.rb"
  end
  
  watch('app/controllers/application_controller.rb')  { "spec/controllers" }

  # Capybara features specs
  watch(%r{^app/views/(.+)/.*\.(erb|haml)$})          { |m| "spec/features/#{m[1]}_spec.rb" }

  # Turnip features and steps
  watch(%r{^spec/acceptance/(.+)\.feature$})
  watch(%r{^spec/acceptance/steps/(.+)_steps\.rb$})   { |m| Dir[File.join("**/#{m[1]}.feature")][0] || 'spec/acceptance' }
end


````

To test use

````console
guard
````
and if you are using older version
````console
 bundle exec guard
````



