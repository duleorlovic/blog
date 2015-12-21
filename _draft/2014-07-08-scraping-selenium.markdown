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

USER_EMAIL = "asd@asd.asd"
USER_PASSWORD = "asdasd"
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




To parse some text, you can use nokogiri `data = Nokogiri::HTML(html_page)`

[http://www.w3.org/TR/xpath](http://www.w3.org/TR/xpath)
Usefull selector 
http://ejohn.org/blog/xpath-css-selectors/

xpath
  find id  "//*[@id='my_id']"
  class "//*a[contains(@class,'my_class')]"
  text "//*[contains(text(),'ABC')]"
  parrent "../"
  some child of this ".//"


Mechanize session 

~~~
class ImageService

  attr_reader :name
  def initialize name
    @name = name
  end

  def get_links
    agent = Mechanize.new
    page = agent.get 'https://www.google.com/imghp'
    form = page.form('f')
    form.q = name
    page = agent.submit(form)

    # results
    table = page.search(".//table[@class='images_table']").first
    results = []
    table.search('a').each do |a|
      results.append( {
        image_url: a.search('img').first.attributes["src"].to_s,
        site_url: a.attributes["href"].to_s[7..-1], # /url?q=
      })
    end
    results
  rescue
    page.body.to_s
  end

end
~~~

~~~
<% if @results.class == Array %>
  <% results.each do |res| %>
    <%= res[:image_url] %>
  <% end %>
<% else %>
  <iframe id="FileFrame" src="about:blank"></iframe>
  <script type="text/javascript">
var doc = document.getElementById('FileFrame').contentWindow.document;
doc.open();
doc.write('<%=raw @results %>');
doc.close();
  </script>
  <%= @links %>
<% end %>
~~~
