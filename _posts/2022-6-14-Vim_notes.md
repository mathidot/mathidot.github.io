---
layout : post
titile : Vim notes
tags : Missing Semester
---

#### 5 modes

* normal (`Esc` or `ctrl+[`): moving around a file
* insert (`i`): inserting text
* replace (`R`): replacing text
* visual (plain `v`, line `V`, block `ctrl+v`, last block `gv`): selecting blocks of text()
* command-line (`:`): running command

#### command-line mode

* `:q` quit window
* `:w` write
* `:wq` write and quit
* `:e {name of file}` open file for editing
* `:ls` show open buffers
* `:help {topic}` open help, such as `:help :w`, `:help w`
* `:sp` seperate another window

#### movement in normal mode

* Basic movements：`hjkl` left down up right

* Words: `w` next words, `b` begin of word, `e` end of word

* Lines: `0` beginning of line, `$` end of line, `^` first non-blank character

* Screens: `H` top of screen, `M` middle of screen, `L` bottom of screen

* Scroll: `ctrl + d` page down, `ctrl + u` page up

* File: `gg` beginning of file, `G` end of file

* Find: `f{c}` forward find c, `t{c}` forward to c, `F{c}` backward find c, `T{c}` backward to c

  > difference between `f` and `t`:
  >
  > * `f` places the cursor on the found character
  > * `t` palces the cursor on the preceding character it found
  >
  > ref: https://stackoverflow.com/questions/12495442/what-do-the-f-and-t-commands-do-in-vim

* Search: `/{regex}`, `n`/`N` for navigating matches. 搜索结束之后输入 `enter` 转入到第一个结果位置，之后输入 `n` 跳转到下一个结果位置, `shift + n` jump to previous match.
* Misc: `%` find the coresponding item (找到相应的项，例如两个括号之间进行跳转)

#### selection in normal mode

* Visual: `v`
* Visual line: `V`
* Visual block: `ctrl + v`
* Last visual block: `gv`

#### edit commands

* `i` insert at now cursor
* `a` append at end
* `o / O` insert line down (below), up (above)
* `d{motion}` delete {motion}. such as `dd` delete this line, `d$` delete to end of line, `d0` delete to begin of line, `dw` delete word, `dG` delete to end of file, `dl` delete right character
* `c{motion}` change {motion}. `cw` change word etc, `cl` equals to `s`, `cc` equals to `S`
* `x` delete character (equals to `dl`)
* `s` delete character and start insert (substitute) (equals to `xi`, and short for `cl`)
* `u` undo, `ctrl + r` redo
* `y` copy. such as `yy` copy this line, `yw` copy the word, `y4l` copy right 4 characters
* `p` paste
* `~` flips the case of a character (大小写转换)
* `.` repeat previous editing command

#### count with manipulation

* `3w` move 3 words forward
* `5j` move 5 lines down
* `7dw / d7w` delete 7 words
* `7yw / y7w` copy 7 words

#### vim extensions

since Vim8.0 不再需要额外的插件管理器，使用内置的即可。

将插件放在目录 `~/.vim/pack/vendor/start/` 下（可以使用 git clone 命令）。统一管理 dotfiles 之后安装插件方法见 `piaoliangkb/dotfiles` 仓库介绍的装方法 (install as git submodules).

