---
layout: single
title: Ruby Snippets
date:   2019-04-12 17:34
description: 
tags:
- ruby
---

# spec/rspec_helper.rb
```ruby
require 'rspec-rails'
require 'faker'
require 'webmock/rspec'

###############################################################################
### General
RSpec.configure do |config|
  config.order        = :random
  config.fixture_path = "#{::Rails.root}/spec/fixtures"
  config.infer_spec_type_from_file_location!
  config.filter_rails_from_backtrace!

  config.include ActiveSupport::Testing::TimeHelpers
end

### Loads all RSpec shared examples and shared contexts
# spec/shared/contexts/*.rb
# spec/shared/examples/*.rb
Dir[File.join(__dir__, 'shared', 'contexts', '*.rb')].each { |file| require file }
Dir[File.join(__dir__, 'shared', 'examples', '*.rb')].each { |file| require file }

###############################################################################
### Shoulda::Matchers
require 'shoulda/matchers'

Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end

###############################################################################
### FactoryBot
require 'factory_bot_rails'
require 'factory_bot'

RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end

###############################################################################
### DatabaseCleaner
require 'database_cleaner'
RSpec.configure do |config|

  config.around(:each) do |example|
    DatabaseCleaner.cleaning do
      example.run
    end
  end

  # set `:type` for services (Service Objects) directory
  config.
    define_derived_metadata(
      file_path: Regexp.new('/spec/services/')
    ) do |metadata|
    metadata[:type] = :service
  end

end

###############################################################################
### Capybara
# Check .env.test for examples of flags you can use in .env.test.local

require "selenium/webdriver"
require "chromedriver-helper"

Capybara.app_host              = ENV['HOST_URL']
Capybara.current_driver        = ENV['CAPYBARA_DRIVER'].to_s.to_sym
Capybara.default_max_wait_time = ENV['CAPYBARA_DEFAULT_MAX_WAIT'].to_s.to_sym

# Helper that makes requests to the frontend skip rack and behave like normal requests
def frontend
  Capybara.app_host   = ENV['FRONTEND_URL']
  Capybara.run_server = false
  yield
  Capybara.app_host   = nil
  Capybara.run_server = true
end

# (Capybara) saves a screenshot and PNG of each test
Rails.configuration do |c|
  c.after(:each) do |example|
    if example.metadata[:type] == :feature
      save_and_open_page
      save_and_open_screenshot
    end
  end
end
```