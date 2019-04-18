[TOC]

# 前言



Key promoter 快捷键提示 <https://plugins.jetbrains.com/plugin/4455?pr=idea>

CamelCase 驼峰式命名和下划线命名交替变化 <https://plugins.jetbrains.com/plugin/7160?pr=idea>

CheckStyle-IDEA 代码规范检查 <https://plugins.jetbrains.com/plugin/1065?pr=idea>

FindBugs-IDEA 潜在 Bug 检查 <https://plugins.jetbrains.com/plugin/3847?pr=idea>

MetricsReloaded 代码复杂度检查 <https://plugins.jetbrains.com/plugin/93?pr=idea>

Statistic 代码统计 <https://plugins.jetbrains.com/plugin/4509?pr=idea>

JRebel Plugin 热部署 <https://plugins.jetbrains.com/plugin/?id=4441>

CodeGlance 在编辑代码最右侧，显示一块代码小地图 <https://plugins.jetbrains.com/plugin/7275?pr=idea>

GsonFormat 把 JSON 字符串直接实例化成类 <https://plugins.jetbrains.com/plugin/7654?pr=idea>

Eclipse Code Formatter 使用 Eclipse 的代码格式化风格，在一个团队中如果公司有规定格式化风格，这个可以使用。 <https://plugins.jetbrains.com/plugin/6546?pr=idea>

Ace Jump AceJump其实是一款能够代替鼠标的软件，只要安装了这款插件，可以在代码中跳转到任意位置。按快捷键进入 AceJump 模式后（默认是 Ctrl+J），再按任一个字符，插件就会在屏幕中这个字符的所有出现位置都打上标签，你只要再按一下标签的字符，就能把光标移到该位置上。换言之，你要 移动光标时，眼睛一直看着目标位置就行了，根本不用管光标的当前位置。 









# 一、IdeaVim

## 1.安装

- 安装

    > 依次选择 `File` -> `Settings` -> `Plugins` -> `Install JetBrains plugin`，搜索 `IdeaVim` 进行安装



- 启用与禁用

    > - 启用：勾选 `tools` ->  `Vim Emulator`
    > - 禁用：不勾选 `tools` ->  `Vim Emulator`

    

## 2. 配置

vim的一些配置也可以在ideavim中使用，关键在于配置文件.ideavimrc（windows下为_ideavimrc）。该文件默认是不存在的，需要手动创建。



1. 在~目录下，创建.ideavimrc（windows下为_ideavimrc）。
2. 添加配置内容：



```shell
" Vim 的默认寄存器和系统剪贴板共享
set clipboard+=unnamed
set history=100000
" select模式下复制
if has("clipboard")
    vnoremap <C-C> "+y
endif
" 映射到idea快捷键s
" 弹出输入框，可以跳到指定类
nnoremap <Space>gc :action GotoClass<CR>
" 弹出输入框，跳转到指定操作
nnoremap <Space>ga :action GotoAction<CR>
" 跳转到定义
nnoremap <Space>gd :action GotoDeclaration<CR>
" 跳转到实现
nnoremap <Space>gi :action GotoImplementation<CR>
" 跳转到指定的文件
nnoremap <Space>gf :action GotoFile<CR>
" 跳转到方法的声明
nnoremap <Space>gs :action GotoSuperMethod<CR>
" 跳转到测试
nnoremap <Space>gt :action GotoTest<CR>
" 跳转到变量的声明
nnoremap <Space>gS :action GotoSymbol<CR>

" 查找使用
nnoremap <Space>fu :action FindUsages<CR>
" 显示使用
nnoremap <Space>su :action ShowUsages<CR>

" 前进，相当似于eclipse中的alt+方向右键
nnoremap gf :action Forward<CR>
" 后退，相当于eclipse中的alt+方向左键
nnoremap gb :action Back<CR>

" gh=go head, 映射vim中的^
nnoremap gh ^
" gl=go last，映射vim中的$
nnoremap gl $

" Window operation
nnoremap <Space>ww <C-W>w
nnoremap <Space>wc <C-W>c
nnoremap <Space>wj <C-W>j
nnoremap <Space>wk <C-W>k
nnoremap <Space>wh <C-W>h
nnoremap <Space>wl <C-W>l
nnoremap <Space>ws <C-W>s
nnoremap <Space>w- <C-W>-
nnoremap <Space>w+ <C-W>+
nnoremap <Space>w= <C-W>=

nnoremap <Space>wv <C-W>vf
```

























# 参考资料

1. https://github.com/JetBrains/ideavim
2. [如何成为 IntelliJ IDEA 键盘流？](https://www.zhihu.com/question/20783392)
3.  [【idea系列】ideaVim安装及配置 原](https://my.oschina.net/funcy/blog/1832719)







