# LDAP-Rails
This is a demo project and instructions on how to authenticate with an LDAP (Active Directory) server using the Devise gem at a very basic level. 

### Getting Started ###
To run the demo, run through the following.

#### Step 1 ####
```sh
git clone https://github.com/JaisonBrooks/LDAP-Rails.git
```

#### Step 2 ####
Add your LDAP SERVER and PORT in the ```config/secrets.yml```

```yaml
defaults: &defaults
  ldap_server: YOUR_SERVER
  ldap_port: YOUR_PORT
```

#### Step 3 ####
Rake the DB
```sh
rake db:migrate
```

#### Step 4 ####
Run your server and enjoy
```sh
rails s
```
___
## Add LDAP to your Application ##
These steps will show you how you can add basic LDAP authentication to your application.

#### Step 1 ####
Add Devise and Net-Ldap to your Gemfile.

```ruby
gem 'devise'
gem 'net-ldap'
```

#### Step 2 ####
Install and setup Devise, see the following link for more on how to use Devise.

```sh
# Install Devise
rails g devise:install

# Setup User
rails g devise user

# Build Views
rails g devise:views

```

#### Step 3 ####
Add the following to the ```config/secrets.yml``` file under each environement or under defaults.

```yaml
ldap_server: YOUR_SERVER
ldap_port: YOUR_PORT
```

#### Step 4 ####
Create an the following file ```config/initializers/ldap_authenticatable.rb``` and insert the following code:

```ruby
require 'net/ldap'
require 'devise/strategies/authenticatable'

module Devise
  module Strategies
    class LdapAuthenticatable < Authenticatable
      def authenticate!
        if params[:user]
          ldap = Net::LDAP.new
          ldap.host = Rails.application.secrets.ldap_server
          ldap.port = Rails.application.secrets.ldap_port
          ldap.auth email, password

          if ldap.bind
            user = User.find_or_create_by(email: email)
            success!(user)
          else
            fail(:invalid_login)
          end
        end
      end

      def email
        params[:user][:email]
      end

      def password
        params[:user][:password]
      end

    end
  end
end

Warden::Strategies.add(:ldap_authenticatable, Devise::Strategies::LdapAuthenticatable)
```

#### Step 5 ####
Modify the ```config/initializers/devise.rb``` and add the following:

```ruby
Devise.setup do |config|
  
  config.warden do |manager|
      manager.default_strategies(:scope => :user).unshift :ldap_authenticatable
    end
    
  # The rest of your devise config...
  
end
```

#### Step 6 ####
Add the following to the controller(s) you want only authenticated users to access. You can add to the ```application_controller.rb``` if you want the entire application only accessible to authenticated users (excep the sign in page).

```ruby
before_action :authenticate_user!
```

#### Step 7 ####
Rake the db and start the server

```sh
rake db:migrate

rails s
```

#### Finally ####
You should be able to authenticate with using your LDAP active directory creds.

This LDAP Authentication is very basic and your LDAP server may require further configruation in your Rails app to properly authenticate with it. Checkout the Net-LDAP documentation for more information. There are plenty more customizations you can do from here. Since Devise comes with alot of standard features, there are plenty of features you dont need when using LDAP as your source for authentication. I'd suggest your read up on the Devise gem and learn how about how to disable features that you wont be using.

#### Hope this helps :) ####
