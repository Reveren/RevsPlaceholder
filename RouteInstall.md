# EXAMPLE
EXAMPLE OF MY CURRENT PROJECT

## Installation

First we'll create a new project

```console
rails new example
```

cd into directory

```console
cd example
```

Now we need to change the Gemfile a little to load some of our first add-ons. These are a collection of some of my most used and future to be used addons. Note that the versions might change from the original Gemfile and the pasted text below. (more specifically, take note of rails version)

```ruby
source 'https://rubygems.org'
ruby '2.2.3'
gem 'rails', '4.2.5'
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.1.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 2.0'

group :development do
  gem 'sqlite3'
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
group :test, :production do
  gem 'mysql2', '< 0.4.0'
end

gem 'bootstrap-sass'
gem 'font-awesome-rails'
gem 'devise'
```

Now run bundle install.

```console
bundle install
```

During this setup, I'm only going to run `rake db-migrate` only once at the end. This will save some time, but will make error debugging a little bit more difficult during this setup. I highly recommend running `rake db:migrate` after each instance of running `rails generate` or the shortcut version `rails g`.

## Days

Create the model for the days. 

```console
rails g model Day name:string abbrev:string
```

## Weeks

Create the model for the weeks. 

```console
rails g model Week description:string abbrev:string
```

## States

Create the model for the days. 

```console
rails g model State name:string abbrev:string
```

## Icons

Create the model for the icons. 

```console
rails g model Icon name:string fontawesome:string bootstrap:string
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

Build the views so you can customize the templates later (in our views/users folder).

```console
rails generate devise:views users
```

Add username to Users, since it is not in there by default with Devise

```console
rails generate migration add_username_to_users username:string:uniq
```

Run rake db:migrate

```console
rake db:migrate
```

Modify `application_controller.rb` and add username, email, password, password confirmation and remember me to `configure_permitted_parameters`

```ruby
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) { |u| u.permit(:username, :email, :password, :password_confirmation, :remember_me) }
    devise_parameter_sanitizer.for(:sign_in) { |u| u.permit(:login, :username, :email, :password, :remember_me) }
    devise_parameter_sanitizer.for(:account_update) { |u| u.permit(:username, :email, :password, :password_confirmation, :current_password) }
  end
end
```

Tell Devise to use `:login` in the `authentication_keys`

Modify `config/initializers/devise.rb` to have:

```ruby
config.authentication_keys = [ :login ]
```

Modify `config/initializers/devise.rb` file to have

```ruby
config.scoped_views = true
```

Overwrite Devise's `find_for_database_authentication` method in `User` model as well as be sure to add case **insensitivity** to your validations on `:username`, and add validations. Basically your user.rb should look like this at this point:

```ruby
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  # Virtual attribute for authenticating by either username or email
  # This is in addition to a real persisted field like 'username'
  attr_accessor :login

  validates :username,
    :presence => true,
    :uniqueness => {
      :case_sensitive => false
    }

  validate :validate_username
  
  def validate_username
    if User.where(email: username).exists?
      errors.add(:username, :invalid)
    end
  end

  def self.find_for_database_authentication(warden_conditions)
    conditions = warden_conditions.dup
    if login = conditions.delete(:login)
      where(conditions.to_hash).where(["lower(username) = :value OR lower(email) = :value", { :value => login.downcase }]).first
    elsif conditions.has_key?(:username) || conditions.has_key?(:email)
      conditions[:email].downcase! if conditions[:email]
      where(conditions.to_hash).first
    end
  end

end
```

*Modify the views*

sessions/new.html.erb:

```console
  -  <p><%= f.label :email %><br />
  -  <%= f.email_field :email %></p>
  +  <p><%= f.label :login %><br />
  +  <%= f.text_field :login %></p>
```

registrations/new.html.erb:

```console
  +  <p><%= f.label :username %><br />
  +  <%= f.text_field :username %></p>
     <p><%= f.label :email %><br />
     <%= f.email_field :email %></p>
```

registrations/edit.html.erb:

```console
  +  <p><%= f.label :username %><br />
  +  <%= f.text_field :username %></p>
     <p><%= f.label :email %><br />
     <%= f.email_field :email %></p>
