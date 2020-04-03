---
layout: course
title: Course overview + the shell
date: 2020-04-03 17:05:16
tags:
- course
- tools
---
## Course overview + the shell

### 动机
作为计算机科学家，我们知道计算机擅长于执行重复性任务。 但是，我们常常忘记了这与我们希望程序执行的计算同样适用
于我们对计算机的使用。 我们拥有触手可及的各种工具，使我们能够在处理任何与计算机相关的问题时提高工作效率并解
决更复杂的问题。 然而，我们中的许多人仅使用其中的一小部分。 我们只知道死记硬背了足够多的神奇咒语，而当我们陷
入困境时，只能从互联网上盲目地复制粘贴命令。

这门课程就是为了解决这个问题。

我们想教您如何充分利用已知的工具，向您展示要添加到工具箱中的新工具，并希望为您激发一些自己探索（或构建）更多工具的热情。 我们认为这是大多数计算机科学课程中缺少的一个学期。

### Topic 1: The Shell

#### Shell是什么

如今，计算机具有各种接口来为它们提供命令。精美的图形用户界面，语音界面，甚至AR / VR随处可见。
这些功能可满足80％的用例，但通常在允许您执行的操作上受到根本的限制_——您无法按下不存在的按钮或
发出尚未编程的语音命令。为了充分利用您的计算机提供的工具，我们必须走传统，然后使用文本界面
：The Shell。

您几乎可以接触到的所有平台都具有一种或多种形式的shell，其中许多平台都有多个shell供您选择。
尽管它们的细节可能有所不同，但它们的核心几乎都是相同的：它们使您可以运行程序，为它们提供输
入以及以半结构化的方式检查其输出。

在本讲座中，我们将重点介绍Bourne Again SHell，简称“ bash”。这是使用最广泛的shell之一，其语
法类似于您在许多其他shell中看到的语法。要打开shell提示符（您可以在其中键入命令），首先需要一
个终端。您的设备可能已安装了一个，也可以很容易地安装一个。

#### 使用shell

启动终端时，您会看到一个提示，通常看起来像这样：
~~~bash
missing:~$
~~~

这是外壳的主要文本界面。 它告诉您这台机器是`missing`，并且“当前工作目录”或当前位置为`〜`（“ home”
的缩写）。 `$`告诉您您不是root用户（稍后将进行介绍）。 在此提示符下，您可以键入命令，然后由外壳
程序解释。 最基本的命令是执行程序：
~~~bash
missing:~$ date
Fri 03 Apr 2020 05:55:02 PM CST
missing:~$
~~~
在这里，我们执行了日期程序，该程序可能会打印出当前日期和时间,
然后，`shell`要求我们执行另一个命令。 我们还可以执行带有参数的命令：
~~~bash
missing:~$ echo hello
hello
~~~

在这种情况下，我们告诉`shell`使用参数`hello`执行程序`echo`。 `echo`程序只是打印出其参数。 shell
程序通过按空格将命令拆分来解析命令，然后运行第一个单词指示的程序，并提供每个后续单词作为程序可以
访问的参数。 如果要提供包含空格或其他特殊字符的参数（例如，名为“My Photos”的目录），则可以用'或“
（（”My Photos“））引号，或者仅使用（My\  Photos）。

但是shell如何知道如何找到日期或回显程序？ 好的，shell是一个编程环境，就像Python或Ruby一样，因此
它具有变量，条件，循环和函数（下一节！）。 在shell中运行命令时，实际上是在编写shell解释的一小
段代码。 如果要求shell程序执行与其编程关键字之一不匹配的命令，它将查询名为`$PATH`的环境变量，该
变量列出了外壳程序在收到命令后应在哪个目录中搜索程序：
~~~bash
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
~~~

当我们运行echo命令时，shell会看到它应该执行程序echo，然后在`$PATH`中以：分隔的目录列表中搜索该名
称的文件。 找到它后，它将运行它（假定文件是可执行文件；稍后会对此进行更多介绍）。 我们可以使用哪
个程序找出给定程序名执行的文件。 我们还可以通过提供要执行的文件的路径来完全绕过`$PATH`。

#### Navigating in the shell

