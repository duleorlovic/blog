---
layout: post
title: Vim theory
---

# Lean Vim Script The Hard Way

<http://learnvimscriptthehardway.stevelosh.com/>

## Mappings

maping keys syntax `:noremap <newkeys> <keynames>`

* list all mappins `:map`
* since mappings easilly could be broken, ALWAYS use `nore` non recursive
variant `noremap`, `nnoremap` ... so it use default meanings of keynames
* you can map control CTRL key, for example `:map <c-d> dd`
* do not use comments `"` in mapping
* `nmap` normal, `vmap` visual, `imap` insert, `omap` operator, `cmap` command,
  `xmap` ex command mode.
Dor example in insert mode `:imap <esc>ddi`
* remove mappings `:unmap <key>` or `nunmap`
* you can map two keys, for example `:map -d dd`
* change leader key `:let mapleader = "-"` so you can write `:map <leader>d dd`
(note that it does not have effect for already defined mappings)
* `:let maplocalleader = "\"` can be used for mapings only for local buffer.
Mapping can be for specific buffer (file) `:nnoremap <buffer> <leader>d dd`. You
can use both `:nnoremap <buffer> <localleader>d dd`

## Abbreviations

Abbreviations are similar to mappings but for insert, replace and command mode

* `:iabbrev adn and` this will replace `adn<non-keyword-character>` with `and`
Difference between `map` is that map does not count for non-keyword-character

* `:iabbrev <buffer> --- &mdash;` will replace `---` with `&mdash` but only for
current buffer. It is `<buffer>` local abbreviation and is usefull when you want
something only for specific type, for example `:autocmd FileType javascript
:iabbrev <buffer> iff if ()<left>` will enable little snippet only for js.

* `cabbrev E e` can be used to replace `E ` with `e ` on command line so when
  you type `:E<space>` it will end up in `:e<space>`

## Autocommands

Autocommands are used to run commands on specific events

`:autocmd BufNewFile * :write`.

First param is event and can be something of: entering insert mode, not pressing
a key for some time. It can can combine, usually `BufNewFile,BufRead` for all
open files, existing or not. Some events `:help autocmd-events`

* `BufNewFile` when you open new files not saved to disk
* `BufRead` after you open existing file
* `BufWritePre` just before write
* `FileType` when vim sets filetype
  * `:autocmd FileType javascript nnoremap <buffer> <localleader>c I//<esc>`

Second param is "pattern" that filter more specific: for example: `*.txt` is
only triggered for txt files

Last part is command that is executed.

You can group autocommands (`autocmd!` is to clear)

~~~
augroup filetype_html
    autocmd!
    autocmd FileType html nnoremap <buffer> <localleader>f Vatzf
augroup END
~~~

## Operator pending mappings

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

`:normal!` means that it simulate pressing in normal mode. `<cr>` at the end
executes `:normal!` command. You can indent from command mode: `:normal >>`
`<c-u>` needs to remove the range that vim may insert.

`:execute` takes vimscript string and performs as command, `:execute "write"`.
It is used since `:normal` can not recognize special characters like `<cr>`.
`execute` will substitute any special charactes before running string, special
chars are `:help expr-quote`
* `\r` return `<CR>`
* `\n` new line `<NL>`
For example change current title header in markdown text with `cih`:

~~~
:onoremap ih :<c-u>execute "normal! ?^==\\+$\r:nohlsearch\rkvg_"<cr>`
~~~

## Status Line

`:set statusline=%f\ -\ FileType:\ %y` will change status line to show

* `%f` path to the file
* `%F` full path to the file
* `%y` filetype of the file
* `%L` total lines

## Variables

* print env variables on command line: `:echo $MYVIMRC` or `:echo filetype?`.
 `:echom $PATH` will save output into `:messages` so you can view later
* to set variable use `:let foo = "bar"` and pring `:echo foo`

* to set option you can use `:set xxx` or `:set xxx=value` or `:let &xxx=value`
  To unset `:set noxxx` or `:set xxx!`. To check if it is set `:set xxx?` or
  `:echo &xxx`. To set to default value use ampersand `&` like `:set iskeyword&`
  For boolean options `1` is true and `0` is false. `:let` is more
  powerfull than `:set` since it can use math and other operations.
  You can add or remove item from list, you can do with `:set k+=v` or `-=`.
  If you have space inside value than you need to escape `:set xxx+=my\ value`
* to set registers run `let @a = "hello"` so you can paste `"ap`. To read
  register run `:echo @a` (`:echo @"` is unnamed yank register, `:echo @\ ` is
  search register ... `:help registers`).
