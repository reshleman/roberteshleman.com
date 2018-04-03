---
layout: post
title: "Learning to Work Efficiently with Vim"
date: 2014-07-18 14:31:09 -0400
comments: true
redirect_from: /blog/2014/07/18/learning-to-work-efficiently-with-vim/
categories: ["tech", "vim", "rails", "ruby", "metis"]
---

*This summer, I'm learning Ruby on Rails at [Metis](http://www.thisismetis.com), a 12-week class taught by some great folks from [thoughtbot](http://www.thoughtbot.com). This post is part of a series sharing my experience and some of the things I'm learning.*

Until a few months ago, [Vim](http://en.wikipedia.org/wiki/Vim_%28text_editor%29) was a program I had never considered using. I had always perceived Vim to be an editor with a steep learning curve that wasn’t worth my time as a web programmer. It was a tool, I thought, used mostly by people who spent their time writing obscure shell scripts or coding the Linux kernel in C.

As it turns out, I was wrong, and I’m glad I was.

<!-- More -->

## Overcoming the Learning Curve

Now, that’s not to say that Vim’s learning curve isn’t steep — the program can be notoriously unforgiving to a new user. But once I learned a few basic commands to move around, and understood the [different modes](http://vimdoc.sourceforge.net/htmldoc/intro.html#vim-modes-intro), Vim became much easier to work with.

Encouraged by one of my instructors at Metis, I spent up to an hour at the beginning of each day practicing with [vimtutor](http://linuxcommand.org/man_pages/vimtutor1.html). (There are also a number of [web-based](http://vim-adventures.com/) [tutorials](http://www.openvim.com/tutorial.html), and this [keyboard layout cheat-sheet](http://www.viemu.com/vi-vim-cheat-sheet.gif).)

After using vimtutor each morning, I then challenged myself to code in Vim throughout the day. At first, I worked much more slowly as I learned different commands and occasionally googled for help. After a few days, though, I could tell that I was beginning to get the hang of it. It was amazing to see the impact of using they keyboard to navigate, without switching over to the mouse every few minutes.

## Hard Work’s Rewards

Having now spent a few months using Vim, I’m really beginning to appreciate its power. Using [MacVim](https://code.google.com/p/macvim/) and a few extensions (for syntax highlighting, file-switching, *et al.*), I now have most of the key features of a text editor like [Sublime](http://www.sublimetext.com/) and still have the additional power of Vim. Overall, my workflow is much more efficient.

Needless to say, Vim has quickly become an indispensable tool, even (especially?) for a web developer like me.

## A Not-Nearly-Exhaustive List of Vim Magic

Below are just a few of the Vim shortcuts that I’ve found especially useful as I’ve been learning Rails.

### Vanilla Vim Shortcuts

* `c` is for “change”; it deletes in the given direction then enters insert mode (i.e., `cw` to change to the end of the word, or `c$` to change to the end of the line)
* `d` deletes in the given direction (i.e., `dw` to delete to the end of the word); `D` deletes to the end of the line
* `o` places you in insert mode in a new line below the current line; `O` does the same above the current line
* `:!` lets you run a terminal command from within Vim, helpful for running Ruby files (like `:!rb filename.rb`)
* `=` automatically indents lines in the given direction. `gg=G` automatically indents the whole file.

### Multi-Window Commnds

* `:split` for a horizontal window split (optionally append a filename to open)
* `:vsplit` for a vertical window split (optionally append a filename to open)
* `Ctl-W` commands allow moving between split windows
* `Ctl-W Ctl-W` switches between active split windows
* `Ctl-W [h,j,k,l]` moves to the split window in the given direction

### Vim Rails Magic

*This magic comes courtesy of the `rails.vim` extension, available [here](https://github.com/tpope/vim-rails).*

* `:Rinitializer` jumps to the routes file
* `:Rmigration` jumps to the most recent migration file
* `:R` from inside a view jumps you to the corresponding action within the appropriate controller and vice-versa
* `:Rcontroller wombats` jumps to the “wombats” controller
* `:Rview wombats/index.html.erb` does the same for the view
* `:Rmodel wombat` does the same for the model
* Append `!` to any of the last three to create the file with a boilerplate class definition

### File Switching

* `:e` followed by a path/filename switches to a different file
* `Ctl-p` (via this [extension](https://github.com/kien/ctrlp.vim)) has been a **huge** time-saver with its fuzzy filename matching.

### Others?

This is just a short list. Feel fee to tweet me your other favorites [@RobertEshleman](https://twitter.com/RobertEshleman) or comment below. =)
