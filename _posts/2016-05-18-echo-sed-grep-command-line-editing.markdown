---
layout: post
title: Echo Sed Grep command line editing
tags: echo sed grep edit command-line
---

# Echo

`echo -e "\n" >> filename` will add new line (that's why `-e` to the
filename). You can use single quotes so you do not need to write `-e` and `\n`
in multiline text but you need to escape other `'`. The best was to write
multiline text is with `cat > filename << HERE_DOC ... some lines with ' or "
...  HERE_DOC`. First `\HERE_DOC` or `'HERE_DOC'` when no parametar expanded.
Remember to use double `<<` not single `<` redirection. Inside `HERE_DOC` there
should not be backtick  \`\` since it will be evaluated as shell command.

~~~
cat > my.txt << 'HERE_DOC'
here can be ' " \ / anything
HERE_DOR
~~~

# Sed

`sed -i '/haus/a home' filename` will inplace (`-i`) search for *haus* and
append *home* after that line (beside insert before `i`, this could be `a`
append, `c` change matched line, `d` delete matched line). Multiple lines need
to have "\" at the end of line (multiline *echo ' ...'* does not need that
trailing backslash). You can use `sed -i "// a" file.txt` but then you need
`\n\` at the end of each line.  Remember that no char (even space) could be
after last `\`.

Most common usage is to add before some line, for example line begins with
`test`. If you need single quote `'` for example `'a'` than you need to use
prefix `$` and escape quote with single backslash ` \ `. You could also use
string concatenation: single with double quote string `'"'a'"'`.
If you need `\"` than put `\\"` so backslash survive bash command.

~~~
sed -i config/secrets.yml -e $'/^test:/i \
  # aws s3\
  aws_bucket_name: <%= ENV[\'AWS_BUCKET_NAME\'] %>'
~~~

Instead of search, you can append on line numbers `3,6aTEXT` or at the last line
`sed '$aTEXT_AT_END` (last line is `$` or `"\$"` if `"` is used) or at first
line

~~~
sed -i app/assets/stylesheets/application.scss -e '1i\
/*\
 *= require rails_bootstrap_forms\
 */'
~~~

`sed 's/find/replace/g` will replace word `find` with `replace`.

Brackets need to be escaped like `\(`, `\1` means first group, `&` means matched
text...
[tutorial](http://www.thegeekstuff.com/2009/10/unix-sed-tutorial-advanced-sed-substitution-examples/)
Adding `,` and new `text` after `match` could be with `sed
's/\(match.*\)/\1,\ntext/`

Sed has something different regular expressions so follow this [link](http://www.gnu.org/software/sed/manual/html_node/Regular-Expressions.html)

When you want to change chars (and not whole line) than you can use `s/me/you/g`, or several regexp in same command for example

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
and don't want to escape `'` and to add "\" to the end of each line, than you
can try following command [inspiration](https://stackoverflow.com/questions/26770426/use-sed-in-bash-to-replace-string-with-heredoc/26770678#26770678)
which will replace new lines with `ћ` (`N` means multiline match, `a` label... multiline match `:a;N;$!ba;s/\n/ћ/g`, note that `$` needs to be escaped if inside `""`) and than return back new line.

Quote in `'HERE_DOC'` will not [substitute
params](http://tldp.org/LDP/abs/html/here-docs.html#EX71C) so `$` `/` or \
will remain in here doc. Instead of `<<` you can use `<<-` and closing HERE_DOC
can be indented with *tab* character.  For regexp we need to escape `/` and *\*
with `\/` and `\\` (`s:\\:\\\\:g;s:/:\\/:g`)

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

If you need `sudo` than you can use `tee` for example

~~~
cat << HERE_DOC | sudo tee -a /etc/systemd/system/mongodb.service
HERE_DOC
~~~

# Grep

Just a few command line options with grep

* `grep pattern filename` so if you omit filename than standard input will be
  used, which is nice to test your big regex, for example `asd <<` (exit with
  Ctrl+d)
  * `grep "asd <<"`
* `grep -o 111` will output only matched parts (not whole possible big lines)
* `grep -l asd` output only filenames
* grep only specific file type is with `grep asd --include \*.yml` or with a
  find command
  * `find . -name *.yml -exec grep asd {} \;`
* rename could be done with simple regex:
  * `rename 's/\.in.html.erb/\.html.erb/' app/views/*/*`
  * `rename 's/\.in.html.erb/\.html.erb/' app/views/*/*/*` for nested folders
* if you want to output only matching group than it is better to use `sed`
  * `sed -n 's/^.*[^0-9]\([0-9][0-9]*\).*/\1/p'` get only numbers
  * `echo "asd123" | sed -n 's/asd/***/p'` replace asd with `***`
* exclude mathing with `grep -v` that is invert match

# Regex

<https://regex101.com/> enable multiline flag if you are using `$` end of line
<https://github.com/tom-lord/regexp-examples#usage> to show matching examples

* OR is with `(|)`. But you need to escape `|` for example: `\(asd\|qwe\)` lines
that contains asd or qwe. We need parentheses because alternator
operator `|` has the lowest precedence of all, so usually you want world
boundaries like `^([0-9]|[1-9][0-9])$` (0..99, but not 0asd, or asd9
* contains asd but not qwe is `asd((?!qwe).)*$`. Oposite not containing.
* contains `asd` and `qwe` but not `zxc` in between `asd((?!qwe).)*zxc`
* include end of line for multiline search, use matcher `\_.` finds any
character including end-of-line. Use `\n` for new
line character. `\{-}` stopping at first occurence (`*` is too greedy and would
stop at last occurence). If you want ruby regex ignore new line you can use
modifier `m`, like match all dl `s.match /dl.*dl/m`
* match start of line with lowercase `^[a-z]` (but one string could have
multiple lines). Better is to check start of string with lowercase `\A[a-z]` (in
ruby example) `"123\ntest".match /^[a-z]/` will return `t`... (you can use
alternative `string.start_with? /^[a-z]/`
* repetative match is with `{number}` like for example 000...999 `^[0-9]{3}$`

When using grep, you can enable Per regexp PCRE with `-P` or `--per-regexp`.
That is needed for negative lookahead.
When using `-P` than no need to escape.

In bash you need to escape *()|\* (for both single and double quotes).
In vim you also need to escape again (for both single and double quotes):
`grep '\(asd\\|qwe\)'` is the same as in vim `:grep '\\(asd\\\|qwe\\)'`. Note
When using `-P` than no need one escape (vim escape is still required).

In vim [vim pattern](http://vimdoc.sourceforge.net/htmldoc/pattern.html) search
inside buffer.


# AWK

* split by one char: `echo "1:3" | awk '{split($0,a,":");print a[1] a[2]}' # 13`
* split by multiple char: `echo "1:3x5" | awk -F':|x' '{print $1 $2 $3}' # 135`
