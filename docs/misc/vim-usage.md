
upd on 2022.12.31: 

> bram 你是杀软吗，windows 下 gvim 这适配做的和什么一样，难绷。
> 
> 只能说守着那套老东西死活不肯改变的确实就这样。
> 
> 相比之下 Sublime 就好得多，都是个人开发，sublime 明显比你注重用户体验吧，笑晕了。

----

Practise more.

考场 Vimrc:

```vim
color blue
set backspace=indent,eol,start
set clipborad^=unnamed,unnamedplus
set backup nu mouse=a ts=4 sw=4 sts=4 
set ar ai acd guifont=Consolas:h12:cANSI

inoremap [ []<Esc>i
inoremap {<CR> {}<Esc>i<CR><ESC>O
inoremap ( ()<Esc>i
inoremap " ""<Esc>i
inoremap ' ''<Esc>i
```

## 编辑模式

### Normal Mode

一般 Vim 的命令都是在 Normal mode 下面完成的。

所有模式按 `ESC` 都可以进入 Normal mode。

Normal mode 不可以进行输入，键盘上所有的字母基本都是个命令。

也可以用 `:` 进入 Command mode。

下面的命令会标记 `[N]` 表示该快捷键在 Normal mode 下可用。

### Command Mode

可以进行一些文件操作，属性的设置。

还可以利用 `:!<Command>` 在一个外部终端里面执行 `<Command>` 命令。

比如 `!g++ % -o %< -Wall -std=c++17 -O2 -g` 就可以编译当前的文件（cpp）。

### Insert Mode

正常的输入模式，Vim Easy 只有这个模式。

### Visual Mode

选择模式，有三种。

第一种是在 Normal mode 下按 `v` 进入的可视模式，这个就是类似其他 Editor 的选择功能。

第二种是按 `V` 也就是 `S-v` 进入的行可视模式，这个模式会自动选择光标所在行，并且会复制换行符。

第三种是按 `C-v` 进入的块可视模式，这个模式会选中一个块（矩形），比如下面这个实例：

```cpp
	std::cin.tie(0) -> sync_with_stdio(false);
	std::cin.exceptions(cin.failbit | cin.badbit);
```

我们想把它们前面的 `std::` 全部删掉，就可以用块可视模式选中然后删除（当然你也可以一个个查找然后repalce掉它们）。

### Replace Mode

按 `R` 进入，顾名思义是用来批量 replace 的。

## 编辑器快捷键

### 移动类

+ `[N/V] h j k l ` ： 上下左右（可以在前面加上数字表示重复多少次，比如 $3l$ 就是往左移动三个字符（Vim 的所有命令基本都支持这个））
+ `[N/V] gg` ：跳到文件的开头。
+ `[N/V] G`：跳到文件的结尾
+ `[N/V] <Number>G  / :<Number>` ： 跳到第 `<Number>` 行
+ `[N/V] <Number>|`：把光标移动到第 `<Number>` 列上
+ `[N/V] Backspace Space`： 移动到上一个字符/下一个字符
+ `[N/V] + Enter`：移动到下一行第一个非空白字符
+ `[N/V] -`：移动到上一行第一个非空白字符。
+ `[N] f<Char>`：移动到同一行的下一个 `<Char>` 字符处。
+ `[N] F<Char>`：移动到同一行的上一个 `<Char>` 字符处。
+ `[N/V] ^/$` ：移动到行头/尾。
+ `[N/V] H`: 把光标移到屏幕最顶端一行。
+ `[N/V] M`: 把光标移到屏幕中间一行。
+ `[N/V] L`: 把光标移到屏幕最底端一行。

### 纯编辑类

+ `[N] i`：进入 Insert Mode（在当前光标块的左侧插入）
+ `[N] a`：进入 Insert Mode（在当前光标块的右侧插入）
+ `[N] O`：在当前行上方新建一行，并进入插入模式，光标处于行首（除非有 auto indent）
+ `[N] o`：在当前行下方新建一行，并进入插入模式，光标处于行首（除非有 auto indent）
+ `[N] I`：将光标移动至本行末尾，同时进入插入模式。
+ `[N] A`：将光标移动至本行首，同时进入插入模式。
+ `[N] u/C-r`：Undo 和 Redo。
+ `[N] ~`：反转光标块字符的大小写。

### 复制粘贴类

+ `[V] y`：复制在可视模式下选中的文本（可以加组合键来复制一个单词/句子之类的，不过我不咋用就不写了）。
+ `[N/V] yy` 复制当前行。
+ `[N] ggVG` ：这是一个组合键，等价于全文复制（会多复制一个换行符）。
+ `[N/V] d`：剪切（需要类似 `d<Number><Move>` 的组合键，比如 `d8l` 就是删除光标块右边8个字符（包括光标块本身（光标实际上是在光标块的左边）），`d8h` 则是左边（不含光标块），`dG` 是从当前行一直删到文件结尾）。
+ `[N/V] dd`：剪切一整行。
+ `[N/V] x<Number>`：删除光标右边 `<Number>` 个字符（光标是在光标块的左边，所以如果你直接按 `x` 会删除光标块选中的字符）。
+ `[N/V] X<Number>`：左边。
+ `[C] :<i>,<j> dd`：剪切第 $i$ 行到第 $j$ 行的所有文本。
+ `[N] p/P` ：在光标的后/前位置粘贴（注意是否复制了换行符，可以使用 `reg` 查看对应寄存器里的东西）
+ `[N/V] "<char>y`：复制到 `<char>` 寄存器当中（`0 ~ 9` 的数字，`a ~ z/A ~ Z` 的字母，`+-*`）。

