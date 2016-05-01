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

ruby '2.3.0'
gem 'rails', '4.2.6'
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.1.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 2.0'
gem 'sdoc', '~> 0.4.0', group: :doc

group :development do
  gem 'web-console', '~> 2.0'
  gem 'spring'
  gem 'better_errors'
  gem 'quiet_assets'
  gem 'binding_of_caller', '~> 0.7.2'
end

group :development, :test do
  gem 'sqlite3'
  gem 'byebug'
end

group :test do
#  gem 'capybara'
end

group :production do
  gem 'pg'
  gem 'rails_12factor'
  gem 'puma'
end

# for bootstrap 4 tooltips and popovers
source 'https://rails-assets.org' do
  gem 'rails-assets-tether', '>= 1.1.0'
end

gem 'jquery-turbolinks'
gem 'bootstrap', '~> 4.0.0.alpha3'
gem 'font-awesome-rails'
gem 'jquery-ui-rails'
gem 'devise'
# gem 'pundit'
# gem 'awesome_nested_set'
# gem "the_sortable_tree", "~> 2.5.0"
# gem 'ransack'
# gem 'local_time'
# gem 'sucker_punch'
# gem 'stripe'
# gem 'stripe_event'
# gem 'roadie'
# gem 'gibbon', '~> 2.0'
# gem 'paperclip'
# gem 'aws-sdk', '< 2.0'
# gem 'font_assets'
# gem 'will_paginate'
# gem 'cocoon'
# gem 'gmaps4rails'
# gem 'geocoder'
# gem "recaptcha", require: "recaptcha/rails"
# gem 'friendly_id'
# gem 'ckeditor_rails'
```

Now run bundle install.

```console
bundle install
```

Add some tables for fixed values like US States:

**Days**

Create the model for the days. 

```console
rails g model Day name:string abbrev:string
rake db:migrate
```

**Weeks**

Create the model for the weeks. 

```console
rails g model Week description:string abbrev:string
rake db:migrate
```

**States**

Create the model for the days. 

```console
rails g model State name:string abbrev:string
rake db:migrate
```

**Icons**

Create the model for the icons. 

```console
rails g model Icon name:string fontawesome:string bootstrap:string
rake db:migrate
```

## Devise

If Devise was already added to your Gemfile from the first step above, you need to start by running the generator:

```console
rails generate devise:install
```

The generator will install an initializer which describes ALL of Devise's configuration options. It is *imperative* that you take a look at it. When you are done, you are ready to add Devise to any of your models using the generator:

```console
rails generate devise User username:string:uniq role_id:integer mailchimp:boolean
```

Add the role table for user roles:

```console
rails g model Role name:string
```

Build the views so you can customize the templates later (in our views/users folder).

```console
rails generate devise:views users
```

Run rake db:migrate

```console
rake db:migrate
```

Add a file `registrations_controller.rb` and add the following to it:

```ruby
class RegistrationsController < Devise::RegistrationsController
  
  prepend_before_action :check_captcha, only: [:create]
  
  private

    def sign_up_params
      params.require(:user).permit(:username, :email, :mailchimp, :password, :password_confirmation)
    end
    
    def sign_in_params
      params.require(:user).permit(:username, :email, :login, :password, :password_confirmation, :remember_me)
    end
  
    def account_update_params
      params.require(:user).permit(:username, :email, :mailchimp, :password, :password_confirmation, :current_password)
    end

    def check_captcha
      if verify_recaptcha
        true
      else
        self.resource = resource_class.new sign_up_params
        respond_with_navigational(resource) { render :new }
      end 
    end

end
```

**Routes**

Tell routes that this new controller exists by changing `devise_for :users` to this:

```ruby
devise_for :users, :controllers => { registrations: 'registrations' }
```

**Initializer**

Tell Devise to use `:login` in the `authentication_keys` by modifying the Initializer:

Modify `config/initializers/devise.rb` on line **34** to have:

```ruby
config.authentication_keys = [ :login ]
```

Modify `config/initializers/devise.rb` on line **214** to have:

```ruby
config.scoped_views = true
```

**User Model**

In the `User` model, overwrite Devise's `find_for_database_authentication` method. Also, be sure to add case *insensitivity* to your validations on `:username` and any other validations you see fit.

Basically your `user.rb` should look like this now:

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

**Config**

You need to set up the default URL options for the Devise mailer in each environment. Here is a possible configuration for `config/environments/development.rb`:

```ruby
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

