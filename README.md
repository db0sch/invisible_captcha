# Invisible Captcha

[![Gem Version](https://badge.fury.io/rb/invisible_captcha.svg)](http://badge.fury.io/rb/invisible_captcha) [![Build Status](https://travis-ci.org/markets/invisible_captcha.svg)](https://travis-ci.org/markets/invisible_captcha)

Simple and flexible spam protection solution for Rails applications. Based on the `honeypot` strategy to provide better user experience.

## Background

This strategy is based on adding an input field into the form that:

* should not be visible by the real users
* should be left empty by the real users
* will most be filled by spam bots

## Installation

Add this line to you Gemfile:

```
gem 'invisible_captcha'
```

Or install the gem manually:

```
gem install invisible_captcha
```

## Usage

There are different ways to implement, at Controller or Model level:

### Controller style

View code:

```erb
<%= form_tag(create_topic_path) do |f| %>
  <%= invisible_captcha %>
<% end %>
```

Controller code:

```ruby
class TopicsController < ApplicationController
  invisible_captcha only: [:create, :update]
end
```

This method will add a filter (a `before_filter`) into the controller that triggers when the spam is detected. By default it responds with no content (only headers: `head(200)`). But you are able to define your own callback passing the `on_spam` option:

```ruby
invisible_captcha only: [:create, :update], on_spam: :your_on_spam_callback_method

private

def your_on_spam_callback_method
  redirect_to root_path
end
```

### Controller style (resource oriented):

In your form:

```erb
<%= form_for(@topic) do |f| %>
  <%= f.invisible_captcha :subtitle %>
<% end %>
```

In your controller:

```ruby
invisible_captcha only: [:create, :update], honeypot: :subtitle
```

### Model style

View code:

```erb
<%= form_for(@topic) do |f| %>
  <%= f.invisible_captcha :subtitle %>
<% end %>
```

Model code:

```ruby
class Topic < ActiveRecord::Base
  attr_accessor :subtitle # define a virtual attribute, the honeypot
  validates :subtitle, :invisible_captcha => true
end
```

If you are using [strong_parameters](https://github.com/rails/strong_parameters), don't forget to keep the honeypot attribute into the params hash:

```ruby
def topic_params
  params.require(:topic).permit(:subtitle)
end
```

## Options and customizations

### Controller method options:

The `invisible_captcha` method accepts some options:

* `only`: apply to given controller actions
* `except`: exclude to given controller actions
* `honeypot`: custom name of honeypot
* `resource`: custom name of controller
* `on_spam`: custom callback when spam is detected

### Plugin options:

You also can customize some plugin options:

* `sentence_for_humans`: text for real users if input field was visible
* `error_message`: error message for model validation (model implementation)
* `honeypots`: collection of default honeypots
* `visual_honeypots`: make honeypots visible, useful to test personalizations

To change these defaults, add the following to an initializer (recommended `config/initializers/invisible_captcha.rb`):

```ruby
InvisibleCaptcha.setup do |config|
  config.sentence_for_humans = 'If you are a human, ignore this field'
  config.error_message = 'You are a robot!'
  config.honeypots << 'another_fake_field'
  config.visual_honeypots = false
end
```

## Contribute

Any kind of idea, feedback or bug report are welcome! Open an [issue](https://github.com/markets/invisible_captcha/issues) or send a [pull request](https://github.com/markets/invisible_captcha/pulls).

## License

Copyright (c) 2012-2014 Marc Anguera. Invisible Captcha is released under the [MIT](LICENSE) License.
