# Scraping tool
#
# run with: ruby ./snag_selenium.rb
# or in irb:
# driver = nil,wait=nil,id=nil
# eval File.open('./snag_selenium.rb').read
#

require "selenium-webdriver"
# http://selenium.googlecode.com/git/docs/api/rb/Selenium/WebDriver.html
# http://docs.seleniumhq.org/docs/index.jsp
require 'csv'

USER_EMAIL = "dominospizzajobs411@gmail.com"
USER_PASSWORD = "getsome2**"
TEST_MODE = false

if driver.nil?
  driver = Selenium::WebDriver.for :firefox
  #driver.manage.timeouts.implicit_wait = 10 do not use this since it can hang out
  wait = Selenium::WebDriver::Wait.new(:timeout => 30) # seconds

  driver.navigate.to "https://hiring.snagajob.com/tms/?refid=hbtsignin"

  puts "Signing in..."
  element = nil
  wait.until { element = driver.find_element(:name, 'UserName') }
  element.send_keys USER_EMAIL

  element = driver.find_element(:name, 'Password')
  element.send_keys USER_PASSWORD

  element.submit
end


puts "Finding profile_search..."
profile_search = nil
wait.until { profile_search = driver.find_element(:xpath, '//*[text()[contains(.,"Profile")]]') }
profile_search.click





xpath
  find id  "//*[@id='my_id']"
  class "//*a[contains(@class,'my_class')]"
  text "//*[contains(text(),'ABC')]"
  parrent "../"
