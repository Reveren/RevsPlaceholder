# routy
Route your day

## Installation

First we'll create a new project

```ruby
rails new route
```

Now we need to change the Gemfile a little to load some of our first add-ons. These are a collection of some of my most used and future to be used addons. Note that the versions might change from the original Gemfile and the pasted text below. (more specifically, take note of rails version)

```ruby
source 'https://rubygems.org'
ruby '2.2.3'
gem 'rails', '4.2.5'
gem 'sqlite3'
# gem 'mysql2', '< 0.4.0'
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.1.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 2.0'


group :development do
  gem 'web-console', '~> 2.0'
  gem 'spring'
  gem 'better_errors'
  gem 'quiet_assets'
  gem 'rails_layout'
  gem 'spring-commands-rspec'
  gem 'binding_of_caller', '~> 0.7.2'
  gem 'annotate', '~> 2.6', '>= 2.6.10'
end
group :development, :test do
  gem 'byebug'
  gem 'factory_girl_rails'
  gem 'faker'
  gem 'rspec-rails'
end
group :test do
  gem 'capybara'
  gem 'database_cleaner'
  gem 'launchy'
  gem 'selenium-webdriver'
end

gem 'bootstrap-sass'
gem 'font-awesome-rails'
gem 'devise'
# gem 'growlyflash'
# gem 'paperclip'
# gem 'groupdate'
# 
# gem 'friendly_id'
# gem 'kaminari'
# gem 'bootstrap-kaminari-views'
# gem 'administrate'
```

Now run bundle install.

```ruby
bundle install
```

Create the database by running migrate.

```ruby
rake db:migrate
```

Let's create the models for the days and the states. 

```ruby
rails g model Day name:string abbrev:string
rails g model State name:string abbrev:string
```

Before we run migrate, let's add some limits to the migration. You will need to open and modify both migrations that we just created. They can be found in:
`db / migrate / XXXXX_create_days.rb` 
`db / migrate / XXXXX_create_states.rb` 

We will add a limit of 25 to the names and 2 or 3 to the abbreviations like so:

**Days:**

```ruby
class CreateDays < ActiveRecord::Migration
  def change
    create_table :days do |t|
      t.string :name, :limit => 25
      t.string :abbrev, :limit => 3

      t.timestamps null: false
    end
  end
end
```

**States:**

```ruby
class CreateStates < ActiveRecord::Migration
  def change
    create_table :states do |t|
      t.string :name, :limit => 25
      t.string :abbrev, :limit => 2

      t.timestamps null: false
    end
  end
end
```

Now we can run the migrate.

```ruby
rake db:migrate
```

## Company

Let's use a scaffold to save some time. We will build the Company controller first. 

```ruby
rails g scaffold Company name address city state_id:integer zip:integer notes:text active:boolean
```

```ruby
rake db:migrate
```

Adjust your `routes.rb` file to make a landing page. Also remove the notes in there. 

```ruby
root to: "companies#index"
```

## Boostrap

Let's add Bootstrap and Font Awesome to our Gemfile.

```ruby
gem 'bootstrap-sass'
gem 'font-awesome-rails'
```

Remove the scaffold.css from your css in `app/assets/stylesheets/scaffold.css`.

In the same folder, change the rename `application.css` to `application.css.scss` by adding `scss` to the end of it.

Import Bootstrap styles in `app/assets/stylesheets/application.css.scss`.

```scss
@import "bootstrap-sprockets";
@import "bootstrap";
@import "font-awesome";
```

`bootstrap-sprockets` must be imported before `bootstrap` for the icon fonts to work.

Also, again make sure the file has `.scss` extension. If you have just generated a new Rails app, it may come with a `.css` file instead. If this file exists, it will be served instead of Sass.

Then, remove all the `*= require_self` and `*= require_tree .` statements from the sass file. Instead, we use `@import` to import Sass files.

**Do not use** `*= require` **in Sass or your other stylesheets will not be able to access the Bootstrap mixins or variables.**

Require Bootstrap Javascripts in `app/assets/javascripts/application.js`. Make sure it goes ***after*** `//= require jquery`. Usually it goes after `//= require turbolinks`.

```js
//= require bootstrap
```

## Devise

Devise 4.0 works with Rails 4.2 onwards. You can add it to your Gemfile with:

```ruby
gem 'devise'
```

Run the bundle command to install it.

After you install Devise and add it to your Gemfile, you need to run the generator:

```console
rails generate devise:install
```

The generator will install an initializer which describes ALL of Devise's configuration options. It is *imperative* that you take a look at it. When you are done, you are ready to add Devise to any of your models using the generator:

```console
rails generate devise User
```

Next, check the User MODEL for any additional configuration options you might want to add, such as confirmable or lockable. For example, if you add the confirmable option in the model, you'll need to uncomment the Confirmable section in the migration. Then run `rake db:migrate`

Next, you need to set up the default URL options for the Devise mailer in each environment. Here is a possible configuration for `config/environments/development.rb`:

```ruby
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

You should restart your application (`ctrl-c` and then `rails s`) after changing Devise's configuration options. Otherwise, you will run into strange errors, for example, users being unable to login and route helpers being undefined.

Put the notices in the tempalte in `app/views/layouts/application.html.erb` just above the `<%= yield %>`.

```
<% if notice %><div class="alert alert-info"><%= notice %></div><% end %>
<% if alert %><div class="alert alert-warning"><%= alert %></div><% end %>
```

While we're at it, we might as well put the Bootstrap container in place. Replace this line:

```ruby
<%= yield %>
```

with this line:

```ruby
<div class="container">
  <%= yield %>
