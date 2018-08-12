---
title: SpaceVim使用
date: 2018-08-08 23:39:28
tags:vim,SpaceVim
---

# SpaceVim简介

> SpaceVim 是一个社区驱动的模块化 Vim IDE，以模块(layers)的方式组织管理插件以及相关配置， 为不同的语言开发量身定制了相关的开发模块，该模块提供代码自动补全， 语法检查、格式化、调试、REPL 等特性。用户仅需载入相关语言的模块即可得到一个开箱即用的Vim-IDE。
>
> 灵感来自于 Emacs 界首选配置 Spacemacs。

# 入门指南

## 安装

在安装 SpaceVim 之前，确保已安装 `git` 和 `curl，`这两个工具用来 下载插件以及字体。

```shell
curl -sLf https://spacevim.org/cn/install.sh | bash
```

安装结束后，初次打开 `vim` 或者 `neovim` 时， SpaceVim 会**自动**下载并安装插件。

## 配置

SpaceVim 的默认配置文件为 `~/.SpaceVim.d/init.toml`，下面为一个简单的配置示例。 如果需要查阅更多 SpaceVim 配置相关的信息，请阅读 SpaceVim 用户文档。

```toml
# 这是一个基础的 SpaceVim 配置示例

# 所有的 SpaceVim 选项都列在 [option] 之下
[options]
    # 设置 SpaceVim 主题及背景，默认的主题是 gruvbox，如果你需要使用更
    # 多的主题，你可以载入 colorscheme 模块
    colorscheme = "gruvbox"
    # 背景可以取值 "dark" 和 "light"
    colorscheme_bg = "dark"
    # 启用/禁用终端真色，在目前大多数终端下都是支持真色的，当然也有
    # 一小部分终端不支持真色，如果你的 SpaceVim 颜色看上去比较怪异
    # 可以禁用终端真色，将下面的值设为 false
    enable_guicolors = true
    # 设置状态栏上分割符号形状，如果字体安装失败，可以将值设为 "nil" 以
    # 禁用分割符号，默认为箭头 "arrow"
    statusline_separator = "nil"
    statusline_separator = "bar"
    # 设置顶部标签列表序号类型，有以下五种类型，分别是 0 - 4
    # 0: 1 ➛ ➊ 
    # 1: 1 ➛ ➀
    # 2: 1 ➛ ⓵
    # 3: 1 ➛ ¹
    # 4: 1 ➛ 1
    buffer_index_type = 4
    # 显示/隐藏顶部标签栏上文件类型图标，这一图标需要安装 nerd fonts，
    # 如果未能成功安装这一字体，可以隐藏图标
    enable_tabline_filetype_icon = true
    # 是否在状态栏上显示当前模式，默认情况下，不显示 Normal/Insert 等
    # 字样，只以颜色区分当前模式
    enable_statusline_display_mode = false

# SpaceVim 模块设置，主要包括启用/禁用模块

# 启用 autocomplete 模块, 启用模块时，可以列出一些模块选项，并赋值，
# 关于模块的选项，请阅读各个模块的文档
[[layers]]
name = "autocomplete"
auto-completion-return-key-behavior = "complete"
auto-completion-tab-key-behavior = "cycle"

# 禁用 shell 模块, 禁用模块时，需要加入 enable = false
[[layers]]
name = "shell"
enable = false

# 添加自定义插件
[[custom_plugins]]
name = "lilydjwg/colorizer"
merged = 0
```

# 模块（layers）/快捷键

## 模块使用

- 使能模块

  ```shell
  [[layers]]
  name = "shell"
  default_position = "top"
  default_height = 30
  ```

- 禁止模块

  ```shell
  [[layers]]
  name = "shell"
  enable = false
  ```

以下介绍软件开发中经常使用的的模块及快捷键。

## tools

在`.SpaceVim/autoload/SpaceVim/layers/tools.vim`文件中可以看到 tools 下有哪些插件。从作者关于 tools 这个模块的说明中可以得知，这些插件的快捷键都和插件的默认定义是一致的，并没有做任何更改。

下面选择几个非常有用的插件进行说明。

