---
layout: post
title: Vim theory
---

# Internals

keynames:

* `<c-d>` is control Ctrl + d
* `<esc>` is escape key
* `<space>` space
* `<cr>` enter
* `<bs>` backspace

special variables

* `<leader>` by default is `\ `
* `<non-keyword-character>` maps all non keyword chars: `:set iskeyword?`
  returns `iskeyword=@,48-57,_,192-255,-` ie alphabetic ascii, digits 0-9, _ ,
  and some special chars

# Lean Vim Script The Hard Way

<http://learnvimscriptthehardway.stevelosh.com/>

* print variables on command line: `:echo $MYVIMRC`, `:echom` will save output
  into `:messages` so you can view later
* to set variables you can use `:set xxx` or `:set xxx=value`. To unset `:set
  noxxx` or `:set xxx!`.  To check if it is set `:set xxx?`
* `:help xxx` or `:help 'xxx'`
* maping keys `:noremap <newkeys> <keynames>`
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
  * `:let maplocalleader = "\\"` mapings only for local buffer
* abbreviations are similar to mappings but for insert, replace and command mode
  * `:iabbrev adn and` this will replace `adn<non-keyword-character>` with `and`
  so that is difference between `nmap` since `map` does not count for
  non-keyword-character
