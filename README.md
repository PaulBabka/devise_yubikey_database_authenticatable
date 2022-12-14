# Devise - Yubikey Database Authentication
   
[![Build Status](https://travis-ci.org/mort666/yubikey_database_authenticatable.png?branch=master)](https://travis-ci.org/mort666/yubikey_database_authenticatable)

NB: This repo was originally cloned from  mort666 / yubikey_database_authenticatable

The legacy repo was not being maintained, and the latest version of the GEM did not include the most recent features.

This extension to Devise adds a modified Database Authentication strategy to allow the authentication of a user with Two Factor Authentication provided by the Yubikey OTP token

This extension requires the used to already have a valid account and password and verifies that the user exists along with the password provided by verifying that the user presented a valid Yubikey OTP.

## Installation

This plugin requires Rails 4.x, 3.0.x, 3.1.x and 3.2.x and Devise 2.2.3+. Additionally the Yubikey Ruby library found here is required.

[https://github.com/titanous/yubikey](https://github.com/titanous/yubikey)
                                                 
The latest git version has a fix for a MITM attack element when communicating with the Yubico servers, this doesn't appear to be reflected in the published gem.

The gem for the Yubikey library will need to be added to your Gemfile. To install the plugin add this plugin to your Gemfile.

	gem 'yubikey_database_authenticatable'

## Setup

Once the plugin is installed, all you need to do is setup the user model which includes a small addition to the model itself and to the schema.

In order to communicate with the Yubikey authentication services the API key will need to be provided, this should be included into the Devise config, set yubikey_api_key and yubikey_api_id in the Devise configuration file (`config/initializers/devise.rb`).

Get a key here: [https://upgrade.yubico.com/getapikey/](https://upgrade.yubico.com/getapikey/)

``` ruby
	config.yubikey_api_key = "" # => API Key must be set to validate one time passwords
	config.yubikey_api_id = ""  # => API ID must be set to validate one time passwords
```

The following needs to be added to the User module.

``` ruby
	add_column :users, :use_yubikey, :boolean
	add_column :users, :registered_yubikey, :string
```

then finally add to the model:

``` ruby
	class User < ActiveRecord::Base

      devise :yubikey_database_authenticatable, :trackable, :timeoutable

      # Setup accessible (or protected) attributes for your model if using rails 3 or lower
      attr_accessible :use_yubikey, :registered_yubikey, :yubiotp

	  attr_accessor :yubiotp
		
	  def registered_yubikey=(yubiotp)
	    write_attribute(:registered_yubikey, yubiotp[0..11])
	  end
	
      ...
	end
```

If using rails 4, the params are controlled by strong params and need to be updated in your application_controller.rb. The following settings reflect a devise config allowing username or email and password or yubikey

``` ruby
  	before_filter :configure_permitted_parameters, if: :devise_controller?

  	protected
    	def configure_permitted_parameters
      	  devise_parameter_sanitizer.for(:sign_up) { |u| u.permit(:username, :email, :password, :password_confirmation) }
      	  devise_parameter_sanitizer.for(:sign_in) { |u| u.permit(:username, :email, :password, :login, :use_yubikey, :registered_yubikey, :yubiotp) }
      	  devise_parameter_sanitizer.for(:account_update) { |u| u.permit(:username, :email, :password, :password_confirmation, :current_password) }
    	end
```

## Local Verifier

If you're using a local Yubikey verifier (rather than the YubiCloud verifier) you'll need to specify the following in `config/initializers/devise.rb`:

``` ruby
  config.yubikey_api_url = ""           # => API Verifier URL
  config.yubikey_certificate_chain = "" # => Path to SSL cert for verifier
```

While a local verifier may work without an API key, you must use one per the upstream Yubikey module. If you're using the ykval server, ensure your database has a valid API ID and secret in the `clients` table.

## Copyright

Copyright (c) 2011-2015 Stephen Kapp, Released under MIT License 

Some bits borrowed from moneytree fork of original gem.
