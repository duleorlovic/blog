---
layout: post
title: Echo and Sed command line editing
tags: echo sed edit command-line
---

# Echo

`echo -e "\n" >> filename` will add new line (that's why `-e` to the
filename). You can use single quotes so you do not need to write `-e` and `\n`
in multiline text but you need to escape other `'`. The best was to write
multiline text is with `cat > filename << HERE_DOC ... some lines with ' or " ...
HERE_DOC`. First `\HERE_DOC` or `'HERE_DOC'` when no parametar expanded.
Remember to use double `<<` not single `<` redirection.

~~~
cat > my.txt << 'HERE_DOC'
here can be ' " \ / anything
HERE_DOR
~~~

# Sed

`sed -i '/haus/a home' filename` will inplace (`-i`) search for *haus* and 
append *home* after that line (beside insert before `i`, this could be `a`
append and `c` change matched line). Multiple lines need to have `\` at the end
of line (multiline *echo ' ...'* does not need that trailing backslash). You can
use `sed -i "// a" file.txt` but then you need `\n\` at the end of each line.
Remember that no char (even space) could be after last `\`.

Most common usage is to add before some line, for example line begins with
`test`. If you need really need `'` for example `'a'` than you can replace it
with `'"'a'"'`. If you need `\"` than put `\\"` so backslash survive bash
command.

~~~
sed -i config/secrets.yml -e '/^test:/i \
  # aws s3\
  aws_bucket_name: <%= ENV["AWS_BUCKET_NAME"] %>'
~~~

You can append on line numbers `3,6aTEXT` or at the last line `sed
'$aTEXT_AT_END` (last line is `$` or `"\$"` if `"` is used)

`sed 's/find/replace/g` will replace word `find` with `replace`.

Brackets need to be escaped like `\(`, `\1` means first group, `&` means matched
text...
[tutorial](http://www.thegeekstuff.com/2009/10/unix-sed-tutorial-advanced-sed-substitution-examples/)
Adding `,` and new `text` after `match` could be with `sed
's/\(match.*\)/\1,\ntext/`

Sed has something different regular expressions so follow this [link](http://www.gnu.org/software/sed/manual/html_node/Regular-Expressions.html)

When you want to change chars (and not whole line) than you can use `s/me/you/g', or several regexp in same command for example

~~~
sed src/app/index.module.coffee \
  -e 's/, /,\n  /g' \
  -e 's/  \[/\[\n  /g' \
  -e 's/\]/,\n\]/'
~~~


If you have a lot regexp, than its better to use [here-doc](http://tldp.org/LDP/abs/html/here-docs.html)
and read from standard input `-` [link](https://unix.stackexchange.com/questions/45591/using-a-here-doc-for-sed-and-a-file/45592#45592?newreg=39c715cb752f44a9bba9b3f3f74f2015)

~~~
sed src/app/index.module.coffee -f - <<HERE_DOC
s/, /,\n  /g
s/  \[/\[\n  /g
s/\]/,\n\]/
HERE_DOC
~~~

Note that all commands one per line. As with `sed ''` you need backslash for each line of multiline command.

 When you really need to add multiline template
and don't want to escape `'` and to add `\` to the end of each line, than you
can try following command [inspiration](https://stackoverflow.com/questions/26770426/use-sed-in-bash-to-replace-string-with-heredoc/26770678#26770678)
which will replace new lines with `ћ` (`N` means multiline match, `a` label... multiline match `:a;N;$!ba;s/\n/ћ/g`, note that `$` needs to be escaped if inside `""`) and than return back new line.

Quote in `'HERE_DOC'` will not [substitute params](http://tldp.org/LDP/abs/html/here-docs.html#EX71C) so `$` `/` or `\` will remain in here doc. Instead of `<<` you can use `<<-` and closing HERE_DOC can be indented with *tab* character.
For regexp we need
to escape `/` and `\` with `\/` and `\\` (`s:\\:\\\\:g;s:/:\\/:g`)

~~~
sed src/app/index.module.coffee -e "s/client/$(sed ':a;N;$!ba;s/\n/ћ/g;s:\\:\\\\:g;s:/:\\/:g;' <<'HERE_DOC'
    I'm "$just"
    some long /'\ multiline <\template>.
HERE_DOC
)/g;s/ћ/\n/g"
~~~

Sometimes we need to add/replace line with template (not replace regexp)
than we need to move replacing `s/#/\n/g` outside of sed since it will
not be applied to new template.
Note than we can't use inline replacement but we can move tmp file.

~~~
sed src/app/index.module.coffee -e "/PLACEHOLDER/a $(sed ':a;N;$!ba;s/\n/ћ/g;s:\\:\\\\:g;s:/:\\/:g;' <<'HERE_DOC'
    I'm "$just"
    some long /'\ multiline <\template>.
HERE_DOC
)" | sed "s/ћ/\n/g" > tmp && mv tmp src/app/index.module.coffee
~~~

Here is another solution which replace the PLACEHOLDER line
[link](https://stackoverflow.com/questions/26770426/use-sed-in-bash-to-replace-string-with-heredoc/26770678#26770678)

~~~
cat > /tmp/template <<\HEREDOC
    I'm "$just"
    some long /'\ multiline <\template>.
HERE_DOC
sed -i myfile.rb -e '/PLACEHOLDER/ {
  r /tmp/template
  d
}'
rm /tmp/template
~~~

