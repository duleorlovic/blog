5 categories:

* base: usually single element selectors or pseudo-class (but not a class selector). Its default styling for elements in all occurences on the page.
* layout: divide page into sections. prefix with `l-` like `l-inline`
* module: reusable parts: callouts, product lists... When used in different part of page you should use subclassing class_name-module, for example: `<div class="pod pod-callout">` instead of specificity war `.callout .pod`
* state: how module or layout looks in particular state (active/inactive, home page/sidebar, small/big screens), prefixed with `is-`, like `is-hidden`. For specific states of module, you can use `is-callout-minimized` and place it inside module code so its loaded only when module is loaded (for just-in-time loading).
State changes can happen by: class name (js add/remove class), pseudo class (:hover,:focus) and media query. Always keep module all css code inside same file.
* theme: how module or layout might look


http://work.stevegrossi.com/2014/09/06/how-to-write-css-that-scales/
