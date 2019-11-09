# Linux command: xargs

> The name `xargs`, pronounced EX-args, means “combine arguments.” `xargs` builds and executes command lines by gathering together arguments it reads on the standard input.
> 
> From: [https://www.gnu.org/software/findutils/manual/html_mono/find.html#Overview](https://www.gnu.org/software/findutils/manual/html_mono/find.html#Overview)


# 处理文件名中的空格或制表符

以下文件名包含空格：

```bash
$ find ~ -type f -iname '*kafka术语*' 2> /dev/null
/Users/liyang/github_projects/geektime2pdf/geektime_Kafka核心技术与实战/02 | 一篇文章带你快速搞定Kafka术语.pdf
/Users/liyang/Dropbox/ebooks/geektime/Kafka 核心技术与实战/02 | 一篇文章带你快速搞定Kafka术语.pdf
```

如果直接把输出管道（`|`）给其它命令处理会出错，例如：

```bash
$ find ~ -type f -iname '*kafka术语*' 2> /dev/null | md5sum
f25c95b7c99cefbc506a325178766d94  -
```

此时可以借助 `xargs` 命令来完成任务：

```bash
$ find ~ -type f -iname '*kafka术语*' 2> /dev/null | xargs -d '\n' md5sum
e4a05490854408e565fbc2d53ae4c3f3  /Users/liyang/github_projects/geektime2pdf/geektime_Kafka核心技术与实战/02 | 一篇文章带你快速搞定Kafka术语.pdf
1f45894c0aeae6987adcfbf511f218c9  /Users/liyang/Dropbox/ebooks/geektime/Kafka 核心技术与实战/02 | 一篇文章带你快速搞定Kafka术语.pdf
```

以上命令的关键是 `xargs -d '\n'`，就是让 `xargs` 仅仅把「换行符」当作分隔符（Delimiter），来分隔输入的文本流，而不是默认的「空白字符 + 换行符」。

> xargs reads items from the standard input, delimited by **blanks** (which can be protected with double or single quotes or a backslash) or **newlines**, ...
> 
> From: [The man-page of xargs](https://linux.die.net/man/1/xargs)


在 POSIX 兼容系统中，blanks 通常指的是 `[:blank:]` 字符组（Character Classes），包含「空格」和「制表符」两种字符。（_POSIX 还有一个 `[:space:]` 字符组，是 `[:blank:]` 字符组的**超集**_。）

> **blank**
> - Define characters to be classified as `blank` characters.
> - In the POSIX locale, only the `space` and `tab` shall be included.
> 
> **space**
> - Define characters to be classified as white-space characters.
> - In the POSIX locale, exactly `space`, `form-feed`, `newline`, `carriage-return`, `tab`, and `vertical-tab` shall be included.
> 
> From: [POSIX Locale Definition](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap07.html#tag_07_03_01)


# 处理文件名中的换行符

如果文件名包含换行符，前面所说的 `xargs -d '\n'` 方法就会失效，此时必须使用 `xargs -0`：

```bash
$ find ~ -type f -iname '*kafka术语*' -print0 2> /dev/null | xargs -0 md5sum
e4a05490854408e565fbc2d53ae4c3f3  /Users/liyang/github_projects/geektime2pdf/geektime_Kafka核心技术与实战/02 | 一篇文章带你快速搞定Kafka术语.pdf
1f45894c0aeae6987adcfbf511f218c9  /Users/liyang/Dropbox/ebooks/geektime/Kafka 核心技术与实战/02 | 一篇文章带你快速搞定Kafka术语.pdf
```

**注意**：

- `xargs -0` 的意思是，告诉 `xargs`，输入的文本流的分隔符是 ASCII `NUL`（`\000`）字符。因此，管道前面的命令必须提供 `NUL` 分隔的输出文本流，对于 `find` 命令，就必须使用 `-print0` 选项。换句话说，**`xargs -0` 必须搭配 `find -print0` 使用，单独使用其中一个是无效的**。
- 其它支持 `NUL` 分隔符的命令都有各自专用的选项：
  - `locate -0`
  - `grep -z`, `grep -Z`
  - `sort -z`
- 如果碰到不支持 `NUL` 作为分隔符的命令，例如 `head`, `tail`, `ls`, `echo`, `sed`, `tar -v`, `wc`, `which` 等等，`xargs -0` 就不会产生预期的效果。

> Many Unix utilities are line-oriented. These may work with `xargs` as long as the lines do not contain `'`, `"`, or a space. Some of the Unix utilities can use `NUL` as record separator (e.g. `perl` (requires `-0` and `\0` instead of `\n`), `locate` (requires using `-0`), `find` (requires using `-print0`), `grep` (requires `-z` or `-Z`), `sort` (requires using `-z`)). Using `-0` for `xargs` deals with the problem, but _many Unix utilities _**_cannot_**_ use `NUL` as separator (e.g. `head`, `tail`, `ls`, `echo`, `sed`, `tar -v`, `wc`, `which`)_.
> 
> From: [https://en.wikipedia.org/wiki/Xargs#Separator_problem](https://en.wikipedia.org/wiki/Xargs#Separator_problem)


# `find | xargs` vs `find -exec`

从脚本执行效率的角度来说，`find | xargs` 的效率明显超过 `find -exec`，处理的文件数量越多，`xargs` 的效率优势就越显著，网上已经有[详细的评测文章](https://www.everythingcli.org/find-exec-vs-find-xargs/)，我特意把它保存为 PDF，以防链接失效。


```bash
# https://www.everythingcli.org/find-exec-vs-find-xargs/

du -hs
 37G    .

# 'time' will print the total time taken for the command to finish

# time find -exec \;
time find . -name \*.php -type f -exec grep -Hn '$test' {} \;
real    1m24.433s
user    0m29.022s
sys     0m43.304s

# find -exec \+
time find . -name \*.php -type f -exec grep -Hn '$test' {} \+
real    0m13.050s
user    0m5.315s
sys     0m2.179s

# find | xargs -n1
time find . -name \*.php -type f -print0 | xargs -0 -n1 grep -Hn '$test'
real    0m55.159s
user    0m23.692s
sys     0m28.618s

# find | xargs
time find . -name \*.php -type f -print0 | xargs -0 -grep -Hn '$test'
real    0m12.047s
user    0m4.997s
sys     0m3.593s
```

> So obviously `find \+` and `xargs` (without `-n1`) are much faster because there is **no overhead in **`**fork**`** and **`**exec**`. It decreases I/O dramatically and seen from the times commands they are up to **6x faster**.


即便如此，`find | xargs` 和 `find -exec` 还是有各自的优缺点，以及适合各自的工作场景，不能简单的用前者完全取代后者。