[Awesome Vim](https://vimawesome.com/)

https://missing.csail.mit.edu/2020/editors/#extending-vim

**example install `ctrlp`:**

* Install and configure a plugin: `ctrlp.vim`.
* Create the plugins directory with `mkdir -p ~/.vim/pack/vendor/start`
* Download the plugin: `cd ~/.vim/pack/vendor/start`; `git clone https://github.com/ctrlpvim/ctrlp.vim`
* Read the documentation for the plugin. Try using `CtrlP` to locate a file by navigating to a project directory, opening Vim, and using the Vim command-line to start `:CtrlP`.
* Customize `CtrlP` by adding configuration to your ~/.vimrc to open `CtrlP` by pressing `Ctrl-P`.

**example install `lightline`:**

* Install plugin `lightline.vim`
* `git clone https://github.com/itchyny/lightline.vim ~/.vim/pack/vendor/start/lightline`

## 搜索和替换

`:s` （替换）命令（[文档](http://vim.wikia.com/wiki/Search_and_replace)）。

* ```plaintext
  %s/foo/bar/g
  ```

  * 在整个文件中将 foo 全局替换成 bar

* ```plaintext
  %s/\[.*\](\(.*\))/\1/g
  ```

  * 将有命名的 Markdown 链接替换成简单 URLs

## 多窗口

* 用 `:sp` / `:vsp` 来分割窗口
* 同一个缓存可以在多个窗口中显示。

## 宏

* `q{字符}` 来开始在寄存器`{字符}`中录制宏

* `q`停止录制

* `@{字符}` 重放宏

* 宏的执行遇错误会停止

* `{计数}@{字符}`执行一个宏{计数}次

* 宏可以递归

  * 首先用`q{字符}q`清除宏
  * 录制该宏，用 `@{字符}` 来递归调用该宏 （在录制完成之前不会有任何操作）

* 例子：将 xml 转成 `json` ([file](https://missing-semester-cn.github.io/2020/files/example-data.xml))

  * 一个有 “name” / “email” 键对象的数组

  * 用一个 Python 程序？

  * 用 `sed` / 正则表达式

    * `g/people/d`
    * `%s/<person>/{/g`
    * `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
    * …

  * Vim 命令 / 宏

    * `Gdd`, `ggdd` 删除第一行和最后一行

    * 格式化最后一个元素的宏 （寄存器）

      ```plaintext
      e
      ```

      * 跳转到有 `<name>` 的行
      * `qe^r"f>s": "<ESC>f<C"<ESC>q`

    * 格式化一个

      的宏

      * 跳转到有 `<person>` 的行
      * `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`

    * 格式化一个

      标签然后转到另外一个的宏

      * 跳转到有 `<person>` 的行
      * `qq@pjq`

    * 执行宏到文件尾

      * `999@q`

    * 手动移除最后的 `,` 然后加上 `[` 和 `]` 分隔符

# 扩展资料

* `vimtutor` 是一个 Vim 安装时自带的教程
* [Vim Adventures](https://vim-adventures.com/) 是一个学习使用 Vim 的游戏
* [Vim Tips Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki)
* [Vim Advent Calendar](https://vimways.org/2019/) 有很多 Vim 小技巧
* [Vim Golf](http://www.vimgolf.com/) 是用 Vim 的用户界面作为程序语言的 [code golf](https://en.wikipedia.org/wiki/Code_golf)
* [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
* [Vim Screencasts](http://vimcasts.org/)
* [Practical Vim](https://pragprog.com/titles/dnvim2/)（书籍）

# 课后练习

[习题解答](https://missing-semester-cn.github.io/missing-notes-and-solutions/2020/solutions//editors-solution)

1. 完成 `vimtutor`。 备注：它在一个 [80x24](https://en.wikipedia.org/wiki/VT100)（80 列，24 行） 终端窗口看起来效果最好。

2. 下载我们提供的 [vimrc](https://missing-semester-cn.github.io/2020/files/vimrc)，然后把它保存到 `~/.vimrc`。 通读这个注释详细的文件 （用 Vim!）， 然后观察 Vim 在这个新的设置下看起来和使用起来有哪些细微的区别。

3. 安装和配置一个插件：

   `ctrlp.vim`

   1. 用 `mkdir -p ~/.vim/pack/vendor/start` 创建插件文件夹
   2. 下载这个插件： `cd ~/.vim/pack/vendor/start; git clone https://github.com/ctrlpvim/ctrlp.vim`
   3. 阅读这个插件的 [文档](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md)。 尝试用` CtrlP` 来在一个工程文件夹里定位一个文件， 打开 Vim, 然后用 Vim 命令控制行开始 `:CtrlP`.
   4. 自定义 `CtrlP`： 添加 [configuration](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options) 到你的 `~/.vimrc` 来用按 Ctrl-P 打开 `CtrlP`

4. 练习使用 Vim, 在你自己的机器上重做 [演示](https://missing-semester-cn.github.io/2020/editors/#demo)。

5. 下个月用 Vim 完成_所有的_文件编辑。每当不够高效的时候，或者你感觉 “一定有一个更好的方式”时， 尝试求助搜索引擎，很有可能有一个更好的方式。如果你遇到难题，可以来我们的答疑时间或者给我们发邮件。

6. 在其他工具中设置 Vim 快捷键 （见上面的操作指南）。

7. 进一步自定义你的 `~/.vimrc` 和安装更多插件。

8. （高阶）用 Vim 宏将 XML 转换到 JSON ([例子文件](https://missing-semester-cn.github.io/2020/files/example-data.xml))。 尝试着先完全自己做，但是在你卡住的时候可以查看上面[宏](https://missing-semester-cn.github.io/2020/editors/#macros) 章节。



## `Vimtutor`笔记

| KEY                              | ACTION                                   |
| -------------------------------- | ---------------------------------------- |
| x                                | 删除一次字符                             |
| dw                               | 删除一个单词                             |
| d$                               | 从当前位置至行尾，全部删除               |
| p                                | 将删除的文本粘贴到光标行之后             |
| rx                               | 将光标处的字符替换为 x                   |
| ce                               | 替换单词，直到单词结尾                   |
| CTRL-G                           | 显示光标所在行号和文件状态               |
| G                                | 跳转到最后一行                           |
| number G                         | 跳转到第 number 行                       |
| gg                               | 跳转到第一行                             |
| /                                | 向前搜索                                 |
| ?                                | 向后搜索                                 |
| n                                | 搜索下一个                               |
| N                                | 搜索上一个                               |
| Ctrl-O                           | 返回较旧位置                             |
| Ctrl-I                           | 跳转到较新位置                           |
| %                                | 寻找匹配的 ), ], 或 }                    |
| :s/old/new/g                     | 使用 ‘new’ 替换 ‘old’                    |
| :#,#s/old/new/g                  | #,# 是行号，替换两行之间的文本           |
| :%s/old/new/g                    | 全文范围内查找替换                       |
| :%s/old/new/gc                   | 全文范围内查找替换，每次替换前需用户确定 |
| !`<cmd>`                         | 执行外部命令                             |
| :w FILENAME                      | 保存文件                                 |
| v                                | 开启可视化选择模式                       |
| :r FILENAME                      | 在当前位置插入文件内容 retrieve          |
| :r !ls                           | 插入当前目录下的文件列表                 |
| o                                | 在当前行下方插入一行，并进入插入模式     |
| O                                | 在当前行上方插入一行，并进入插入模式     |
| R                                | 进入替换模式                             |
| y                                | 拷贝                                     |
| :set ic                          | 忽略大小写                               |
| :set noic                        | 大小写敏感                               |
| :set hls                         | 开启高亮搜索                             |
| :set ic                          | 开启递增搜索                             |
| :help                            | 获取帮助                                 |
| Ctrl-W Ctrl-W                    | 在窗口间跳转                             |
| :e ~/.vimrc                      | 编辑 vim 配置文件                        |
| :r $VIMRUNTIME/vimrc_example.vim | 插入 vim 配置文件示例                    |
| :help vimrc-intro                | 查看 vim 配置文件的文档介绍              |
| :set nocp                        | 进入非兼容模式                           |
| Ctrl-D and `<TAB>`               | 自动补全命令                             |

很多修改文本的命令，通常包含一个操作符和一个动作。常见的动作如下表所示：

| KEY  | MOTION                                   |
| ---- | ---------------------------------------- |
| w    | 直到下一个单词开始，不包含它的第一个字符 |
| e    | 直到当前单词结尾，包含最后一个字符       |
| $    | 直到行尾，包含最后一个字符               |

补充：

```shell
dd: delete a whole line
2dd: delete two lines
0: move to the start of the line using a zero
U(capital): to undo all changes on a line
u(lowercase): to undo previous actio
```























