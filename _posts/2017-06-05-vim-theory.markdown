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
  noxxx` or `:set xxx!`.  To check if it is set `:set xxx?`. To see current
  value you can use `=` and `<TAB>` for example `:set xxx=` and press tab so vim
  autocomplete the value so you can edit it
* `:help xxx` or `:help 'xxx'`

## Mappings

maping keys `:noremap <newkeys> <keynames>`

* since mappings easilly could be broken, ALWAYS use `nore` non recursive
variant `noremap`, `nnoremap` ... so it use default meanings of keynames
* you can use control CTRL `:map <c-d> dd`
* do not use comments `"` in mapping
* `nmap` normal, `vmap` visual `imap` insert mode only mappings, for example
in insert mode `:map <esc>ddi`
* remove mappings `:unmap <key>` or `nunmap`
* you can map two keys, for example `:map -d dd`
* change leader key `:let mapleader = "-"` so you can write `:map <leader>d dd`
(note that it does not have effect for already defined mappings)
* `:let maplocalleader = "\\"` can be used for mapings only for local buffer.
Mapping can be for specific buffer (file) `:nnoremap <buffer> <leader>d dd`. You
can use both `:nnoremap <buffer> <localleader>d dd`
* `:setlocal nowrap` is only for current buffer (file), when you open another
file (inside same window) it will not have nowrap. If you have same mappings
than local will be chosen since it is more specific

## Abbreviations

Abbreviations are similar to mappings but for insert, replace and command mode

* `:iabbrev adn and` this will replace `adn<non-keyword-character>` with `and`
Difference between `map` is that map does not count for non-keyword-character

## Autocommands

Autocommands are used to run commands on specific events

`:autocmd BufNewFile * :write`.

First param is event and can be something of: entering insert mode, not pressing
a key for some time...

* `BufNewFile` when you open new files not saved to disk
* `BufRead` after you open existing file
* `FileType` when vim sets filetype
  * `:autocmd FileType javascript nnoremap <buffer> <localleader>c I//<esc>`

Second param is "pattern" that filter more specific: for example: `*.txt` is
only triggered for txt files

14