* to get current value you can use `=` and `<TAB>` for example `:set xxx=` and
press tab so vim autocomplete the value so you can edit it. Or you can press
enter after the name (skip equal sign), for example `:se ft<cr>`

Local scope variables are with prefix like `:let b:a=2` (`b:` buffer, `w:`
window, `t:` tab, `g:` global, `l:` local to function, `a:` argument)

`:setlocal nowrap` is only for current buffer (file), when you open another
file (inside same window) it will not have nowrap. If you have same mappings
than local will be chosen since it is more specific.

Multiline commands can be written as:

~~~
:echom "foo" | echom "bar"

" or

:augroup testgroup
:   autocmd BufWrite * :echom "Baz"
:autgroup END
~~~

Conditional

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

Vim is by default ignorecase but it can be changed. There are casa insensitive
and case sensitive equal operator regardless of `ignorecase` setting (so you use
that in your scripts `:help expr4`):

~~~
:if "Asd" == "asd" | echo "case insensitive equal" | end
:set noignorecase
:if "Asd" != "asd" | echo "case sensitive different" | end

:if "Asd" ==? "asd" | echo "case insensitive different" | end
:if "Asd" !=# "asd" | echo "case sensitive equal" | end
~~~

Functions must start with capital if they are unscoped.
Parameters are always pefixed with `a:` (argument scope).

~~~
:function MyFuction(name)
: let result = "hi " . a:name
: return result
:endfunction

:call MyFunction("Duke") " when there is no return. default return is 0
:echom MyFunction("Duke")
~~~

Variable types can be:

* numbers `100`, `0xff` (hex representation), `010` (octal representation)
* float `100.1`, `1.001e+2` (e notation, decimal is required). For division to
  work in float you need to use float as one of params (type cast, coercion of
  second param is automatic) `echo 3/2.0`
* string `"asd"`, Concatenation is with dot `.` (`+` is only for numbers).
  Special characters: `\\`, `\"`, `\n`

  26

TODO 
https://github.com/mhinz/vim-galore

TODO read and merge to the text
Also `:help usr_41`, or `:help eval.txt` and a

* `:nnoremap _ f_x~` find next `_`, remove and uppercase
  [link](http://vim.wikia.com/wiki/Converting_variables_to_or_from_camel_case)
  for moving underscore to CamelCase
* snippet `nnoremap ,html :-1read $HOME/.vim/.skeleton.html<CR>3jwf>a` [my
snippets](https://github.com/duleorlovic/config/tree/master/vim/snippets)
* `<silent>` is good if you are having mappings launch ex commands.
* black screen is caused to silent command
  https://github.com/vim/vim/issues/1253 use `:redraw!`


* `nnoremap ,html :asd` first letter `n` means that this applies only in normal
  mode, `no remap` means do not reinvoke if those commands `,html` are used for
  something else, `map` means simply when I type `,html` please type `:asd`
  Other modes are: `cmap` control.
* `normal!` (with bang) means execute this exactly as I write (`normal` will not
  ignore mappings that user could have written and destroy your commands)
* functions can be used to wrap some commands and give it a name.
  Use `function!` so you can overwrite it without error. Undo will undo whole
  function at once.

  ~~~
  function! MyFuction()
    normal! mmu`m
  endfunction
  nnoremap <leader>sp :call MyFuction()<cr>
  ~~~
* source current file `nnoremap <leader>sop :source %<cr>` is used to source
  .vimrc so you do not need to exit and start vim again. You can reload any
  `.vim` file, for example `:source ~/config/vim/syntastic.vim`
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


# See above indent

similar but does not work for rspec
http://vim-taglist.sourceforge.net/installation.html `:TlistToggle`