- MattesGroeger/vim-bookmarks -- 标签相关

  > 在写代码是，需要经常转换行，这时候我们需要对要记住位置的行加入标签，然后进行跳转。

  ![](https://camo.githubusercontent.com/bc2bf1746e30c72d7ff5b79331231e8c388d068a/68747470733a2f2f7261772e6769746875622e636f6d2f4d617474657347726f656765722f76696d2d626f6f6b6d61726b732f6d61737465722f707265766965772e676966)

  | Action                                        | Shortcut     | Command                       |
  | --------------------------------------------- | ------------ | ----------------------------- |
  | Add/remove bookmark at current line           | `mm`         | `:BookmarkToggle`             |
  | Add/edit/remove annotation at current line    | `mi`         | `:BookmarkAnnotate <TEXT>`    |
  | Jump to next bookmark in buffer               | `mn`         | `:BookmarkNext`               |
  | Jump to previous bookmark in buffer           | `mp`         | `:BookmarkPrev`               |
  | Show all bookmarks (toggle)                   | `ma`         | `:BookmarkShowAll`            |
  | Clear bookmarks in current buffer only        | `mc`         | `:BookmarkClear`              |
  | Clear bookmarks in all buffers                | `mx`         | `:BookmarkClearAll`           |
  | Move up bookmark at current line              | `[count]mkk` | `:BookmarkMoveUp [<COUNT>]`   |
  | Move down bookmark at current line            | `[count]mjj` | `:BookmarkMoveDown [<COUNT>]` |
  | Move bookmark at current line to another line | `[count]mg`  | `:BookmarkMoveToLine <LINE>`  |
  | Save all bookmarks to a file                  |              | `:BookmarkSave <FILE_PATH>`   |
  | Load bookmarks from a file                    |              | `:BookmarkLoad <FILE_PATH>`   |

## cscope

cscope 作用摘抄如下：

>- all references to a symbol
>- global definitions
>- functions called by a function
>- functions calling a function
>- text string
>- regular expression pattern
>- a file
>- files including a file

配置：

1. 安装cscope(建议源码安装)

2. 源码根目录执行 cscope -Rbq(可以有更复杂的选项)

3. SpaceVim 配置文件中添加 cscope layer

   ```shell
   [[layers]]
   name = 'cscope'
   ```

使用：

| Key Binding | Description                            |
| ----------- | -------------------------------------- |
| `SPC m c =` | Find assignments to this symbol        |
| `SPC m c i` | Create cscope index                    |
| `SPC m c c` | Find functions called by this function |
| `SPC m c C` | Find functions calling this function   |
| `SPC m c d` | find global definition of a symbol     |
| `SPC m c r` | find references of a symbol            |
| `SPC m c f` | find file                              |
| `SPC m c F` | find which files include a file        |
| `SPC m c e` | search regular expression              |
| `SPC m c t` | search text                            |

## tags

GNU GLOBAL 作用官网摘抄如下：

> GNU GLOBAL is a source code tagging system that works the same way across diverse environments, such as Emacs editor, Vi editor, Less viewer, Bash shell, various web browsers, etc.
>
> You can locate various objects, such as functions, macros, structs, classes, in your source files and move there easily. It is useful for hacking a large projects which contain many sub-directories, many `#ifdef` and many `main()`functions. It is similar to ctags or etags, but is different from them in the following two points:
>
> - independence of any editor
> - capability to treat definition and reference

配置：

1. 安装 GNU GLOBAL (建议源码安装)
2. 源码根目录执行 gtags (更复杂的选项请自行研究)
3. SpaceVim 配置文件中添加 gtags layer

使用：

| Key Binding | Description                    |
| ----------- | ------------------------------ |
| `SPC m g c` | create a tag database          |
| `SPC m g u` | manually update tag database   |
| `SPC m g f` | jump to a file in tag database |
| `SPC m g d` | find definitions               |
| `SPC m g r` | find references                |

# FAQ

> 下文摘自官方文档

## 为什么选择 Toml 作为默认配置语言？

在往期的版本中，一直使用的 Vim 脚本作为配置文件，而 SpaceVim 读取配置文件的机制是 直接载入该脚本。Vim 在载入脚本时是边载入边执行的，这就意味着当你的配置文件中间部分 出现语法错误时，并不能阻止前半部分配置被载入，排错时非常有影响。

因此我们选择了另外一种更加健壮的语言来配置 SpaceVim，SpaceVim 会完整读取该配置文件， 如果文件中间出现语法错误，导致解析失败。那么该配置会被完全舍弃，而使用 SpaceVim 的 默认配置，这就大大降低了因配置文件错误导致 SpaceVim 运行出错的可能性。

在配置文件格式选择时，我们在 json、yaml、xml、toml 这四中文件格式之间也做了比较。

1. yaml 依赖缩进，配置转移时易出错，不与考虑
2. xml 缺少 vim 解析库， 不与考虑
3. json 时一个比较好的配置信息传输格式，并且 Vim 有一个解析的函数，但是 json 格式 不支持注释，手写编辑时，阅读性太差。

因此，我们选择了 Toml 作为默认的配置格式，并且解析后，缓存为 json 文件。SpaceVim 在启动时直接读取缓存 json 文件，效率更高。

## 为什么 SpaceVim 颜色主题和官网不一致？

因为在 SpaceVim 中，默认情况下是启用了终端真色，因此你需要确保你的终端支持真色。 但是并不是每种终端默认都支持真色的。因此，当你的终端不支持真色时， 你可以在配置文件里面禁用真色支持：

```
  enable_guicolors = false
```

## 如何增加自定义快捷键？

使用 Toml 作为默认配置文件后，无法在配置文件里面直接添加 Vim 快捷键， 这点让很多用户感到困惑。实际上，SpaceVim 支持指定载入配置时需要调用的函数。

比如，我需要加入这样一个快捷键，使用 `<Leader> w` 来保存当前文件。那么， 我需要修改配置文件，并指定一个载入时需要调用的方法：

修改 `~/.SpaceVim.d/init.toml`，加入 `bootstrap_before` 选项：

```
[options]
  bootstrap_before = "myspacevim#init"
```

添加文件 `~/.SpaceVim.d/autoload/myspacevim.vim`, 并加入如下内容：

```
function! myspacevim#init() abort
  nnoremap <Leader>w :w<cr>
endfunction
```