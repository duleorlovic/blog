---
layout: post
title: Vim theory
---

# Lean Vim Script The Hard Way

<http://learnvimscriptthehardway.stevelosh.com/>

# Mappings

`:help map` `:help 40`
maping keys syntax `:noremap <newkeys> <keynames>`

* list mappings `:map`. To save them in a file use those 3 commands
  ```
  :redir! > ~/Download/vim_keys.txt
  :silent verbose map
  :redir END
  ```
* since mappings easilly could be broken, ALWAYS use `nore` non recursive
variant `noremap`, `nnoremap` ... so it use default meanings of keynames
* `:map <unique> <script> asd asd` means that it will not use mappings from
  other places (only mappings defined in script) and it will fail if it is
  already defined
* you can map control CTRL key, for example `:map <c-d> dd` (`<c-d>` or `<esc>`
  is called angle bracket notation)
* `map` (normal visual and operator-pending), `nmap` normal,  `vmap` visual,
  `omap` operator-pending, `imap` insert, `map!` insert and command line, `cmap`
  command line, `xmap` ex command mode
* remove mappings `:unmap <key>` or `nunmap`
* you can map two keys, for example `:map -d dd`

* change leader key `:let mapleader = "-"` so you can write `:map <leader>d dd`
  (note that it does not have effect for already defined mappings)
* `:let maplocalleader = "\"` can be used for mapings only for local buffer.
  Mapping can be for specific buffer (file) `:nnoremap <buffer> <leader>d dd`.
  You can use both `:nnoremap <buffer> <localleader>d dd`
* to check if user has already define a mapping
  ```
  :echo !hasmapto('<Plug>TypecorrAdd')
  ```

Some examples
* `:nnoremap _ f_x~` find next `_`, remove and uppercase for moving underscore
  to CamelCase
