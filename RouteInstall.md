# routy
Route your day

## Installation

First we'll create a new project

```console
rails new route
```

cd into directory

```console
cd route
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

```console
bundle install
```

During this setup, I'm only going to run `rake db-migrate` only once at the end. This will save some time, but will make error debugging a little bit more difficult further down the road if you missed something. 

## Days

Create the model for the days. 

```console
rails g model Day name:string abbrev:string
```

## States

Create the model for the days. 

```console
rails g model State name:string abbrev:string
```

## Icons

Create the model for the icons. 

```console
rails g model Icon description:string fontawesome:string bootstrap:string example:attachment
```

## Devise

If Devise was already added to your Gemfile from the first step above, you need to start by running the generator:

```console
rails generate devise:install
```

The generator will install an initializer which describes ALL of Devise's configuration options. It is *imperative* that you take a look at it. When you are done, you are ready to add Devise to any of your models using the generator:

```console
rails generate devise User
```

Build the views so you can customize the templates later.

```console
rails g devise:views
```

## Company

Let's use a scaffold to save some time. We will build the Company controller first. 

```console
rails g scaffold Company name address city state_id:integer zip:integer notes:text active:boolean
```

Add foreign keys to migration.

```ruby
class CreateCompanies < ActiveRecord::Migration
  def change
    create_table :companies do |t|
      t.integer :user_id
      t.string :name
      t.string :address
      t.string :city
      t.integer :state_id
      t.integer :zip
      t.text :notes
      t.boolean :active

      t.timestamps null: false
    end
    add_foreign_key :companies, :users, column: :user_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
  end
end
```

## Contacts

Now let's build the Contacts.

```console
rails g scaffold Contact first_name last_name title phone alt_phone email notes:text active:boolean
```

Add foreign keys to migration.

```ruby
class CreateContacts < ActiveRecord::Migration
  def change
    create_table :contacts do |t|
      t.integer :user_id
      t.integer :company_id
      t.string :first_name
      t.string :last_name
      t.string :title
      t.string :phone
      t.string :alt_phone
      t.string :email
      t.boolean :active

      t.timestamps null: false
    end
    add_foreign_key :contacts, :users,     column: :user_id,    primary_key: :id, on_update: :cascade, on_delete: :cascade
    add_foreign_key :contacts, :companies, column: :company_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
  end
end
```


## Stops

Now let's build the Stops.

```console
rails g scaffold Stop user_id:integer company_id:integer contact_id:integer day_id:integer active:boolean week notes:text
```

Add foreign keys to migration.

```ruby
class CreateStops < ActiveRecord::Migration
  def change
    create_table :stops do |t|
      t.integer :user_id
      t.integer :company_id
      t.integer :contact_id
      t.integer :day_id
      t.boolean :active
      t.string :week
      t.text :notes

      t.timestamps null: false
    end
    add_foreign_key :stops, :users,     column: :user_id,    primary_key: :id, on_update: :cascade, on_delete: :cascade
    add_foreign_key :stops, :companies, column: :company_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
    add_foreign_key :stops, :contacts,  column: :contact_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
    add_foreign_key :stops, :days,      column: :day_id,     primary_key: :id, on_update: :cascade, on_delete: :cascade
  end
end
```

## Timeline Updates

Now let's build the Timeline Updates.

```console
rails g scaffold Update user_id:integer stop_id:integer icon_id:integer notes:text
```

Add foreign keys to migration.

```ruby
class CreateUpdates < ActiveRecord::Migration
  def change
    create_table :updates do |t|
      t.integer :user_id
      t.integer :stop_id
      t.integer :icon_id
      t.text :notes

      t.timestamps null: false
    end
    add_foreign_key :updates, :users, column: :user_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
    add_foreign_key :updates, :stops, column: :stop_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
    add_foreign_key :updates, :icons, column: :icon_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
  end
end
```

## Routes

Adjust your `routes.rb` file to make a landing page. Also remove the notes in there. 

```ruby
root to: "companies#index"
```

## Config

You need to set up the default URL options for the Devise mailer in each environment. Here is a possible configuration for `config/environments/development.rb`:

```ruby
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

## Models

Now let's adjust our models to compensate for our foreign keys. 

**State**

```ruby
  has_one :company
```

**Day**

```ruby
  has_many :stops
```

**Icon**

```ruby
  has_many :updates
```

**User**

```ruby
  devise :database_authenticatable, :registerable, :recoverable, :rememberable, :trackable, :validatable 
  has_many :companies
```

**Company**

```ruby
  belongs_to :user
  belongs_to :state
```

**Contact**

