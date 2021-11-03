---
title: "Designing Maintainable Calabash Tests: Part Two"
categories:
  - Blog
tags:
  - Calabash
  - Cucumber
  - Mobile App Testing
---

## Background
In a previous example entitled ["Designing maintainable calabash tests using Screen Objects"](../designing-manintainable-calabsh-tests-pt1/) I introduced a possible solution to creating maintainable cross platform calabash tests. Whilst this was a good start, there was room for improvement.<br>

In my opinion, there is no better way to design maintainable automated tests than to adopt an object-oriented/Page-Object like approach. The obvious benefit is a robust and well maintainable test suite. <br>

## The Problem
While on the face of it the first example may seem fine, but it could have implications. The fact that we are accessing calabash commands directly from our screen-objects, and the inability to drive the tests using different drivers (should you decide) could provide a maintenance headache in the future.<br>

## The Solution
We refactor our Base Class, abstracting all common driver (calabash) actions and write our own wrapper methods around these. We can then leverage them in our screen-objects.

### Example
In this example we are going to refactor the code from the [previous example.](../designing-manintainable-calabsh-tests-pt1/).
```ruby
require 'calabash-android'
require 'calabash-android/operations'
 
class BaseClass
  include Calabash::Android::Operations
 
  def initialize(driver)
    @driver = driver
  end
 
  def method_missing(sym, *args, &block)
    @driver.send sym, *args, &block
  end
 
  def tap_on(element)
    touch("* marked:'#{element}'")
  end
 
  def exists?(element)
    element_exists("* marked:'#{element}'")
  end
 
  def keyboard_enter_text(el, text)
    query("* id:'#{el}'", {:setText => text})
  end
 
  def wait_for_no_progress_bars
    performAction('wait_for_no_progress_bars')
  end
 
  def wait_for_dialog_to_close
    performAction('wait_for_dialog_to_close')
  end
 
  def wait_for_element(element)
    wait_for { exists?(element) }
  end
 
  def self.element(name, &block)
    define_method(name.to_s, &block)
  end
 
  class << self
    alias :value :element
    alias :action :element
  end
 
end

```
<br>
As you can see above, within our base-class we now have a set of common methods that wrap around calabash commands. I have also removed reference to Calabash ABase. The previous example inherited from the Calabash base class. There’s no need for this now we are creating our own wrapper methods around calabash commands.

Now we have refactored this, (and like the previous examples) the screen-objects will inherit from it. We must now refactor our sceen-objects to remove any direct calls to calabash commands, and replace them with our custom methods.

### Before
```ruby
class LoginScreen < DroidPress
 
  element(:username_field)     { 'username' }
  element(:password_field)     { 'password' }
  element(:login_button)       { 'save' }
 
  value(:not_logged_in?)       { element_exists("* id:'#{login_button}'") }
 
  action(:touch_login_button)  { touch("* id:'#{login_button}'") }
 
  def login_with(username, password)
      query("* id:'#{username_field}'", {:setText => username})
      query("* id:'#{password_field}'", {:setText => password})
      performAction('scroll_down')
      touch_login_button
      performAction('wait_for_no_progress_bars')
      performAction('wait_for_dialog_to_close')
  end
 
end
```

### After
```ruby
class LoginScreen < BaseClass
 
  element(:username_field)     { 'nux_username' }
  element(:password_field)     { 'nux_password' }
  element(:login_button)       { 'nux_sign_in_button' }
  element(:forgot_password)    { 'forgot_password' }
  element(:create_account_btn) { 'nux_create_account_button' }
 
  value(:await)                { wait_for_element(username_field) }
  value(:not_logged_in?)       { exists?(username_field) }
 
  action(:touch_login_button)  { tap_on(username_field) }
 
  def login_with(username, password)
    keyboard_enter_text(username_field, username)
    keyboard_enter_text(password_field, password)
    touch_login_button
    wait_for_no_progress_bars
  end
 
end
```

As you can see little has changed, the only thing we have done is called our wrapper methods from the base class instead of calling calabash commands directly (The ID’s etc have changed slightly as this example is using the latest version of the wordpress app).<br>
Once this is complete our step definitions remain unchanged, however to remove duplication I decided to refactor these slightly.
### Before
```ruby
Given(/^the app is launched$/) do
  @screen = page(WordPressApp)
end
 
When(/^I login with (valid|invalid) credentials to Add WordPress.com blog$/) do |negate|
  @screen.welcome_screen.await
  @screen.welcome_screen.touch_add_blog
  @screen.login_screen.await
  @screen.login_screen.login_with(USERS[:valid][:email], USERS[:valid][:password]) if negate.eql? 'valid'
  @screen.login_screen.login_with(USERS[:invalid][:email], USERS[:invalid][:password]) if negate.eql? 'invalid'
end
 
Then /^I (should|should not) be logged in$/ do |negate|
  if negate.include? 'not'
    @screen.login_screen.should be_not_logged_in
  else
    @screen.home_screen.await
    @screen.home_screen.should be_logged_in
  end
end
```

### After
```ruby
When(/^I login with (valid|invalid) credentials to Add WordPress.com blog$/) do |negate|
  @screen.login_screen.await
  @screen.login_screen.login_with(USERS[:valid][:email], USERS[:valid][:password]) if negate.eql? 'valid'
  @screen.login_screen.login_with(USERS[:invalid][:email], USERS[:invalid][:password]) if negate.eql? 'invalid'
end
 
Then /^I (should|should not) be logged in$/ do |negate|
  if negate.include? 'not'
    @screen.login_screen.should be_not_logged_in
  else
    @screen.home_screen.should be_logged_in
  end
end
```

You will see that I have removed the `Given the app is launched` step. Rather than initialize this class at the beginning of each scenario I moved into a hook:
```ruby
  #hooks.rb file (lives in the support dir)
  Before do |scenario|
    @screen = WordPressApp.new(self)
  end
```
This hook will initialize our screen-objects before each scenario is executed.

## Summary
We are now in a position where our screen-objects can access common actions without directly accessing calabash code. Therefore, if in the event that calabash commands change, you will only have to refactor one class.<br>

As well as this, should you decide to switch drivers (for example Appium) it would require little effort do do this.<br>

All example code can be found on GitHub, [here](https://github.com/ianrhamilton/-calabash-screen-object-example-part-two).<br>

Thanks for reading, any feedback welcome!<br>

~ Ian
