## 原理

先将文件名通过 `r !ls` 等命令读入到 vim 中，然后将每行改为 `mv <原文件名> <新文件名>` 后（适用于 Linux 及 MacOS），运行 `:w !sh` 逐行执行命令进行重命名。

## 实操

### 1. 环境创建

```bash
for i in {1..20}
do
touch $i.txt
done
```

运行后使用 `ls` 可以看到当前目录出现如下一些文件：

```bash
➜ ls
1.txt  10.txt 11.txt 12.txt 13.txt 14.txt 15.txt 16.txt 17.txt 18.txt 19.txt 2.txt  20.txt 3.txt  4.txt  5.txt  6.txt  7.txt  8.txt  9.txt
```

### 2. 打开 vim

直接运行 `vim` 打开 vim 即可（不用打开文件）

### 3. 读取当前目录下的文件名到 vim 中

在 vim 命令模式下输入：

```
:r !ls
```

### 4. 构建 mv 语句

#### 使用替换方式构建 mv 语句：

```
:%s/\(.*\).txt/mv & \1.png/g
```

以上替换语句 **可能** 的一个替换效果为将 `1.txt` 替换为 `mv 1.txt 1.png` 。其中，`\( \)` 之间包含的是一个正则表达式的组（不懂该概念的自行百度），`&` 和 `\0` 是一个作用，`\1` 代表第一个组，`\0` 代表整个匹配到的串。

#### 使用宏的方式构建 mv 语句：

1. 使用宏编程的方式，先将每行当前的文件名复制一份，如 `1.txt 1.txt`
2. 再使用宏编程的方式，将第 2 个 `1.txt` 改为 `1.png`
3. 在文档顶部使用 `<C-v> G` 快捷键块选择，然后用 `I` 在行首位置输入 `mv` （注意此处 mv 之后有一个空格）

经过上面的构建， **可能** 存在这样一行 `mv 1.txt 1.png` 。

### 5. 批量执行语句

在 vim 命令模式下输入：

```
:w !sh
```

### 6. 退出 vim

此时，可以使用 `q!` 退出。