</div>
```

Build the views so you can customize the templates.

```ruby
rails g devise:views
```

Now let's add the user_id to companies. 

```ruby
rails g migration AddUserIdToCompanies user_id:integer
```

Before we run migrate, we need to adjsut the migration to add the foreign keys. We will use the old up and down format. Your new migration should look like this. 

```ruby
class AddUserIdToCompanies < ActiveRecord::Migration
  def up
    add_column :companies, :user_id, :integer
    add_foreign_key :companies, :users, column: :user_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
  end
  
  def down
    remove_foreign_key :companies, column: :user_id
    remove_column :companies, :user_id
  end
end
```

Now we canrun migrate.

```ruby
rake db:migrate
```

Now let's adjust our models. 

**User Model**

```ruby
has_many :companies
```

**Company Model**

```ruby
belongs_to :user
```

### Change the Controller
We can now add the logged in user to company creation by changing this line:

```ruby
def create
  @company = Company.new(company_params)
```

to this:

```ruby
def create
  @company = current_user.companies.new(company_params)
```

At this point the server should start and you hsould be able to see the basic framework, but we need to make sure those who are logged in only see what they created. The change is similar to the one above, but it will break the site until the first user is created. We will add the user to the db:seed file and then run it. 

First, we'll finish up with the controller. Add this to the top just under the first `before_action`:
```ruby
before_action :authenticate_user!
```

Now change the index definition from this:

```ruby
def index
  @companies = Company.all
end
```

to this:

```ruby
def index
  @companies = current_user.companies.all
end
```

## Seeds.rb

Open the seed file in `db/seeds.rb` and paste this info. 
 
 ```ruby
# ### Days of the week ####################
# This is to set up the days of the week
# #########################################

  
  
day_list = [
  ['Sunday', 'Sun'],
  ['Monday', 'Mon'],
  ['Tuesday', 'Tue'],
  ['Wednesday', 'Wed'],
  ['Thursday', 'Thu'],
  ['Friday', 'Fri'],
  ['Saturday', 'Sat']
]

day_list.each do |name, abbrev|
  Day.create( name: name, abbrev: abbrev )
end

# ### US States ###########################
# This is to set up all the US states  
# #########################################

state_list = [
	['Alaska','AK'],
	['Alabama','AL'],
	['Arizona','AZ'],
	['Arkansas','AR'],
	['California','CA'],
	['Colorado','CO'],
	['Connecticut','CT'],
	['Delaware','DE'],
	['District of Columbia','DC'],
	['Florida','FL'],
	['Georgia','GA'],
	['Hawaii','HI'],
	['Idaho','ID'],
	['Illinois','IL'],
	['Indiana','IN'],
	['Iowa','IA'],
	['Kansas','KS'],
	['Kentucky','KY'],
	['Louisiana','LA'],
	['Maine','ME'],
	['Maryland','MD'],
	['Massachusetts','MA'],
	['Michigan','MI'],
	['Minnesota','MN'],
	['Mississippi','MS'],
	['Missouri','MO'],
	['Montana','MT'],
	['Nebraska','NE'],
	['Nevada','NV'],
	['New Hampshire','NH'],
	['New Jersey','NJ'],
	['New Mexico','NM'],
	['New York','NY'],
	['North Carolina','NC'],
	['North Dakota','ND'],
	['Ohio','OH'],
	['Oklahoma','OK'],
	['Oregon','OR'],
	['Pennsylvania','PA'],
	['Rhode Island','RI'],
	['South Carolina','SC'],
	['South Dakota','SD'],
	['Tennessee','TN'],
	['Texas','TX'],
	['Utah','UT'],
	['Vermont','VT'],
	['Virginia','VA'],
	['Washington','WA'],
	['West Virginia','WV'],
	['Wisconsin','WI'],
	['Wyoming','WY']
]

state_list.each do |name, abbrev|
  State.create( name: name, abbrev: abbrev )
end

# ############## Users ####################
# This is to set up data to test with.   
# Make sure the root user is set first.
# #########################################

user_list = [
  ['user@example.com', 'password', 'password']
]

user_list.each do |email, password, password_confirmation|
  User.create( email: email, password: password,  password_confirmation: password_confirmation)
end


# ############## Companies ################
# This is to set up data to test with.   
# Make sure the root user is setup first.
# #########################################

company_list = [
  ['Twitter', '1355 Market Street', 'San Francisco', '4', '94103', '', '1', '1'],
  ['Apple', '1 Infinite Loop', 'Cupertino', '4', '95014', '', '1', '1'],
  ['Facebook', '1 Hacker Way', 'Menlo Park', '4', '94025', '', '1', '1'],
  ['Google', '1600 Amphitheatre Parkway', 'Mountain View', '4', '94043', '', '1', '1'],
  ['LinkedIn', '2029 Stierlin Ct', 'Mountain View', '4', '94043', '', '0', '1']
]

company_list.each do |name, address, city, state_id, zip, notes, active, current_user|
  User.create( name: name, address: address,  city: city, state_id: state_id, zip: zip, notes: notes, active: active, current_user: current_user)
end
```

Now run the seed file:

```ruby
rake db:seed
```

You should now have a login to use to log into the site with. 