```



## Company

Let's use a scaffold to save some time. We will build the Company controller first. 

```console
rails g scaffold Company user_id:integer name address city state_id:integer zip:integer notes:text
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

      t.timestamps null: false
    end
    add_foreign_key :companies, :users, column: :user_id, primary_key: :id, on_update: :cascade, on_delete: :cascade
  end
end
```

## Contacts

Now let's build the Contacts.

```console
rails g scaffold Contact user_id:integer company_id:integer first_name last_name title phone alt_phone email notes:text
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
      t.text :notes

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
rails g scaffold Stop user_id:integer company_id:integer contact_id:integer day_id:integer week_id:integer notes:text active:boolean priority:integer
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
      t.integer :week
      t.text :notes
      t.boolean :active

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

**Day**

```ruby
  has_many :stops
```

**Week**

```ruby
  has_many :stops
```

**State**

```ruby
  has_one :company
```

**Icon**

```ruby
  has_many :updates
```

**User**

```ruby
  devise :database_authenticatable, :registerable, :recoverable, :rememberable, :trackable, :validatable 
  
  has_many :companies
  has_many :contacts
  has_many :stops
  has_many :updates
```

**Company**

```ruby
  belongs_to :user
  belongs_to :state
```

**Contact**

```ruby
  has_many :stops, dependent: :destroy
  belongs_to :user
  belongs_to :company
```

**Stop**

```ruby
  belongs_to :user
  belongs_to :company
  belongs_to :contact
  belongs_to :day
  belongs_to :week
 ```

**Update**

```ruby
  belongs_to :user
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
  @companies = Company.all.where(user_id: current_user)
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
  @contacts = Contact.all.where(user_id: current_user)
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
  @stops = Stop.all.where(user_id: current_user)
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
  @updates = Update.all.where(user_id: current_user)
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
# This file should contain all the record creation needed to seed the database with its default values.
# The data can then be loaded with the rake db:seed (or created alongside the db with db:setup).
#
# Examples:
#
#   cities = City.create([{ name: 'Chicago' }, { name: 'Copenhagen' }])
#   Mayor.create(name: 'Emanuel', city: cities.first)

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


# ### Days of the week ####################
# This is to set up the days of the week
# #########################################

week_list = [
  ['Every Week', 'Weekly'],
  ['Every Odd Week', 'Odd week'],
  ['Every Even Week', 'Even week'],
  ['Every Three Weeks', '3 weeks']
]

week_list.each do |description, abbrev|
  Week.create( description: description, abbrev: abbrev )
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
  ['Twitter', '1355 Market Street', 'San Francisco', '5', '94103', '', '1'],
  ['Apple', '1 Infinite Loop', 'Cupertino', '5', '95014', '', '1'],
  ['Facebook', '1 Hacker Way', 'Menlo Park', '5', '94025', '', '1'],
  ['Google', '1600 Amphitheatre Parkway', 'Mountain View', '5', '94043', '', '1'],
  ['Microsoft', 'One Microsoft Way', 'Redmond', '48', '98052', '', '1']
]

company_list.each do |name, address, city, state_id, zip, notes, user_id|
  Company.create( name: name, address: address,  city: city, state_id: state_id, zip: zip, notes: notes, user_id: user_id)
end

# ############## Contacts  ################
# This is to set up data to test with.   
# Make sure companies are setup first.
# #########################################

contact_list = [
  ['1','1','Jack','Dorsey','CEO','(415) 222-9670','','',''],
  ['1','2','Tim','Cook','CEO','(800) 692–7753','','',''],
  ['1','3','Mark','Zuckerberg','CEO','(650) 543-4800','','',''],
  ['1','4','Sundar','Pichai','CEO','(650) 253-0000','','',''],
  ['1','5','Satya','Nadella','CEO','(425) 882-8080','','',''],
]

contact_list.each do |user_id, company_id, first_name, last_name, title, phone, alt_phone, email, notes|
  Contact.create(user_id: user_id, company_id: company_id, first_name: first_name, last_name: last_name, title: title, phone: phone, alt_phone: alt_phone, email: email, notes: notes)
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
