todo:
without $
animate faster at the end, slower at begining

<span style="color:green" id="notice"></span>
<span style="color:red" id="alert"></span>
<script>
  <%=raw "flash_notice('#{j notice}');" if notice %>
  <%=raw "flash_notice('#{j alert}');" if alert %>
</script>


function flash_appear($element, message, i) {
  if (i == undefined)
    i = 0;
  $element.text(message.substring(0,i));
  setTimeout(function(){ 
    if (i<message.length)
      flash_appear($element,message,i+1);
  }, 5);
}
function flash_dissapear($element, message, i) {
  if (message == undefined)
    message = $element.text();
  if (i == undefined)
    i = message.length-1;
  $element.text(message.substring(0,i));
  setTimeout(function(){ 
    if (i>0)
      flash_dissapear($element,message,i-1);
  }, 5);
}
function flash_alert(message) {
  flash_appear($('#alert'),message);
  setTimeout(function(){ flash_dissapear($('#alert')); }, 3000);
}
function flash_notice(message) {
  flash_appear($('#notice'),message);
  setTimeout(function(){ flash_dissapear($('#notice')); }, 3000);
}


