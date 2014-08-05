##Sample App
Please note that I am using sublime text editor and terminal
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

###Speeding up test with spork

First of all add gem to gem file

````ruby
group :development, :test do
..........................
..........................
..........................
gem 'spork-rails', '4.0.0'
gem 'guard-spork', '1.5.0'
gem 'childprocess', '0.3.6'
end
````
If you find any problem while installing gem try this

````console
bundle update
````
Next, bootstrap the Spork configuration
````console
bundle exec spork --bootstrap
````
Now we need to edit the RSpec configuration file, located in spec/spec_helper.rb, so that the environment gets loaded in a prefork block, which arranges for it to be loaded only once 
````ruby
Spork.prefork do
# Loading more in this block will cause your tests to run faster. However,
# if you change any configuration or code from libraries loaded here, you'll
# need to restart spork for it take effect.
ENV["RAILS_ENV"] ||= 'test'
require File.expand_path("../../config/environment", __FILE__)
require 'rspec/rails'
require 'rspec/autorun'

# Requires supporting ruby files with custom matchers and macros, etc,
# in spec/support/ and its subdirectories.
Dir[Rails.root.join("spec/support/**/*.rb")].each {|f| require f}

# Checks for pending migrations before tests are run.
# If you are not using ActiveRecord, you can remove this line.
ActiveRecord::Migration.check_pending! if defined?(ActiveRecord::Migration)

RSpec.configure do |config|
# ## Mock Framework
#
# If you prefer to use mocha, flexmock or RR, uncomment the appropriate line:
#
# config.mock_with :mocha
# config.mock_with :flexmock
# config.mock_with :rr

# Remove this line if you're not using ActiveRecord or ActiveRecord fixtures
config.fixture_path = "#{::Rails.root}/spec/fixtures"

# If you're not using ActiveRecord, or you'd prefer not to run each of your
# examples within a transaction, remove the following line or assign false
# instead of true.
config.use_transactional_fixtures = true

# If true, the base class of anonymous controllers will be inferred
# automatically. This will be the default behavior in future versions of
# rspec-rails.
config.infer_base_class_for_anonymous_controllers = false

# Run specs in random order to surface order dependencies. If you find an
# order dependency and want to debug it, you can fix the order by providing
# the seed, which is printed after each run.
#     --seed 1234
config.order = "random"
config.include Capybara::DSL
end
end

Spork.each_run do
# This code will be run each time you run your specs.

end
````
Before running Spork, we can get a baseline for the testing overhead by timing our test suite as follows

````console
time bundle exec rspec spec/requests/static_pages_spec.rb
````
I got an error based on gem dependency.

if so run

````console
bundle install
````
Use it again and if no errors then start sprock server by

````console
bundle exec spork
````
for old version 

and

````console
spork
````
for new version 

We can use sprock with guard using

````console
guard init spork
````
or
````console
bundle exec guard init spork
````
Now add this to the end of Guardfile
````ruby
guard 'rspec',all_after_pass:false,cli:'--drb' do
end  
````
Now go and find sublime text editor packages directory using terminal and run command
````console
git clone https://github.com/maltize/sublime-text-2-ruby-tests.git RubyTest
````

Now restart your sublime text editor

then give this changes to Preferences > Package Settings > RubyTest > Settings - User

````JSON
{
  "check_for_rbenv": true,
  "check_for_rvm": true,
  "check_for_bundler": true
}
````
Now you will be able to run tests from sublime text editor itself

first goto your spec/static_pages_spec.rb and then
with 

`Ctrl+Shift+R` to run a single block of test
`Ctrl+Shift+E` to run last test
`Ctrl+Shift+T` to run all tests in a current file

Next add this lines to helpers/application_helper.rb

````ruby
 def full_title(page_title)
    base_title = "Ruby on Rails Tutorial Sample App"
    if page_title.empty?
      base_title
    else
      "#{base_title} | #{page_title}"
    end
  end
````
and to layouts/application.html.erb

````html.erb
<title><%= full_title yield(:title) %></title>
````










