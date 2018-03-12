---
layout: post
title: Vim theory
---

# Internals

keynames:

* `<c-d>` is control Ctrl + d, in help it can be written as `CTRL-D`
* `<esc>` is escape key
* `<space>` space
* `<cr>` sometimes it is `<Enter>`
* `<bs>` backspace
* `<left>` left arrow
* `<s-left>` shift left
* `<c-left>` control left
* `^I` is when we insert `<tab>`
* `<nl>` new line. If you use `echom "a\nb"` you will see `a^@b` (`CTRL-v` +
  `CTRL-j`)
* `<tab>` tab

special variables

* `<leader>` by default is `\ ` but I use `<space>`. To check current value use:
`:echo mapleader`, to set use `:let mapleader=" "`
* `<localleader>`
* `<non-keyword-character>` maps all non keyword chars: `:set iskeyword?`
  returns `iskeyword=@,48-57,_,192-255,-` ie alphabetic ascii, digits 0-9, _ ,
  and some special chars
* `<nop>` no operation, usefull when you want to disable some keys

# Lean Vim Script The Hard Way

<http://learnvimscriptthehardway.stevelosh.com/>

## Mappings

maping keys `:noremap <newkeys> <keynames>`

* since mappings easilly could be broken, ALWAYS use `nore` non recursive
variant `noremap`, `nnoremap` ... so it use default meanings of keynames
* you can map control CTRL key, for example `:map <c-d> dd`
* do not use comments `"` in mapping
* `nmap` normal, `vmap` visual, `imap` insert, `omap` operator, `cmap` command
mode.
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

`:iabbrev adn and` this will replace `adn<non-keyword-character>` with `and`
Difference between `map` is that map does not count for non-keyword-character

`:iabbrev <buffer> --- &mdash;` will replace `---` with `&mdash` but only for
current buffer. It is buffer local abbreviation and is usefull when you want
something only for specific type, for example `:autocmd FileType javascript
:iabbrev <buffer> iff if ()<left>` will enable little snippet only for js.

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

* print env variables on command line: `:echo $MYVIMRC`, `:echom $PATH` will
save output into `:messages` so you can view later
* to set variable use `:let foo = "bar"` and pring `:echo foo`
* to set option you can use `:set xxx` or `:set xxx=value` or `:let &xxx=value`
  To unset `:set noxxx` or `:set xxx!`. To check if it is set `:set xxx?` or
  `:echo &xxx`. For boolean options `1` is true and `0` is false. `:let` is more
  powerfull than `:set` since it can use math and other operations.
  If you have space inside value than you need to escape `set xxx+=my\ value`
* to set registers run `let @a = "hello"` so you can paste `"ap`. To read
  register run `:echo @a` (`:echo @"` is unnamed yank register, `:echo @\ ` is
  search register ... `:help registers`).
* to get current value you can use `=` and `<TAB>` for example `:set xxx=` and
press tab so vim autocomplete the value so you can edit it

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
