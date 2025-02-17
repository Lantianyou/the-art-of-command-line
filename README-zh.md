# 命令行的艺术

[![Join the chat at https://gitter.im/jlevy/the-art-of-command-line](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/jlevy/the-art-of-command-line?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

- [前言](#前言)
- [基础](#基础)
- [日常使用](#日常使用)
- [文件及数据处理](#文件及数据处理)
- [系统调试](#系统调试)
- [单行脚本](#单行脚本)
- [冷门但有用](#冷门但有用)


![curl -s 'https://raw.githubusercontent.com/jlevy/the-art-of-command-line/master/README.md' | egrep -o '`\w+`' | tr -d '`' | cowsay -W50](cowsay.png)


## 基础

- 学会使用通配符 `*` （或许再算上 `?` 和 `[`...`]`） 和引用以及引用中 `'` 和 `"` 的区别（后文中有一些具体的例子）。

- 熟悉 Bash 中的任务管理工具：`&`，**ctrl-z**，**ctrl-c**，`jobs`，`fg`，`bg`，`kill` 等。

- 学会基本的文件管理工具：`ls` 和 `ls -l` （了解 `ls -l` 中每一列代表的意义），`less`，`head`，`tail` 和 `tail -f` （甚至 `less +F`），`ln` 和 `ln -s` （了解硬链接与软链接的区别），`chown`，`chmod`，`du` （硬盘使用情况概述：`du -hs *`）。 关于文件系统的管理，学习 `df`，`mount`，`fdisk`，`mkfs`，`lsblk`。知道 inode 是什么（与 `ls -i` 和 `df -i` 等命令相关）。

## 日常使用

- 在 Bash 中，可以按下 **ctrl-w** 删除你键入的最后一个单词，**ctrl-u** 可以删除行内光标所在位置之前的内容，**alt-b** 和 **alt-f** 可以以单词为单位移动光标，**ctrl-a** 可以将光标移至行首，**ctrl-e** 可以将光标移至行尾，**ctrl-k** 可以删除光标至行尾的所有内容，**ctrl-l** 可以清屏。键入 `man readline` 可以查看 Bash 中的默认快捷键。内容有很多，例如 **alt-.** 循环地移向前一个参数，而 **alt-*** 可以展开通配符。

- 为了便于编辑长命令，在设置你的默认编辑器后（例如 `export EDITOR=vim`），**ctrl-x** **ctrl-e** 会打开一个编辑器来编辑当前输入的命令。在 vi 风格下快捷键则是 **escape-v**。

- 键入 `history` 查看命令行历史记录，再用 `!n`（`n` 是命令编号）就可以再次执行。其中有许多缩写，最有用的大概就是 `!$`， 它用于指代上次键入的参数，而 `!!` 可以指代上次键入的命令了（参考 man 页面中的“HISTORY EXPANSION”）。不过这些功能，你也可以通过快捷键 **ctrl-r** 和 **alt-.** 来实现。

- 如果你输入命令的时候中途改了主意，按下 **alt-#** 在行首添加 `#` 把它当做注释再按下回车执行（或者依次按下 **ctrl-a**， **#**， **enter**）。这样做的话，之后借助命令行历史记录，你可以很方便恢复你刚才输入到一半的命令。

- 使用 `xargs` （ 或 `parallel`）。他们非常给力。注意到你可以控制每行参数个数（`-L`）和最大并行数（`-P`）。如果你不确定它们是否会按你想的那样工作，先使用 `xargs echo` 查看一下。此外，使用 `-I{}` 会很方便。例如：
```bash
      find . -name '*.py' | xargs grep some_function
      cat hosts | xargs -I{} ssh root@{} hostname
```

- `pstree -p` 以一种优雅的方式展示进程树。

- 使用 `pgrep` 和 `pkill` 根据名字查找进程或发送信号（`-f` 参数通常有用）。

- 了解你可以发往进程的信号的种类。比如，使用 `kill -STOP [pid]` 停止一个进程。使用 `man 7 signal` 查看详细列表。

- 使用 `nohup` 或 `disown` 使一个后台进程持续运行。

- 使用 `netstat -lntp` 或 `ss -plat` 检查哪些进程在监听端口（默认是检查 TCP 端口; 添加参数 `-u` 则检查 UDP 端口）或者 `lsof -iTCP -sTCP:LISTEN -P -n` (这也可以在 OS X 上运行)。

- `lsof` 来查看开启的套接字和文件。

- 把环境变量的设定以及登陆时要执行的命令保存在 `~/.bash_profile`。而对于从图形界面启动的 shell 和 `cron` 启动的 shell，则需要单独配置文件。

- 当变量和文件名中包含空格的时候要格外小心。Bash 变量要用引号括起来，比如 `"$FOO"`。尽量使用 `-0` 或 `-print0` 选项以便用 NULL 来分隔文件名，例如 `locate -0 pattern | xargs -0 ls -al` 或 `find / -print0 -type d | xargs -0 ls -al`。如果 for 循环中循环访问的文件名含有空字符（空格、tab 等字符），只需用 `IFS=$'\n'` 把内部字段分隔符设为换行符。

- 在 Bash 脚本中，使用 `set -x` 去调试输出（或者使用它的变体 `set -v`，它会记录原始输入，包括多余的参数和注释）。尽可能地使用严格模式：使用 `set -e` 令脚本在发生错误时退出而不是继续运行；使用 `set -u` 来检查是否使用了未赋值的变量；试试 `set -o pipefail`，它可以监测管道中的错误。当牵扯到很多脚本时，使用 `trap` 来检测 ERR 和 EXIT。一个好的习惯是在脚本文件开头这样写，这会使它能够检测一些错误，并在错误发生时中断程序并输出信息：
```bash
      set -euo pipefail
      trap "echo 'error: Script failed: see failed command above'" ERR
```

- 在 Bash 中，变量有许多的扩展方式。`${name:?error message}` 用于检查变量是否存在。此外，当 Bash 脚本只需要一个参数时，可以使用这样的代码 `input_file=${1:?usage: $0 input_file}`。在变量为空时使用默认值：`${name:-default}`。如果你要在之前的例子中再加一个（可选的）参数，可以使用类似这样的代码 `output_file=${2:-logfile}`，如果省略了 $2，它的值就为空，于是 `output_file` 就会被设为 `logfile`。数学表达式：`i=$(( (i + 1) % 5 ))`。序列：`{1..10}`。截断字符串：`${var%suffix}` 和 `${var#prefix}`。例如，假设 `var=foo.pdf`，那么 `echo ${var%.pdf}.txt` 将输出 `foo.txt`。

- 使用括号扩展（`{`...`}`）来减少输入相似文本，并自动化文本组合。这在某些情况下会很有用，例如 `mv foo.{txt,pdf} some-dir`（同时移动两个文件），`cp somefile{,.bak}`（会被扩展成 `cp somefile somefile.bak`）或者 `mkdir -p test-{a,b,c}/subtest-{1,2,3}`（会被扩展成所有可能的组合，并创建一个目录树）。

- 了解 Bash 中的“here documents”，例如 `cat <<EOF ...`。

- 在 Bash 中，同时重定向标准输出和标准错误：`some-command >logfile 2>&1` 或者 `some-command &>logfile`。通常，为了保证命令不会在标准输入里残留一个未关闭的文件句柄捆绑在你当前所在的终端上，在命令后添加 `</dev/null` 是一个好习惯。

- 使用 `screen` 或 [`tmux`](https://tmux.github.io/) 来使用多份屏幕，当你在使用 ssh 时（保存 session 信息）将尤为有用。而 `byobu` 可以为它们提供更多的信息和易用的管理工具。另一个轻量级的 session 持久化解决方案是 [`dtach`](https://github.com/bogner/dtach)。

- ssh 中，了解如何使用 `-L` 或 `-D`（偶尔需要用 `-R`）开启隧道是非常有用的，比如当你需要从一台远程服务器上访问 web 页面。

- 对 ssh 设置做一些小优化可能是很有用的，例如这个 `~/.ssh/config` 文件包含了防止特定网络环境下连接断开、压缩数据、多通道等选项：
```
      TCPKeepAlive=yes
      ServerAliveInterval=15
      ServerAliveCountMax=6
      Compression=yes
      ControlMaster auto
      ControlPath /tmp/%r@%h:%p
      ControlPersist yes
```

- 使用 [`percol`](https://github.com/mooz/percol) 或者 [`fzf`](https://github.com/junegunn/fzf) 可以交互式地从另一个命令输出中选取值。

- 使用 `fpp`（[PathPicker](https://github.com/facebook/PathPicker)）可以与基于另一个命令(例如 `git`）输出的文件交互。

- 将 web 服务器上当前目录下所有的文件（以及子目录）暴露给你所处网络的所有用户，使用：
`python -m SimpleHTTPServer 7777` （使用端口 7777 和 Python 2）或`python -m http.server 7777` （使用端口 7777 和 Python 3）。

- 将 shell 切换为其他用户，使用 `su username` 或者 `sudo - username`。加入 `-` 会使得切换后的环境与使用该用户登录后的环境相同。省略用户名则默认为 root。切换到哪个用户，就需要输入_哪个用户的_密码。

## 文件及数据处理

- 在当前目录下通过文件名查找一个文件，使用类似于这样的命令：`find . -iname '*something*'`。在所有路径下通过文件名查找文件，使用 `locate something` （但注意到 `updatedb` 可能没有对最近新建的文件建立索引，所以你可能无法定位到这些未被索引的文件）。

- 使用 [`ag`](https://github.com/ggreer/the_silver_searcher) 在源代码或数据文件里检索（`grep -r` 同样可以做到，但相比之下 `ag` 更加先进）。

- 了解如何使用 `sort` 和 `uniq`，包括 uniq 的 `-u` 参数和 `-d` 参数，具体内容在后文单行脚本节中。另外可以了解一下 `comm`。

- 了解如何使用 `cut`，`paste` 和 `join` 来更改文件。很多人都会使用 `cut`，但遗忘了 `join`。

- 了解如何使用 `tee` 将标准输入复制到文件甚至标准输出，例如 `ls -al | tee file.txt`。

- 要进行一些复杂的计算，比如分组、逆序和一些其他的统计分析，可以考虑使用 [`datamash`](https://www.gnu.org/software/datamash/)。

- 注意到语言设置（中文或英文等）对许多命令行工具有一些微妙的影响，比如排序的顺序和性能。大多数 Linux 的安装过程会将 `LANG` 或其他有关的变量设置为符合本地的设置。要意识到当你改变语言设置时，排序的结果可能会改变。明白国际化可能会使 sort 或其他命令运行效率下降*许多倍*。某些情况下（例如集合运算）你可以放心的使用 `export LC_ALL=C` 来忽略掉国际化并按照字节来判断顺序。

- 了解如何使用 `awk` 和 `sed` 来进行简单的数据处理。 参阅 [One-liners](#one-liners) 获取示例。

- 使用 [`repren`](https://github.com/jlevy/repren) 来批量重命名文件，或是在多个文件中搜索替换内容。（有些时候 `rename` 命令也可以批量重命名，但要注意，它在不同 Linux 发行版中的功能并不完全一样。）
```sh
      # 将文件、目录和内容全部重命名 foo -> bar:
      repren --full --preserve-case --from foo --to bar .
      # 还原所有备份文件 whatever.bak -> whatever:
      repren --renames --from '(.*)\.bak' --to '\1' *.bak
      # 用 rename 实现上述功能（若可用）:
      rename 's/\.bak$//' *.bak
```

- 根据 man 页面的描述，`rsync` 是一个快速且非常灵活的文件复制工具。它闻名于设备之间的文件同步，但其实它在本地情况下也同样有用。在安全设置允许下，用 `rsync` 代替 `scp` 可以实现文件续传，而不用重新从头开始。它同时也是删除大量文件的[最快方法](https://web.archive.org/web/20130929001850/http://linuxnote.net/jianingy/en/linux/a-fast-way-to-remove-huge-number-of-files.html)之一：
```sh
mkdir empty && rsync -r --delete empty/ some-dir && rmdir some-dir
```

- 若要在复制文件时获取当前进度，可使用 `pv`，[`pycp`](https://github.com/dmerejkowsky/pycp)，[`progress`](https://github.com/Xfennec/progress)，`rsync --progress`。若所执行的复制为block块拷贝，可以使用 `dd status=progress`。

- 使用 `shuf` 可以以行为单位来打乱文件的内容或从一个文件中随机选取多行。

- 了解 `sort` 的参数。显示数字时，使用 `-n` 或者 `-h` 来显示更易读的数（例如 `du -h` 的输出）。明白排序时关键字的工作原理（`-t` 和 `-k`）。例如，注意到你需要 `-k1，1` 来仅按第一个域来排序，而 `-k1` 意味着按整行排序。稳定排序（`sort -s`）在某些情况下很有用。例如，以第二个域为主关键字，第一个域为次关键字进行排序，你可以使用 `sort -k1，1 | sort -s -k2，2`。

- 标准的源代码对比及合并工具是 `diff` 和 `patch`。使用 `diffstat` 查看变更总览数据。注意到 `diff -r` 对整个文件夹有效。使用 `diff -r tree1 tree2 | diffstat` 查看变更的统计数据。`vimdiff` 用于比对并编辑文件。

- 拆分文件可以使用 `split`（按大小拆分）和 `csplit`（按模式拆分）。

- 操作日期和时间表达式，可以用 [`dateutils`](http://www.fresse.org/dateutils/) 中的 `dateadd`、`datediff`、`strptime` 等工具。

- 使用 `zless`、`zmore`、`zcat` 和 `zgrep` 对压缩过的文件进行操作。


## 单行脚本

一些命令组合的例子：

- 当你需要对文本文件做集合交、并、差运算时，`sort` 和 `uniq` 会是你的好帮手。具体例子请参照代码后面的，此处假设 `a` 与 `b` 是两内容不同的文件。这种方式效率很高，并且在小文件和上 G 的文件上都能运用（注意尽管在 `/tmp` 在一个小的根分区上时你可能需要 `-T` 参数，但是实际上 `sort` 并不被内存大小约束），参阅前文中关于 `LC_ALL` 和 `sort` 的 `-u` 参数的部分。
```sh
      sort a b | uniq > c   # c 是 a 并 b
      sort a b | uniq -d > c   # c 是 a 交 b
      sort a b b | uniq -u > c   # c 是 a - b
```

- 使用 `grep . *`（每行都会附上文件名）或者 `head -100 *`（每个文件有一个标题）来阅读检查目录下所有文件的内容。这在检查一个充满配置文件的目录（如 `/sys`、`/proc`、`/etc`）时特别好用。


- 计算文本文件第三列中所有数的和（可能比同等作用的 Python 代码快三倍且代码量少三倍）：
```sh
      awk '{ x += $3 } END { print x }' myfile
```

- 如果你想在文件树上查看大小/日期，这可能看起来像递归版的 `ls -l` 但比 `ls -lR` 更易于理解：
```sh
      find . -type f -ls
```

- 假设你有一个类似于 web 服务器日志文件的文本文件，并且一个确定的值只会出现在某些行上，假设一个 `acct_id` 参数在 URI 中。如果你想计算出每个 `acct_id` 值有多少次请求，使用如下代码：
```sh
      egrep -o 'acct_id=[0-9]+' access.log | cut -d= -f2 | sort | uniq -c | sort -rn
```

- 要持续监测文件改动，可以使用 `watch`，例如检查某个文件夹中文件的改变，可以用 `watch -d -n 2 'ls -rtlh | tail'`；或者在排查 WiFi 设置故障时要监测网络设置的更改，可以用 `watch -d -n 2 ifconfig`。

- 运行这个函数从这篇文档中随机获取一条技巧（解析 Markdown 文件并抽取项目）：
```sh
      function taocl() {
        curl -s https://raw.githubusercontent.com/jlevy/the-art-of-command-line/master/README-zh.md|
          pandoc -f markdown -t html |
          iconv -f 'utf-8' -t 'unicode' |
          xmlstarlet fo --html --dropdtd |
          xmlstarlet sel -t -v "(html/body/ul/li[count(p)>0])[$RANDOM mod last()+1]" |
          xmlstarlet unesc | fmt -80
      }
```

## 冷门但有用

- `expr`：计算表达式或正则匹配

- `m4`：简单的宏处理器

- `yes`：多次打印字符串

- `env`：执行一个命令（脚本文件中很有用）

- `printenv`：打印环境变量（调试时或在写脚本文件时很有用）

- `look`：查找以特定字符串开头的单词或行

- `cut`，`paste` 和 `join`：数据修改

- `fmt`：格式化文本段落

- `pr`：将文本格式化成页／列形式

- `fold`：包裹文本中的几行

- `column`：将文本格式化成多个对齐、定宽的列或表格

- `expand` 和 `unexpand`：制表符与空格之间转换

- `nl`：添加行号

- `seq`：打印数字

- [`gpg`](https://gnupg.org/)：加密并签名文件
