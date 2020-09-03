想要查找 vim 中的快捷键绑定，可以使用 `:verbose map <C-a>`

比如我有一次进行了如下设置:

```
map <C-a> 0
```

目的是为了方便我跳转到行首时手指不要移动太远，但我忽略了 `<C-a>` 的自动加法功能，有一天我使用时，无论如何都不能自动加法，第一反应就是 keymap 冲突，键入 `:verbose map <C-a>` 后，便显示如下：

```
   <C-a>       * 0
        Last set from ~/.vim/base.vim line 172
```

可以看到， 在 `~/.vim/base.vim` 中，我将 `<C-a>` 映射到 `0`，然后我移除了这个 keymap 就可以正常使用了。
