# Vim plugin: vim-easy-align

在 vim-easy-align 的项目主页上是这样介绍自己的：简单易用的 Vim 对齐插件。

> A simple, easy-to-use Vim alignment plugin.
> 
> From: https://github.com/junegunn/vim-easy-align

实际上，这个插件提供强大的**高级**对齐功能，但刚开始接触这些高级功能的时候，可能没那么「简单」。还好，这个项目提供了详细的文档，参照文档可以快速学习。

# 我的必备配置

默认的对齐规则只能在  `<Space>`, `=`, `:`, `,`, `|`, `.`, `#`, `&`, `{`, `}` 这十个**分隔符（delimiter）**所在的位置对齐，而我经常需要根据其它符号对齐，例如**正斜线 `/`** 和 **反斜线`\`**，那就需要在 `.vimrc` 文件中完成以下配置，并重启 Vim 使其生效。

```
let g:easy_align_delimiters = {
\ '/': {
\     'pattern':         '//\+\|/\*\|\*/',
\     'delimiter_align': 'l',
\     'ignore_groups':   ['!Comment'] },
\ '\': { 'pattern': '\\$' }
\ }
```

# 示例：根据正斜线对齐

现有以下 JavaScript 代码：

```js
// https://github.com/jjeejj/geektime2pdf/blob/master/config.js

module.exports = {
    url: 'https://time.geekbang.org/serv/v1/article',
    commentUrl: 'https://time.geekbang.org/serv/v1/comments',
    columnBaseUrl: 'https://time.geekbang.org/column/article/',
    columnName: '软件工程之美',
    firstArticalId: 123456789, //专栏第一篇文章的ID
    isdownloadVideo: false, // 是否下载音频
    isComment: true, // 是否导出评论
    cookie: 'cookie'
};
```

我需要其中带有注释的三行在第一个正斜线位置对齐，可以这样做：

1. `:set filetype=javascript` ，
2. 按下 `V` 键，进入 Visual Mode，选中带有注释的三行，
3. `:EasyAlign /` 。

代码就在斜线位置对齐了：

```js
firstArticalId: 123456789, // 专栏第一篇文章的ID
isdownloadVideo: false,    // 是否下载音频
isComment: true,           // 是否导出评论
```

注意，第一步 `:set filetype=javascript` 是否执行要看情况：

- 如果以上代码已经保存为 `.js` 文件，然后在 Vim 里编辑，就不需要执行。因为打开文件的时候就已经自动检测到了包含的是 JavaScript 代码。
- 如果直接把代码粘贴到 Vim 空白缓冲区，此时 Vim 不知道代码属于哪种语言，就必须手工执行上面这个命令来告知 Vim。

如果代码一开始是下面这样：

```js
// firstArticalId: 123456789, //专栏第一篇文章的ID
// isdownloadVideo: false, // 是否下载音频
// isComment: true, // 是否导出评论
```

此时如果想对齐尾部注释，就需要注意：每一行有两组双斜线 `//` ，经过本文开头的配置，每一组双斜线被视为**一个整体**，而**不是两个单独的斜线**，通过具体的命令很容易发现其中的区别。

- **正确**的对齐命令：`:EasyAlign 2/` 。
- 错误的对齐命令：`:EasyAlign 3/` 。

正确对齐之后的代码是这样：

```js
// firstArticalId: 123456789, // 专栏第一篇文章的ID
// isdownloadVideo: false,    // 是否下载音频
// isComment: true,           // 是否导出评论
```

# 示例：对齐注释或字符串包含的分隔符

默认情况下，vim-easy-align 会忽略**注释**或**字符串**包含的分隔符：

```
" Default:
"   If a delimiter is in a highlight group whose name matches
"   any of the followings, it will be ignored.
let g:easy_align_ignore_groups = ['Comment', 'String']
```

但在某些情况下，我可能依然想要对齐这些分隔符，此时就需要在对齐命令**末尾**添加特定的**选项（option）**。当然，这是 `:EasyAlign` 命令的语法决定的：

```
" Type 1
:EasyAlign[!] [N-th] DELIMITER_KEY [OPTIONS]

" Type 2
:EasyAlign[!] [N-th] /REGEXP/ [OPTIONS]
```

用下面的代码举例说明：

```js
// firstArticalId: 123456789,
// isdownloadVideo: false,
// isComment: true,
```

在上面三行被注释的代码中，我希望**冒号**后面的值能够对齐，此时：

- **正确**的命令：`:EasyAlign : ig[]` 。
- 错误的命令：`:EasyAlign :` 。

这里的 `ig[]` 就是一种**选项（option）**。对齐结果如下：

```js
// firstArticalId:  123456789,
// isdownloadVideo: false,
// isComment:       true,
```

除了直接输入 `ig[]` ，还可以在**交互模式（interactive mode）**用快捷键切换多种忽略方式：

1. `:EasyAlign` ，
2. 按下回车键 `ENTER` 进入交互模式，
3. 按下 `CTRL-G` 循环切换忽略的**语法组（highlight group）**，
4. 输入分隔符。

所有这些针对语法组的忽略都是基于 Vim 的语法高亮功能，所以语法高亮功能必须启用：`:syntax on` 或者 `:syntax enable` 。

---

其它高级用法等我用到了再补充。