`+` 是系统剪贴板，`*` 是当前选择缓冲区。

### 文件操作类

+ `:file filename`：命名当前文件为 `filename`。
+ `:e file`：关闭当前的文件，打开文件 `file`（可以是目录，可以指定文件位置）
+ `:e#`：回到上一个文件
+ `:e`：重新加载当前文件。
+ `:w`：写入当前文件。
+ `:q`：退出。
+ `:wq`：保存并退出。
+ `:saveas filename`：另存为。 

### 屏幕操作类

+ `[N]C-f`: 下翻一屏。
+ `[N]C-b`: 上翻一屏。
+ `[N]C-d`: 下翻半屏。
+ `[N]C-u`: 上翻半屏。
+ `[N]C-e`: 向下滚动一行。
+ `[N]C-y`: 向上滚动一行。
+ `[N]zz`: 将当前行移动到屏幕中央。
+ `[N]zt`: 将当前行移动到屏幕顶端。
+ `[N]zb`: 将当前行移动到屏幕底端。
+ `[N]C-w H/J/K/L` ：将当前窗口移动到屏幕的左/下/上/右处。
+ `[N]C-w h/j/k/l` ：切换到左/下/上/右的窗口。
+ `:split`：分割当前的窗口为两个窗口（水平分割），可以用 `:set scb` 打开同步滚动。
+ `:split <filename>`：水平分割并打开新的文件。
+ `:new`：水平打开一个新窗口并编辑一个新的文件。
+ `:vsplit/:vnew`：竖直分割。
+ `:only`：只保留当前窗口。
+ `:close`：关闭当前窗口。 
### 多文件编辑类

`vim a.txt b.cpp c.json` 可以同时打开多个文件。

+ `:n` 可以编辑下一个文件，`:N` 可以编辑上一个文件。
+ `:ls` 或者 `:args` 可以显示文件列表。 

### 多标签编辑类

注意这个和多文件不一样，一个标签页里可以有多个窗口，多文件不是标签页，只是同时打开了多个文件。

也就是说标签页类似 Workspace。

+ `vim -p files`：打开多个文件，每个文件占用一个标签页。
+ `:tabe/:tabnew`：如果加文件名，就在新的标签中打开这个文件， 否则打开一个空缓冲区。
+ `C-w gf`：在新的标签页里打开光标下路径指定的文件。
+ `:tabn`：切换到下一个标签，`C-PageDown` 也可以。
+ `:tabp`： 切换到上一个标签，`C-PageUp` 也可以。
+ `<n> gt`： 切换到下一个标签。如果前面加了 n，就切换到第 n 个标签。第一个标签的序号是 1。
+ `gT`：切换到上一个。
+ `:tab split`： 将当前缓冲区的内容在新页签中打开。
+ `:tabc[lose]`： 关闭当前的标签页。
+ `:tabo[nly]`： 关闭其它的标签页。
+ `:tabs`： 列出所有的标签页和它们包含的窗口。
+ `:tabm[ove] <N>`： 将当前标签页移动到第 $N$ 个标签页之后。

### 查找/替换类

+ `:s/old/new`：用 `new` 替换当前行第一个 `old`。
+ `:s/old/new/g`：用 `new` 替换当前行所有的 `old`。
+ `:n1,n2s/old/new/g`：用 `new` 替换当前文件 `n1` 行到 `n2` 行所有的 `old`。
+ `:%s/old/new/g`：用 `new` 替换文件中所有的 `old`。
+ `:%s/^/xxx/g`：在每一行的行首插入 `xxx`，`^` 表示行首。
+ `:%s/$/xxx/g`：在每一行的行尾插入 `xxx`，`$` 表示行尾。

Tips: 如果所有替换命令末尾加上了 `c`，则每个替换都将需要用户确认。 如 `:%s/old/new/gc`，加上 `i` 则忽略大小写 `(ignore)`。

### 插件扩展快捷键类

+ `[N] \ci \cu`： 设置注释或者取消注释（Need Nerd-Commenter）
+ `[N] zR`：展开所有 Folding（Markdown）
+ `[N/V] C-N`：选中词汇并进入 Visual-Multi，或者选中下一个匹配（Need Visual-Multi）
	+ `[M] n/N`：获取下一个/上一个事件。
	+ `[M] q`：跳过当前事件。
	+ `C-Down/Up` ：创建垂直光标。
+ `:MarkdownPreview`：预览 Markdown。
+ `:MarkdownPreviewStop`：停止预览。

### 其他类。

+ `:pwd`：显示当前工作目录。
+ 在任意命令后加上 `!` 表示强制执行，例如 `:q!`，`:close!`。
+ 在任意命令前加上 `!` 表示在终端中执行命令。
+ `:terminal` 可以在当前目录打开一个新的终端（Windows下默认 cmd）。
+ `:TOhtml` 可以把当前文档转化为 HTML。 
+ 大部分命令前加上数字表示重复多少次。
+ `cd` 可以更改工作目录。
+ 在 Command 模式下使用 Tab 会有补全（区分大小写）
+ Vim 的大部分命令都是单词的缩写，比如 yank, paste, quit, write.