* snippet `nnoremap ,html :-1read $HOME/.vim/.skeleton.html<CR>3jwf>a` [my
snippets](https://github.com/duleorlovic/config/tree/master/vim/snippets)
* `<silent>` is good if you are having mappings that launch ex commands.
* black screen is caused to silent command
  https://github.com/vim/vim/issues/1253 use `:redraw!`

# Abbreviations

Abbreviations are similar to mappings but for insert, replace and command mode.
They are shown while you are typing and are triggered by a non keyword char.

* `:iabbrev adn and` this will replace `adn<non-keyword-character>` with `and`
  Difference between `map` is that map does not count for non-keyword-character.
  If you have mapping in abbr result you can use `:noreabbrev` to it skips map
* `:abbreviate` list all abbreviations
* `:iabbrev <buffer> --- &mdash;` will replace `---` with `&mdash` but only for
current buffer. It is `<buffer>` local abbreviation and is usefull when you want
something only for specific type, for example `:autocmd FileType javascript
:iabbrev <buffer> iff if ()<left>` will enable little snippet only for js.
* `cabbrev E e` can be used to replace `E ` with `e ` on command line so when
  you type `:E<space>` it will end up in `:e<space>`
* `:abclear` to remove all abbrs or `:unabberviate adn` to remove only one

# Commands

* define custom command `:command MyCom 1delete` user defined commands must
  start with a capital letter
* define number of arguments: one `-nargs=1`, zero or more `-nargs=?` and use
  them with `<args>` or `<q-args>` (properly escaped string) for example
  ```
  :command -nargs=+ Say :echo <q-args>
  ```
* usually command call functions and `<f-args>` will properly pass the arguments
* if your command uses range than use `-range` attribute and use `<line1>` and
  `<line2>` in command.
* use `-bar` than command can be followed by `|`. Also you can use `<Bar>`
  inside command definition to join multiple commands
* to override use bang `:command! Say`. to remove `:delcommand Save`

# Autocommands

Autocommands are used to run commands on specific events (usually `set
filetype` or `call MyFun`)

`:autocmd BufNewFile * write`.

First param is event and can be something of: entering insert mode, not pressing
a key for some time. It can can combine, usually `BufNewFile,BufRead` for all
open files, existing or not. Some events `:help autocmd-events`

* `BufNewFile` when you open new files not saved to disk
* `BufReadPost` after you open existing file
* `BufWritePre` just before write
* `FileType` when vim sets filetype
  * `:autocmd FileType javascript nnoremap <buffer> <localleader>c I//<esc>`

Second param is "pattern" that filter more specific: for example: `*.txt` is
only triggered for txt files

Last part is command that is executed. Note that it is Command-line commands. so
to use normal mode commands you need `normal` (or `:normal!` which ignore
mappings that user could have written).
For example jump to the end of file
```
:autocmd BufReadPost *.log normal G
```
`normal` will use all text after it, so if you need to execute two commands than
wrap `normal G` inside `execute` and use `|`
```
:autocmd BufReadPost *.chg execute "normal ONew entry:\<Esc>" |
  \ 1read !date
```

You can group autocommands (`autocmd!` is to clear)

~~~
augroup filetype_html
    autocmd!
    autocmd FileType html nnoremap <buffer> <localleader>f Vatzf
augroup END
~~~

# Operator pending mappings

Operator is command waiting for movement command and than executes. For example
`d`, `y` and `c` are waiting for movements: `dt,` (delete till ,), `ci(` (change
inner (), `yw` (yank word).
We can remap movement keys for operator mode `:onoremap p i(` (inner braces,
select parametars), or `:onoremap b /end<cr>` (block, until end).
If you want to select before or some block after current position you can
visualy select and it will be executed on that selection.
For example, change params while current is on current line before params:

~~~
onoremap in( :<c-u>normal! f(vi(<cr>
~~~

`:normal!` means that it simulate pressing in normal mode (and not not use
eventual user mappings into consideration). `<cr>` at the end
executes `:normal!` command. You can indent from command mode: `:normal >>`
`<c-u>` needs to remove the range that vim may insert.

`:execute` takes vimscript string and performs as command `:execute "write"`.
It is used since `:normal` can not recognize special characters like `<cr>` (if
you try to use quotes `:normal "'` will just print/show them, so normal without
execute interpolation is just for commands without `<cr>` like operators and
moves).
Another usage is when you need to run normal commands from string (interpolate).
`execute` will substitute any special charactes before running string, special
chars are `:help expr-quote`. You can insert those special characters without
escaping if you use `ctrl-v` (`:help i_ctrl-v`) it will insert terminal code
* `\r` return `<CR>` (if you are using ctrl-v than it is ``)
* `\n` new line `<NL>`
* `\<_key_>` ( `\<Esc>` with ctrl-v ``)
* `\\` is backslash `\`. You can use single  as quote `normal '/,\+'` if you
  have many backslashes and you do not want to escape them, or those parts
  inside execute
  ```
  :executue "normal /foo.\\+\<cr>"
  :executue 'normal /foo.\+' . "\<cr>"
  ```

If you need to get value of option which name is in variable you can use eval
```
:let optname = "path"
:let optval = eval('&' . optname)

" The same thing can be done with:
:exe 'let optval = &' . optname
```

Example using execute to change current title header (=======\n Header) in
markdown text with keys `cih`:

~~~
:onoremap ih :<c-u>execute "normal! ?^==\\+$\r:nohlsearch\rkvg_"<cr>
~~~

# Map keys to execute Operator function

https://learnvimscriptthehardway.stevelosh.com/chapters/32.html Here we will
create usual mapping but execute our function, we will call it grep oprator.
`g@` will call operatorfunc as operator (so it can accept motion as any other
operator like `w` word or `i{` inner braces)
```
nnoremap <leader>g :set operatorfunc=GrepOperator<cr>g@

function! GrepOperator(type)
  echom "test"
endfunction
```

# Options and Variables

`:help 41`
For example
```
:let i = 1
:while i < 5
:  echo "count is " i
:  let i += 1
:endwhile
```
Yank those lines and execute with `:@"` command.
* `.vimrc` script file execute any colon command (ex command, command-line
  command) and you can omit colon in scripts
* to set a variable use `let {variable} = {expression}`. Local scope variables
  are with prefix like `:let b:a=2` (`b:` buffer, `w:` window, `t:` tab, `g:`
  global, `l:` local to function, `a:` argument, `s:` script).
  `:echo v:version` `v:` are vim variables
  Note that variables still exists when script finish. Check with:
  ```
  :if !exists("s:call_count")
  :  let s:call_count = 0
  :endif
  :let s:call_count = s:call_count + 1
  :echo "called" s:call_count "times"
  ```
  To clear variable use `:unlet s:call_count` (`:unlet!` to skipp error if not
  exists).
* `:setlocal nowrap` is only for current buffer (file), when you open another
  file (inside same window) it will not have nowrap. If you have same mappings
  than local will be chosen since it is more specific.
* Numbers can be: decimal `123`, hexadecimal `0x7f`, octal `012` (starts with
  zero), and binary `0b101` (starts with `0b`). `:echo 0xf` always print decimal
* Strings inside double quote can include special chars `"\tHi \"you\""` and
  single quote `'Hi ''you'''` can't use special chars. Concatenate with dot.
  Special characters are excaped with \ like `"\t \<Esc>"`
* use `$NAME` for environment variable, `&name` for option and `@r` for register
* to set option you can use `:set xxx` or `:set xxx=value` or `:let &xxx=value`
  To unset `:set noxxx` or `:set xxx!`. To check if it is set `:set xxx?` or
  `:echo &xxx`. To set to default value use ampersand `&` like `:set iskeyword&`
  For boolean options `1` is true and `0` is false. `:let` is more
  powerfull than `:set` since it can use math and other operations.
  Escape space with backslash like `:set tags=my\ nice\ file`
* to set variable use `:let foo = "bar"` and print `:echo 'foo=' foo` arguments
  will be separated with space.
  You can add or remove item from list, you can do with `:set k+=v` or `-=`.
  If you have space inside value than you need to escape `:set xxx+=my\ value`
* to set registers run `let @a = "hello"` so you can paste `"ap`. To read
  register run `:echo @a` (`:echo @"` is unnamed yank register, `:echo @\ ` is
  search register ... `:help registers`).
* to get current value of variable you can use `=` and `<TAB>` for example `:set
  xxx=` and press tab so vim autocomplete the value so you can edit it. Or you
  can press enter after the name (skip equal sign), for example `:se ft<cr>`
* print env variables on command line: `:echo $MYVIMRC` or `:echo filetype?`.
 `:echom $PATH` will save output into `:messages` so you can view later (it will
 differently show special characters `:echom "a\nb"` will not be shown as new
 line)

Multiline commands can be written as:

~~~
:echom "foo" | echom "bar"

" or

:augroup testgroup
:   autocmd BufWrite * :echom "Baz"
:autgroup END
~~~

To loop instead of `:while` you can use `:for i in range(1,4)`. Function
`range(n)` will create a list `[0, 1, ... n-1]`. Usefull in buffers
```
:for line in getline(1,20)
```

Inside loop you can use `:continue` (just to start and keep looping) and
`:break` (break the loop)
`sleep 40m` to sleep for 40 miliseconds

For conditional use `:if {condition}` `:else` `:elseif` and `:endif`.
You can use `a > 1 ? 'big' : 'small'`.
String what does not start with number, when converted to number is always 0
which means `false`

~~~
: if 0
:   echom "not possible"
: elsif "also_zero0"
:  echom "also 0"
: elsif "1one" - 1
:   echom "also zero value"
: elsif "zero1" + "1one"
:   echom "finally"
: else
: end
~~~

Vim is by default ignorecase but it can be changed. There are case insensitive
(append `?`) and case sensitive (append `#`) equal operator regardless of
`ignorecase` setting (so you use that in your scripts `:help expr4`):

~~~
:if "Asd" == "asd" | echo "case insensitive equal" | end
:set noignorecase
:if "Asd" != "asd" | echo "case sensitive different" | end

:if "Asd" ==? "asd" | echo "case insensitive equal" | end
:if "Asd" !=# "asd" | echo "case sensitive different" | end
~~~

String match `a =~ b` and not match `a !~ '\.$'` (does not end with full stop).
Also supports case insensitive `!~?` and case sensitive `!~#`.

# Function

Functions must start with capital if they are unscoped.
Parameters are always pefixed with `a:` (argument scope).

~~~
:function MyFuction(name)
: let result = "hi " . a:name
: return result
:endfunction

:call MyFunction("Duke")
:echom MyFunction("Duke")
~~~

There are many functions `:help functions`
Working with text in current buffer:
* `getline('.')` get the line under the cursor, `setline('.', new_line)` set

String manipulation
* `substitute({expr}, {pattern}, {sub}, {flags})`
* `strlen("foo")`
* `split("one,two,three", ",")` and `join(["one","two"], "...")`

List manipulation
* `map({list}, {expression})` change each list item with expression which should
  be a string or function. If it is a string than `v:val` is containing item
  `:call map(mylist, '"> " . v:val . " <"')`
  When it is a function than you can define function
  ```
  :func KeyValue(key, val)
  :  return a:key . '-' . a:val
  :endfunc
  :call map(myDict, function('KeyValue'))
  ```
  It is shorter when using a |lambda|: `:call map(myDict, {key, val -> key . '-'
  . val})`

Define function that is called with a range with `range` keyword
```
:function Count_lines() range
:  return a:lastline - a:firstline
:endfunction

" call with
:10,30call Count_lines()
```

Define function on dictionary so you can use it with `self` variable. `get(self,
v:val, '???')` return item if exists, otherwise `'???'`.
```
:let uk2nl = {'one': 'een', 'two': 'twee', 'three': 'drie'}
:function uk2nl.translate(line) dict
:   return join(map(split(a:line), 'get(self, v:val, "???")'))
:endfunction

:echo uk2nl.translate('three two five one')
dire twee ??? een
```

You can define arbitrary number of arguments with `function Show(start, ...)` so
inside it `a:0` is count of those arguments `a:1`, `a:2` ...

To list all functions
```
:function
```

To show function defition
```
:function Min
```

Debug `:help debug-scripts`
Delete function `:delfunction Min`.
Store function in variable `let Afunc = function('Min')` and call using call
function `:echo call(Afunc, [])`
In script you can use global variable to know if functions are loaded so you can
undefine them (or you can simply call `finish` to exit)
```
" This is the XXX package

if exists("XXX_loaded")
  delfun XXX_one
  delfun XXX_two
endif

function XXX_one(a)
... body of function ...
endfun

function XXX_two(b)
... body of function ...
endfun

let XXX_loaded = 1
```

# Lists and dictionaries

List is ordered sequence of things
```
:let alist = ['a', 1]
```
add one item using `add()` function, add multiple items with `extend()`,
concatenate with `+`
```
:call add(alist, 'foo')
```

Dictionary store key-value
```
:let dict = {'one': 1. 'two': 2}
```
use them like index `dict['one']` or `dict.one`. Assign new key
```
:let dict['three'] = 3
```
Retrieve keys with `:let keys = keys(dict)`

# Exceptions

```
:try
:  read pascal.tmpl
:catch /E484:/
:  echo 'Sorry, pascal.tmpl can not be found'
:finally
:  echo 'This is always executed
:endtry
```

# Comments

Commane is after double quote till the end of line
* There can be no comment after `:map`, `:abbreviate`, `:execute` and `!`
  You can add `|` so there are two commands

# Command line ex special characters

`:help cmdline-special`
In command line you can get work under cursor with `<cword>` inside `:grep`
which is quickfix command along with `:make`.

For `:echo` we need to `expand()` string to see the value. Echo is different
from :grep in terms that it works with strings and we can use functions (similar
to `:execute`)
```
:nnoremap <leader>g :grep -R '<cWORD>' .<cr>
:echo expand('<cWORD>')
:execute ':echo 2 ' shellescape(expand('<cWORD>'))
:nnoremap <leader>g :silent execute "grep! -R " . shellescape(expand("<cWORD>")) . " ."<cr>:copen<cr>
```

# Plugins

If you need to export function so user can use mappings than you need to use
`<SID>MyFunc()` (instead of `s:MyFunc()`) so vim know which script to use. For
other place is it ok to use `s:MyFunc()` in autocommands, user commands...
Example of plugin in `~/.vim/plugin/typecorr.vim`
```
" Vim global plugin for correcting typing mistakes
" Last Change:	2000 Oct 15
" Maintainer:	Bram Moolenaar <Bram@vim.org>
" License:	This file is placed in the public domain.

if exists("g:loaded_typecorr")
  finish
endif
let g:loaded_typecorr = 1
let s:save_cpo = &cpo
set cpo&vim
iabbrev teh the
iabbrev otehr other
iabbrev wnat want
iabbrev synchronisation
	\ synchronization
let s:count = 4

let s:count = 4
if !hasmapto('<Plug>TypecorrAdd')
  map <unique> <Leader>a  <Plug>TypecorrAdd
endif
noremap <unique> <script> <Plug>TypecorrAdd  <SID>Add

noremenu <script> Plugin.Add\ Correction      <SID>Add
noremap <SID>Add  :call <SID>Add(expand("<cword>"), 1)<CR>

function s:Add(from, correct)
  let to = input("type the correction for " . a:from . ": ")
  exe ":iabbrev " . a:from . " " . to
  if a:correct | exe "normal viws\<C-R>\" \b\e" | endif
  let s:count = s:count + 1
  echo s:count . " corrections now"
endfunction
if !exists(":Correct")
  command -nargs=1  Correct  :call s:Add(<q-args>, 0)
endif
let &cpo = s:save_cpo
unlet s:save_cpo
```

  33

TODO 
https://github.com/mhinz/vim-galore

TODO read and merge to the text

* functions can be used to wrap some commands and give it a name.
  Use `function!` so you can overwrite it without error. Undo will undo whole
  function at once.

  ~~~
  function! MyFuction()
    normal! mmu`m
  endfunction
  nnoremap <leader>sp :call MyFuction()<cr>
  ~~~
* source current file while editing with `:source %` or using sop `nnoremap
  <leader>sop :source %<cr>` so you do not need to exit and start vim again. You
  can reload any `.vim` file, for example `:source ~/config/vim/syntastic.vim`
* conditionals

  ~~~
  function! MF(level)
    if a:level == 1
      normal! yy
    elseif a:level == 2
      " ....
    endif
  endfunction
  ~~~


# Shell script

Calling external shell script is with bang `:!pwd`. When you visually select it
operates on lines and use it as argument, for example select multuple lines and
run `:'<,'>!sort` to sort them.
If nothing is selected, last selected visual line (whole line) is used.
You can use `cat` to pass argument. For example run `:'<,'>!echo 1`cat`2`, and
it will replace whole line by inserting 1...line...2.
Another way to pass is using `system("echo ", @")` unnamed register.

To work on part of a line, you can use visual match replacement
`:'<,'>s/\%V.*\%V/\=system('echo -n "the result"')`.
To use visual selected text as inputs, you can do it manually by pasting with
`<C-R>` + `"` (" is a standard register for yanking).
Or you can use helper function that will yank to register and get register
https://stackoverflow.com/questions/12805922/vim-vmap-send-selected-text-as-parameter-to-function
```
func! GetSelectedText()
  normal gv"xy
  let result = getreg("x")
  normal gv
  return result
endfunc

vnoremap <F6> :call MyFunc(GetSelectedText())<cr>
```

https://stackoverflow.com/questions/9637921/vim-filter-only-visual-selection-not-the-entire-line


# Translate

Run scripts on visual selection and change with output. I use `translate.rb` and
`sudo ln -s /home/orlovic/config/ruby/translate.rb /usr/local/bin/` and
`chmod +x ~/config/ruby/translate.rb` https://github.com/duleorlovic/config/blob/master/ruby/translate.rb
In vim there is a [ProgramFilter function](https://github.com/duleorlovic/config/blob/master/.vimrc#L482)

There is an issue with ending space (at the end of a line) when pasting. Same
problem is when selecting word and super + p