**Views**

***sessions/new.html.erb:***

Change this:

```html
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true %>
  </div>
```

to this:

```html
  <div class="field">
    <%= f.label :login %><br />
    <%= f.text_field :login, autofocus: true %></p>
  </div>
```

***registrations/new.html.erb:***

Add this:

```ruby
  <div class="field">
    <%= f.label :username %><br />
    <%= f.text_field :username, autofocus: true %>
  </div>
```

***registrations/edit.html.erb:***

Add this:

```ruby
  <div class="field">
    <%= f.label :username %><br />
    <%= f.text_field :username, autofocus: true %>
  </div>
```


## Homepages

Now let's build the two homepages.

```console
rails generate controller Home index welcome
```

## Routes

Adjust your `routes.rb` file to make a landing page. Also remove the notes in there. 

```ruby
Rails.application.routes.draw do

  devise_for :users
  
  authenticated do
    root :to => 'home#index', as: :authenticated
  end
  
  root :to => 'home#welcome'

end
```


## Models

These will need to be adjusted to match your models and relationships. 

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
  has_many :tupdates
```

**User**

```ruby
  has_many :companies
  has_many :contacts
  has_many :stops
  has_many :tupdates
```

## Controllers

### home_controller.rb

Change the layout of the welcome page so it does not contain the navbar.

```ruby
  def welcome
    render layout: "empty"
  end
```

### Duplicate `application.html.erb`

Now in the `views/layout` folder, duplicate the `application.html.erb` file to `empty.html.erb`

## Boostrap

### CSS 
Gonna be using Bootstrap 4 which at this writing is still in alpha. 

Add this to your Gemfile:

```ruby
# for bootstrap 4 tooltips and popovers
source 'https://rails-assets.org' do
  gem 'rails-assets-tether', '>= 1.1.0'
end
```

Add the gem and run `bundle install`.

```ruby
gem 'bootstrap', '~> 4.0.0.alpha3'
gem 'font-awesome-rails'
```

```console
bundle install
```

Remove the scaffold.css from your css in `app/assets/stylesheets/scaffold.css`.

In the same folder, change the rename `application.css` to `application.scss` by adding `s` to it.

Open `app/assets/stylesheets/application.scss` and import Bootstrap styles.

```scss
@import "bootstrap";
@import "font-awesome";
```

`bootstrap-sprockets` is no longer needed with version 4.

**Important**

Remove all the `*= require_self` and `*= require_tree .` statements from the file. (We use `@import` to import Sass files instead)

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


# ### Weekly etc. ####################

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

# ############## Icons ####################

icon_list = [
  ['Over Coffee', 'fa fa-coffee', ''],
  ['Over Lunch', 'fa fa-cutlery', ''],
  ['Over Drinks', 'fa fa-glass', ''],
  ['In-Person', 'fa fa-user', ''],
  ['Over the Phone', 'fa fa-phone', ''],
  ['Via Email', 'fa fa-envelope-o', '']
]

icon_list.each do |name, fontawesome, bootstrap|
  Icon.create( name: name, fontawesome: fontawesome, bootstrap: bootstrap )
end

# ############## Users ####################

user_list = [
  ['user@example.com', 'password', 'password', 'admin']
]

user_list.each do |email, password, password_confirmation, username|
  User.create( email: email, password: password,  password_confirmation: password_confirmation, username: username)
end
```

## Migrate

We should run `rake db:migrate` again to make sure everythign has run and to see if we have any errors. 

```console
rake db:migrate
```

Now run the seed file:

```console
rake db:seed
```

You should now have a login to use to log into the site with. 

Username: `user@example.com`

Password: `password`
