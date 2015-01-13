---
layout: post
title:  Scraping using Selenium
categories: scraping selenium 
---


# scraping craigslists
#
# start VPN tunnel if you are not from supported geolocation (USA)
# run with ruby ./craigslist_selenium.rb
# or in irb:
# driver = nil,wait=nil,link=nil,email=nil,all_links=nil,email_element=nil
# eval File.open('./craigslist_selenium.rb').read

require "selenium-webdriver"
# http://selenium.googlecode.com/git/docs/api/rb/Selenium/WebDriver.html
# http://docs.seleniumhq.org/docs/index.jsp
require 'csv'

output = CSV.open('data/craigslist.csv', 'wb')

puts "Finding NEXT link craigslist site"
next_link = nil
wait.until do
  begin
    next_link = driver.find_element( :css, '.button.next')
  rescue Selenium::WebDriver::Error::StaleElementReferenceError
    # this happens with a site where page is replaced in javacript and there could be two buttons
    puts "old elemenet is no longer attached to the DOM"
    # when return value is false, it will keep findind
    false
  end
end

reply_element = nil
begin
  wait.until do
    reply_element = driver.find_element id: 'replylink'
  end
rescue Selenium::WebDriver::Error::TimeOutError
  # this exception is raised when wait block do not return true
  puts "\n#{link[:href]} does not have id replylink"
  no_link_count += 1
  # continue with next link
  next
end