```ruby
  has_many :stops, dependent: :destroy
  belongs_to :company
```

**Stop**

```ruby
  belongs_to :user
  belongs_to :contact
  belongs_to :day
  belongs_to :company
```

**Update**

```ruby
  belongs_to :company
  belongs_to :icon
```

## Controllers

### companies_controller.rb

Make sure only logged-in users can see the content.

```ruby
before_action :authenticate_user!
```

Make sure all content that is created is attributed to the person who created it by adding `current_user`:

Change this line: 

```ruby
def create
  @company = Company.new(company_params)
```

to this:

```ruby
def create
  @company = current_user.companies.new(company_params)
```

We also need to make sure the logged-in user can only see what they created.

Change the index definition from this:

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

### contacts_controller.rb

Make sure only logged-in users can see the content.

```ruby
before_action :authenticate_user!
```

Make sure all content that is created is attributed to the person who created it by adding `current_user`:

Change this line: 

```ruby
def create
  @contact = Contact.new(contact_params)
```

to this:

```ruby
def create
  @contact = current_user.contacts.new(contact_params)
```

We also need to make sure the logged-in user can only see what they created.

Change the index definition from this:

```ruby
def index
  @contacts = Contact.all
end
```

to this:

```ruby
def index
  @contacts = current_user.contacts.all
end
```

### stops_controller.rb

Make sure only logged-in users can see the content.

```ruby
before_action :authenticate_user!
```

Make sure all content that is created is attributed to the person who created it by adding `current_user`:

Change this line: 

```ruby
def create
  @stop = Stop.new(stop_params)
```

to this:

```ruby
def create
  @stop = current_user.stops.new(stop_params)
```

We also need to make sure the logged-in user can only see what they created.

Change the index definition from this:

```ruby
def index
  @stops = Stop.all
end
```

to this:

```ruby
def index
  @stops = current_user.stops.all
end
```

### updates_controller.rb

Make sure only logged-in users can see the content.

```ruby
before_action :authenticate_user!
```

Make sure all content that is created is attributed to the person who created it by adding `current_user`:

Change this line: 

```ruby
def create
  @update = Update.new(update_params)
```

to this:

```ruby
def create
  @update = current_user.updates.new(update_params)
```

We also need to make sure the logged-in user can only see what they created.

Change the index definition from this:

```ruby
def index
  @updates = Update.all
end
```

to this:

```ruby
def index
  @updates = current_user.updates.all
end
```

## Boostrap

### CSS 

Make sure Bootstrap and Font Awesome were added and uncommented in our Gemfile. If not, do that and run `bundle install`.

```ruby
gem 'bootstrap-sass'
gem 'font-awesome-rails'
```

Remove the scaffold.css from your css in `app/assets/stylesheets/scaffold.css`.

In the same folder, change the rename `application.css` to `application.css.scss` by adding `scss` to the end of it.

Open `app/assets/stylesheets/application.css.scss` and import Bootstrap styles.

```scss
@import "bootstrap-sprockets";
@import "bootstrap";
@import "font-awesome";
```

`bootstrap-sprockets` must be imported before `bootstrap` for the icon fonts to work.

Remove all the `*= require_self` and `*= require_tree .` statements from the file. (We use `@import` to import Sass files instead)

**Remember, do not use** `*= require` **in Sass or your other stylesheets will not be able to access the Bootstrap mixins or variables.**

### JS

Require Bootstrap Javascripts in `app/assets/javascripts/application.js`. Make sure it goes ***after*** `//= require jquery`. Usually it goes after `//= require turbolinks`. The last five lines should look like this:

```js
//= require jquery
//= require jquery_ujs
//= require turbolinks
//= require bootstrap
//= require_tree .
```

### Template

Put the notices in the template in `app/views/layouts/application.html.erb` just above the `<%= yield %>`.

```html
<% if notice %><div class="alert alert-info"><%= notice %></div><% end %>
<% if alert %><div class="alert alert-warning"><%= alert %></div><% end %>
```

While we're at it, put the Bootstrap container in place to keep content from hugging edge. Replace this line:

```html
<%= yield %>
```

with this line:

```html
<div class="container">
  <%= yield %>
</div>
```


We're almost ready to migrate, but first you need to fill in your seed file with some data so we can run db:seed right after we run db:migrate.



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
  ['user@example.com', 'admin', 'admin']
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

## Migrate

We're finally ready to create the database and migrate it to see if we have any errors. 

```console
rake db:migrate
```

Now run the seed file:

```console
rake db:seed
```

You should now have a login to use to log into the site with. 

Username: `user@example.com`

Password: `admin`
