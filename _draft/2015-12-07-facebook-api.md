---
layout: post
---

# Share buttons

* feed share on [facebook feed](https://developers.facebook.com/docs/sharing/reference/feed-dialog) requires FACEBOOK_APP_ID. It allows share only on timeline (not for other pages you are admin). I allows custom images.
* [share dialog](https://developers.facebook.com/docs/sharing/reference/share-dialog) reads metatags and is cached (it uses `/dialog/share`). Also needs fb sdk and FACEBOOK_APP_ID
* [share button](https://developers.facebook.com/docs/plugins/share-button)
 you can put a link (it uses `/sharer/sharer.php`) without sdk and
 FACEBOOK_APP_ID. Custom images are not supported so you should use meta tags.
 I do not override properties on url (I just provide a link) and let facebook
 get the page and cache it.
 https://developers.facebook.com/docs/sharing/webmasters

 ~~~
 <meta property="og:url"           content="https://www.your-domain.com/your-page.html" />
  <meta property="og:type"          content="website" />
  <meta property="og:title"         content="Your Website Title" />
  <meta property="og:description"   content="Your description" />
  <meta property="og:image"         content="https://www.your-domain.com/path/image.jpg" />

  <meta property="fb:app_id" content="123">
 ~~~

* image should be 600x315 (minimum is 200x200). First user wont see image since
  crawler not yet cached the page. you should add so it is rendered first time
  ~~~
  <meta property="og:image:width"      content="200">
  <meta property="og:image:height"     content="500">
  ~~~
  https://developers.facebook.com/docs/sharing/best-practices/
* image wont be changed until image url is changed https://developers.facebook.com/docs/sharing/opengraph/object-properties#image

* new window in popup will be blocked if it is cause by script from ajax. It should be called `on-click`

~~~
var FACEBOOK_APP_ID='111...111';
// by default fb app is in development mode and can only be used by app admins, developers and testers
// add Contact Email on https://developers.facebook.com/apps/FACEBOOK_APP_ID/settings/
// toggle switch on https://developers.facebook.com/apps/FACEBOOK_APP_ID/review-status/
// add &show_error=true to feed url see all eventual errors
// https://developers.facebook.com/docs/sharing/reference/feed-dialog/v2.5#params
function share_facebook(obj){
  var loc = escape($(obj).attr('href'));
  var title  = escape($(obj).attr('title'));
  var image = escape($(obj).attr('image'));
  var caption = escape($(obj).attr('caption'));
  var description = escape($(obj).attr('description'));
  window.open("https://www.facebook.com/dialog/feed?app_id="+ FACEBOOK_APP_ID +"&link="+ loc +"&picture="+ image +"&name="+ title +"&caption="+ caption +"&description="+ description +"&redirect_uri=http%3A%2F%2Fwww.facebook.com%2F", 'newwindow', 'width=1000, height=500, top='+($(window).height()/2 - 225) +', left='+($(window).width()/2) +', toolbar=0, location=0, menubar=0, directories=0, scrollbars=0');
}

// https://dev.twitter.com/web/tweet-button/web-intent
// text is stripped to 140 - short_url_length (== 23) + one space
function share_twitter(obj){
  var loc = escape($(obj).attr('href'));
  var text  = escape($(obj).attr('text').substring(0,140-23-1));
  window.open('http://twitter.com/intent/tweet?url=' + loc + '&text=' + text + '&', 'twitterwindow', 'height=450, width=550, top='+($(window).height()/2 - 225) +', left='+$(window).width()/2 +', toolbar=0, location=0, menubar=0, directories=0, scrollbars=0');
}

// https://developer.linkedin.com/docs/share-on-linkedin
// click on Customized URL, max length for summary is 256
function share_linkedin(obj){
  var loc = escape($(obj).attr('href'));
  var title  = escape($(obj).attr('title'));
  var summary  = escape($(obj).attr('summary').substring(0,256));
  window.open("http://www.linkedin.com/shareArticle?mini=true&url=" + loc + "&title=" + title + '&summary=' + summary, 'newwindow' ,'width=550, height=500, top='+($(window).height()/2 - 225) +', left='+$(window).width()/2 +', toolbar=0, location=0, menubar=0, directories=0, scrollbars=0');
}
~~~

Usage in view:

~~~
<%# app/views/pages/_share_buttons.html.erb %>
<ul class="share-buttons">
  <li>
    <a href="<%= link %>" class="btn-share btn--face" title="<%= title %>" image="<%= image %>" description="<%= description %>" caption="<%= caption %>" onclick="share_facebook(this); return false;">Post It</a>
  </li>
  <li>
    <a href="<%= link %>" text="<%= title %>: <%= description %>" class="btn-share btn--tweet" onclick="share_twitter(this); return false;" target="_blank">Tweet</a>
  </li>
  <li>
    <a href="<%= link %>" rel="nofollow" title="<%= title %>" summary="<%= description %>" onclick="share_linkedin(this); return false;" target="_blank" class="btn-share btn--linkedin">Share</a>
  </li>
</ul>
~~~

Style

~~~
// app/assets/stylesheet/share_buttons.scss
.share-buttons {
  float: right;
  li {
    display: inline-block;
  }
}
.btn-share {
  color: white;
  padding: 0 12px;
  border: 1px solid #1d3c7e;
  position: relative;
  padding: 0px 5px 0px 46px;
  border-radius: 3px;
  text-decoration: none;
}
.btn--face,.btn--linkedin,.btn--tweet {
  &:before {
    position: absolute;
    content: "";
    width: 30px;
    height: 18px;
    top: 2px;
    left: 9px;
    background-size: 20px 16px;
    background-repeat: no-repeat;
    background-position: 50% 50%;
    border-right: 1px solid white;
  }
}
.btn--face {
  background-color: #3b5998;
  &:before {
    background-image: image-url('face-btn.png');
  }
}

.btn--linkedin {
  background-color: #0078a8;
  &:before {
    background-image: image-url('link-btn.png');
  }
}
.btn--tweet {
  background-color: #55acee;
  border-color: #3E80B3;
  &:before {
    background-image: image-url('tweet-btn.png');
  }
}
~~~
