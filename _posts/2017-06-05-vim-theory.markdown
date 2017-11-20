---
layout: post
title: Vim theory
---

# Internals

keynames:

* `<c-d>` is control Ctrl + d, in help it can be writter as `CTRL-D`
* `<esc>` is escape key
* `<space>` space
* `<cr>`, sometimes it is `<Enter>`
* `<bs>` backspace
* `<left>` left arrow
* `<s-left>` shift left
* `<c-left>` control left
* `^I` is when we insert `<tab>`
* `<nl>` (`^@` is `CTRL-v` + `CTRL-j`)

special variables

* `<leader>` by default is `\ ` but I use space. To check current value use:
`:let mapleader`, to set use `:let mapleader=" "`
* `<localleader>`
* `<non-keyword-character>` maps all non keyword chars: `:set iskeyword?`
  returns `iskeyword=@,48-57,_,192-255,-` ie alphabetic ascii, digits 0-9, _ ,
  and some special chars
* `<nop>` no operation, usefull when you want to disable some keys

# Lean Vim Script The Hard Way

<http://learnvimscriptthehardway.stevelosh.com/>

* print variables on command line: `:echo $MYVIMRC`, `:echom` will save output
  into `:messages` so you can view later
* to set variables you can use `:set xxx` or `:set xxx=value`. To unset `:set
noxxx` or `:set xxx!`.  To check if it is set `:set xxx?`.
* to get current value you can use `=` and `<TAB>` for example `:set xxx=` and
press tab so vim autocomplete the value so you can edit it
* `:help xxx` or `:help 'xxx'`

## Mappings

maping keys `:noremap <newkeys> <keynames>`

* since mappings easilly could be broken, ALWAYS use `nore` non recursive
variant `noremap`, `nnoremap` ... so it use default meanings of keynames
* you can map control CTRL key, for example `:map <c-d> dd`
* do not use comments `"` in mapping
* `nmap` normal, `vmap` visual, `imap` insert, `omap` operator mode.
Dor example in insert mode `:imap <esc>ddi`
* remove mappings `:unmap <key>` or `nunmap`
* you can map two keys, for example `:map -d dd`
* change leader key `:let mapleader = "-"` so you can write `:map <leader>d dd`
(note that it does not have effect for already defined mappings)
* `:let maplocalleader = "\"` can be used for mapings only for local buffer.
Mapping can be for specific buffer (file) `:nnoremap <buffer> <leader>d dd`. You
can use both `:nnoremap <buffer> <localleader>d dd`
* `:setlocal nowrap` is only for current buffer (file), when you open another
file (inside same window) it will not have nowrap. If you have same mappings
than local will be chosen since it is more specific

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

# Operator pending mappings

Operator is command waiting for movement command and than executes. For example
`dt,` (delete till ,), `ci(` (change inner (), `yw` (yank word). We can remap
keys for operator mode `:onoremap p i(` (inner braces, select parametars), or
`:onoremap b /end<cr>` (block, until end). If you want to select also before
current position you can visualy select and it will be executed on all.
For example, change params while current is on current line before params.

~~~
onoremap in( :<c-u>normal! f(vi(<cr>
~~~

`normal!` means that it simulate pressing in normal mode.

`:onoremap ih :<c-u>execute "normal! ?^==\\+$\r:nohlsearch\rkvg_"<cr>`

17
