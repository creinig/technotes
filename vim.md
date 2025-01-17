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

