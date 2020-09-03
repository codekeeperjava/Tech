

## 快捷键

- Command + shift + p : Command Palette
- Command + p : 打开最近文件
- Command + shift + o : 打开文件结构
- Commend + shift + space : 提示参数
- Shift + F12 : 查看被用到的地方
- F12 : 调到变量定义处
- Command Palette - Rename Symbol : 重命名变量
- Command + / : 添加注释或取消注释
- Command + B : 显示/隐藏Active Bar
- Command + J : 显示/隐藏Panel Bar
- Command + 编号 : 在Editor Group间切换
- Command + \ : 创建Editor Group
- Command + Option + 0 ： Editor Group垂直排列
- Command + Option + 方向键 : 在Editor中左右移动
- Tab + Control : 在Editor之间切换
- Command + K, Command + 0 : 折叠所有可以折叠的代码
- Command + K, Command + J : 展开所有折叠代码
- Control + - : Back
- Control + shift + - : Forward



## 功能

### codesnippet

- `> Preferences: Configure User Snippets` - 选择语言，如python
- 在打开的python.json中修改`prefix`和`body`，`body`每一个元素代表一行
- `$1`表示光标停留的第一个位置，匹配时按Tab调到`$2`处，它有一个洋气的名字叫**Tab Stop**。`${1:label}`表示光标停留到第一个位置时，会出现label字符并被选中，当一个文档中有多个相同的Tab Stop时，会同时出现多光标的情况，可以批量修改。

### breadcrumb

其实就是导航栏，但Command Palette下查找还是得用这个，记住叫**Breadcrumb**

![2020071913OzbBT90d420c33.png](http://img.dotleo.cn/blog/2020071913OzbBT90d420c33.png)

### minimap

右上角可以同时显示很多内容的地方

![2020071913Xd8n3G6f6ecf1c.png](http://img.dotleo.cn/blog/2020071913Xd8n3G6f6ecf1c.png)

### 搜索和替换

Cmd + F : 单文件内搜索，**注意**：如果选定部分内容搜索时，需要点击❎号左边的按钮。

Cmd + option + F : 单文件内替换

Cmd + shift + F : 多文件搜索

### Editor相关的几个个性化设置

*可以通过设置搜索找到*

1. Editor:Line Numbers : 设置行号为相对行号或绝对行号或取消
2. Editor: Render Whitespace : 显示空格符
3. Editor: Render Indent Guides : 显示缩进参考线，缩进参考线就是代码块中起始和结束位置的垂直连线
4. Editor: Rulers : 每行字符限定线，比如设置为120，`"editor.rulers": [120]`

![2020071914yTX4Ng4a182f99.png](http://img.dotleo.cn/blog/2020071914yTX4Ng4a182f99.png)

### workbench

显示和隐藏在VIEW - Appearance中设置

![2020071920VI9G0d3a54bd6b.png](http://img.dotleo.cn/blog/2020071920VI9G0d3a54bd6b.png)

### Command Palette

呼出，`Cmd + p`

- `>`:  所有命令
- `:`：跳转到某行
- `@`：文件中的符号
-  `#`：工作区的符号
- 还有一些英文缩写，可以使用`?` 查看

### 文件管理

VS Code所有操作都是**基于已打开的文件或者文件夹的**，可以类比理解IDE以项目为文件管理的目标。一个有趣的细节，当没有文件夹打开时，状态栏呈紫色，打开文件夹后会变为蓝色。

![2020071915cnC6cPc6492dab.png](http://img.dotleo.cn/blog/2020071915cnC6cPc6492dab.png)

VS Code对项目（文件夹）的管理，会生成一个`.vscode`文件夹。该文件夹中的`settings.json`用于存储该项目中的配置文件，几乎和用户设置一样，但它的优先级高，比如一个项目中用2个空格代替Tab，写在这个文件中，所有项目成员就能共享，不用强迫别人其他项目也如此。另外2个常见的文件：一个是调试设置(launch.json)，一个是任务设置(task.json)。

### WorkSpace

VS Code基于文件夹管理，但有人就是想在一个窗口中打开两个文件夹，通常我是喜欢每个文件夹打开一个独立的窗口，打开一个独立窗口可以使用`Cmd + shift + o`，这样在mac下可以快速用 *Cmd + `* 进行切换。

如果非要在一个窗口打开两个文件夹，可以使用 *workspace* 概念完成。

Command Palette - `Workspaces: Add Folder to Workspace...`即可将一个文件夹添加到workspace。

此时这两个文件夹位于一个名为 *UNTITLED(WORKSPACE)* 下，可以 Command Palette - `Workspaces: Save Workspace As...`然后存储该workspace，方便日后打开。

### Git

一般安装git history和gitlens插件，功能自己摸索更好。

### workflow(task)

该功能主要是定义一个task，比如像`mvn install`这样的命令，它会保存在`.vscode/task.json`中。这个功能比较复杂，感觉一般人用不到，可以跳过，如果有需要，可以再学。如果我们想快速执行一个任务，可以直接写python脚本或bash脚本，该功能主要强大在于和编辑器的互动。

参考网盘中的《玩转VS Code》。

### Debug

debug和平时idea中使用的基本一样，看看就会。点击调试按钮，项目中默认是没有`launch.json`文件的，意味着你需要选一个预置的调试器及默认的配置，你也可以通过`create launch.json`文件创建一个，然后填充默认配置，这样就可以直接点击调试按钮进行调试。

可以在`launch.json`中配置*compounds* 属性将多个 *configurations* 同时启动起来调试。

如果想学更高深的，可以进一步学习。

参考网盘中的《玩转VS Code》。

### Editor& Editor Group

VS Code刚开始不支持Tab，只支持Editor Group（我理解为分屏），通过`Cmd + \`横向拆分Editor Group，然后使用`Cmd + option + 0`可以将水平的Editor Group切换为纵向，并且使用`Cmd + 编号`在Group中切换。如果两个文件打开了，可以使用Command Palette中的`View: Move Editor into Right Group`将文件移到一个Group中。

后来支持了Tab(Editor)，使用`Cmd + option + 方向键`左右切换Tab，也可以使用`tab + control`在Tab间来回切换，个人不是很习惯。

### setting & keyboard shortcuts

可以通过点击齿轮按钮或`Cmd + ,`，也可以通过Command Palette - `Preferences: Open User Settings`打开图形化界面。

也可以通过Command Palette - `Preferences: Open Settings(JSON)`打开`settings.json`进行编辑。当然，如果`.vscode/settings.json`文件存在，即存在工作区配置文件，也可以只修改工作区的配置文件。

可以通过点击齿轮的keyboard shortcuts进入图形化快捷键设置。

也可以通过Command Palette - `Preferences: Open Keyboard Shortcuts(JSON)`进入`keybindings.json`进行编辑。

## VS Code的亮点功能

### CodeSnippet

用法见上部分介绍，类似于Idea的Live Template。

### Command Palette

用法见上部分介绍，对执行功能进行了细分。

### 文件管理

文件管理作为一个编辑器的核心功能，一定需要着重了解。

### workflow(task)

这个功能比较复杂，感觉一般人用不到，可以跳过，如果有需要，可以再学。


## 推荐阅读

1. [第一次使用VS Code时你应该知道的一切配置 - 掘金](https://juejin.im/post/5cb87c6e6fb9a068a03af93a#heading-55)
