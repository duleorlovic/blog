---
layout: post
---

https://developers.google.com/google-ads/api/docs/hotel-ads/overview

To use both google ads `AW-123...` and google analytics `UA-123-1` you should
call config multiple times https://stackoverflow.com/questions/54280439/how-to-combine-ua-and-aw-tracking-tags/54304710
```
<script async
src="https://www.googletagmanager.com/gtag/js?id=AW-11116216150">
</script>
<script>
  window.dataLayer = window.dataLayer || []; function
  gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'AW-11116216150');
  gtag('config', 'UA-111111118-1');
</script>
```
