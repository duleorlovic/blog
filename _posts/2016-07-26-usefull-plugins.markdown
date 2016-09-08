---
layout: post
title: Usefull plugins, services and opensource stuff
---

# Javascript plugins

* [jbox](http://stephanwagner.me/jBox) notifications, popups ...
* "One minute ago" can be updated automatically in this [timeago](http://timeago.yarp.com/) plugin
* [zeroclipboard](https://github.com/zeroclipboard/zeroclipboard) to copy some
text on click
* [jsPDF](https://github.com/MrRio/jsPDF) pdf generator [examples](http://mrrio.github.io/jsPDF/#)

* select something
  * [tokenize](http://zellerda.com/projects/jquery/tokenize) for tags,
    preselecting items
  * nice js filter [bouncy content
  filter](http://codyhouse.co/demo/bouncy-content-filter/)
  * javascript input filter
  [http://digitalbush.com/projects/masked-input-plugin/](http://digitalbush.com/projects/masked-input-plugin/)
  * [selectize](http://brianreavis.github.io/selectize.js/) and
  [selectize-rails](https://github.com/manuelvanrijn/selectize-rails)
  * [multiselect](http://loudev.com/) nice way to select multiple items from
    opened select box. If you show in popup with different data, then call
    `select.multiSelect('refresh');` after you initialize multiSelect.
    * if you want to keep order `keepOrder` option shows nice order, but
      submitting the form do not preserve order. You can use this
      [solution](http://stackoverflow.com/a/22271944/287166)
      [issue](https://github.com/lou/multi-select/issues/184)

    ~~~
    select.multiSelect({
      keepOrder: true,
      afterSelect: function(value){
        $('#templates-select option[value="'+value+'"]').remove();
        $('#templates-select').append($("<option></option>").attr("value",value).attr('selected', 'selected'));
      },
    });
    ~~~

    * `keepOrder` does not work for preselected items, solution could be to
      manually select (not already select: selected):

    ~~~
    $.each(selectedTemplateIds, function(index, template_id) {
      select.multiSelect('select', [String(template_id)]);
    });
    ~~~

* number input field [spinner](https://github.com/vsn4ik/jquery.spinner)
* toggle checkbox [draggable
  switch](http://www.bootstrap-switch.org/examples.html)

* calendar date and time picker
  * [pickdate](http://amsul.ca/pickadate.js/)
  * [jquery ui datepicker](https://jqueryui.com/datepicker/)
    Error on chrome if you use `type=date`
    [link](http://stackoverflow.com/questions/16890376/chrome-type-date-and-jquery-ui-date-picker-clashing)
    so use `type=text`
* image file upload drag and drop crop
  * [jquery.fileapi](http://rubaxa.github.io/jquery.fileapi/) supports
    webcam, crop, dragndrop
  * [jQuery-File-Upload](https://blueimp.github.io/jQuery-File-Upload/)
  * [browser-camera](https://davidwalsh.name/browser-camera) html5 getUserMedia
* rich text editor wyswyg
  * [quill](https://quilljs.com/)

# Graphs

* [graphs
gallery](https://github.com/mbostock/d3/wiki/Gallery) and
[http://bl.ocks.org/mbostock](http://bl.ocks.org/mbostock)
[slider](http://square.github.io/crossfilter/)
[people](http://www.findtheconversation.com/concept-map/#population)
[parallel](http://exposedata.com/parallel/) and the best is
[grafana](http://play.grafana.org/)
* [d3 for charts](http://plottablejs.org/examples/)

# Chrome plugins and firefox extensions

* [user agent switcher](https://github.com/chrispederick/user-agent-switcher/) 

# Free tier services

* [formspree](http://formspree.io/) sending emails in javascript
* [caniuse.com](http://caniuse.com/) to see what is supported and what is not
in modern browsers
* optimization for speed [google
insights](https://developers.google.com/speed/pagespeed/insights/)
[gtmetrix](https://gtmetrix.com/reports/www.scuddle.com/zP3xxAuZ)

# Payment

* https://www.braintreepayments.com/
* https://www.paddle.co/
* https://www.payola.io/

# Ruby

* [yml_gtranslate](https://github.com/zenchief/yml_gtranslate) automatic
generate translated yml locale files for rails localisations using google
translate `for f in $(find config/locales/ -type d);do yml_gt en rs $f;done`.
Also usefull when want to grep only en.yml `grep -i catar config/locales
--include *.en.yml -R`
* [memory_profiler](https://github.com/SamSaffron/memory_profiler)

# Rails

* crowdsourcing [catarse](https://github.com/catarse/catarse) [tilt](https://github.com/crowdtilt/crowdtiltopen/) is not actually opensource, since their api should be used

# Templates

* [adminLTE](https://github.com/almasaeed2010/AdminLTE)
[preview](https://almsaeedstudio.com/preview)
[documentation](https://almsaeedstudio.com/themes/AdminLTE/documentation/index.html)
[blog](https://almsaeedstudio.com/blog/features-of-adminlte-2.1)
* [gentelella](https://github.com/puikinsh/gentelella)
[preview](https://colorlib.com/polygon/gentelella/index.html) [rails
version](https://github.com/iogbole/gentelella_on_rails)

# Chrome developer tools

* [network tab filter
  request](https://developers.google.com/web/tools/chrome-devtools/profile/network-performance/resource-loading#filter-requests)
  will filter by filename containg the string `posts`. But it also supports
  keywords with autosuggestions. Examples: `domain:*.com`, `larger-than:1K`, 
* to clear autofill suggestions you can use keyboard shortcut. First select
  suggestion with UP or Down arrows than press Shift + Delete
  [answer](https://support.google.com/chrome/answer/142893?p=settings_autofill&rd=1)

# Nice design and ui tools

* [granim.js](https://sarcadass.github.io/granim.js/examples.html)