Shell上的路径是定界的目录列表。 在Linux和macOS上用`/`分隔，在Windows上用`\`分隔。 在Linux和macOS
上，路径`/`是文件系统的“根”，所有目录和文件都位于该根下，而在Windows上，每个磁盘分区都有一个根（例如C：\）。
我们通常会假设您在此类中使用Linux文件系统。 以`/`开头的路径称为绝对路径。 任何其他路径都是相对路径。
相对路径是相对于当前工作目录的，我们可以在`pwd`命令中看到，而在`cd`命令中更改。 在路径中，`.` 引用当前目录，`..`引用其父目录：

~~~bash
missing:~$ pwd
/home/missing
missing:~$ cd /home
missing:/home$ pwd
/home
missing:/home$ cd ..
missing:/$ pwd
/
missing:/$ cd ./home
missing:/home$ pwd
/home
missing:/home$ cd missing
missing:~$ pwd
/home/missing
missing:~$ ../../bin/echo hello
hello
~~~

注意，我们的shell提示符使我们了解当前的工作目录。 您可以配置提示以显示各种有用的信息，
我们将在以后的讲座中介绍这些信息。

通常，当我们运行程序时，除非另行说明，否则它将在当前目录中运行。 例如，它通常会在此处搜索文件，并在需要时在此处创建新文件。
要查看给定目录中包含的内容，我们使用`ls`命令：
~~~bash
missing:~$ ls
missing:~$ cd ..
missing:/home$ ls
missing
missing:/home$ cd ..
missing:/$ ls
bin
boot
dev
etc
home
...
~~~

除非给出目录作为其第一个参数，否则`ls`将打印当前目录的内容。 大多数命令接受以`-`开头的标志和选项（带有值的标志）以修改其行为。
通常，运行带有`-h`或`--help`标志（在Windows上是`/？`）的程序会打印一些帮助文本，告诉您哪些标志和选项可用。 例如，`ls --help`
告诉我们：
~~~bash
missing:~$ ls -l /home
drwxr-xr-x 1 missing  users  4096 Jun 15  2019 missing
~~~

这为我们提供了有关当前每个文件或目录的更多信息。 首先，
该行开头的d告诉我们`missing`是目录。 然后跟随三组三个字符（`rwx`）。 这些指示文件所有者（missing），拥有组（users）
和其他所有人分别对相关项目具有什么权限。 `-`表示给定的主体没有给定的权限。 在上面，仅允许所有者
修改（w）missing的目录（即在其中添加/删除文件）。 要输入目录，用户必须对该目录（及其父目录）具有
“搜索”（由“执行”：x表示）权限。 要列出其内容，用户必须对该目录具有读取（r）权限。 对于文件，权
限与您期望的一样。 请注意，`/bin`中几乎所有文件都为最后一个组“其他任何人”设置了x权限，因此任何
人都可以执行这些程序。

此时需要了解的其他一些便捷程序是`mv`（用于重命名/移动文件），`cp`（用于复制文件）和`mkdir`（用于创建新
目录）。

如果您想了解有关程序的参数，输入，输出或其总体运行方式的更多信息，请尝试使用`man`程序。 它以程序名称作为
参数，并向您显示其手册页。 按`q`退出。

~~~bash
missing:~$ man ls
~~~

#### Connecting programs
在shell程序中，程序有两个与之关联的主要“流”：它们的输入流和输出流。 当程序尝试读取输入时，它将
从输入流中读取，并且在打印某些内容时，将打印到其输出流中。 通常，程序的输入和输出都是您的终端。
也就是说，您的键盘作为输入，屏幕作为输出。 但是，我们也可以重新连接这些流！

重定向的最简单形式是`< file`和`> file`。 这些使您可以将程序的输入和输出流分别重新连接到文件：

~~~bash
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
~~~

您也可以使用`>>`追加到文件。 这种输入/输出重定向真正发挥作用的地方是管道的使用。
`|` 运算符可让您“链接”程序，以使一个程序的输出为另一程序的输入：

~~~bash
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
~~~

在有关数据争用的讲座中，我们将详细介绍如何利用管道。

#### A versatile and powerful tool

在大多数类Unix系统上，一个用户是特殊的：`root`用户。 您可能已经在上面的文件列表中看到了它。 
`root`用户具有（几乎）所有访问限制，并且可以创建，读取，更新和删除系统中的任何文件。 不过，
由于通常很容易意外破坏某些内容，因此您通常不会以`root`用户身份登录系统。 相反，您将使用`sudo`
命令。 顾名思义，它可以让您“做”“ su”（“超级用户”或“ root”的缩写）做某事。 当您遇到权限被拒绝的
错误时，通常是因为您需要以root用户身份执行操作。 虽然请确保您首先仔细检查您是否真的想那样做！

要成为root用户，需要做的一件事就是写入`/sys`下挂载的sysfs文件系统。 sysfs将许多内核参数公开为
文件，因此您可以在不使用专门工具的情况下轻松地即时重新配置内核。 请注意，在Windows或macOS上
不存在`sysfs`。

例如，笔记本电脑屏幕的亮度通过一个名为`brightness`
~~~bash
/sys/class/backlight
~~~

通过将值写入该文件，我们可以更改屏幕亮度。 您首先可以执行以下操作：
~~~bash
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
~~~

该错误可能令人惊讶。 毕竟，我们使用sudo运行了该命令！ 这是有关shell的重要知识。 诸如`|`，`>`和`<`
之类的操作由shell程序完成，而不是由单个程序完成。 `echo`并不“知道” |。 他只是从输入中读取并写入输
出，无论它是什么。 在上述情况下，shell程序（已通过您的用户身份验证）会尝试打开亮度文件进行写入
，然后再将其设置为sudo echo的输出，但由于shell程序无法以root用户身份运行，因此无法这样做。
利用这些知识，我们可以解决此问题：

~~~bash
$ echo 3 | sudo tee brightness
~~~

由于`tee`程序是打开`/sys`文件进行写入的程序，并且它以root身份运行，因此所有权限都可以使用。 
您可以通过`/sys`控制各种有趣和有用的事情，例如各种系统LED的状态（您的路径可能不同）：

~~~bash
$ echo 1 | sudo tee /sys/class/leds/input6::scrolllock/brightness
~~~

#### Next steps

至此，您已经了解了足以完成基本任务的外壳程序。 您应该能够浏览找到感兴趣的文件并使用大多数程序
的基本功能。 在下一个演讲中，我们将讨论如何使用Shell和许多方便的命令行程序执行和自动化更复杂的任务。
