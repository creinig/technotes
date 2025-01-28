# Vim documentation

* [A Byte of Vim](https://vim.swaroopch.com/) : good introduction to vim
* [Vim Galore](https://github.com/mhinz/vim-galore#readme) : concise but complete and well-written overview
* [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
* [VimCasts](http://vimcasts.org/) : screencasts about vim. also check out the author's book "[Practical vim](https://pragprog.com/book/dnvim2/practical-vim-second-edition)"!
* [Vim Awesome](https://vimawesome.com/) : awesome vim plugins
* [Learn Vimscript the Hard Way](https://learnvimscriptthehardway.stevelosh.com/) : how to extend and customize vim

# Useful Vim tips

## Scrolling cheat sheet

From user MacMartin at https://stackoverflow.com/a/60607822/1814922

```
+-------------------------------+
^                               |
|c-e (keep cursor)              |
|H(igh)             zt (top)    |
|                   ^           |
|           ze      |      zs   |
|M(iddle)  zh/zH <--zz--> zl/zL |
|                   |           |
|                   v           |
|L(ow)              zb (bottom) |
|c-y (keep cursor)              |
v                               |
+-------------------------------+
```

## Navigating buffers

```
:bp[rev] <-- :b <Tab> | :b<n>  --> :bn[ext]

previous     choose   | select     next
```

* :bd[elete] : close buffer
* :ls : list buffers

