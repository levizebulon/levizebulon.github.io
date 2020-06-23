---
title: i3 User’s Guide
date: 2020-06-22 18:34:46
tags:
- software
---

# i3 User’s Guide

[https://i3wm.org/docs/userguide.html](https://i3wm.org/docs/userguide.html)

## 1. Default keybindings

请注意，在没有配置文件的情况下启动i3时，无论您使用的键盘布局如何，`i3-config-wizard`都会为您提供一个配置文件，
该文件中的键位置与您在上图中看到的相匹配。 如果您更喜欢配置文件，并将配置基于`/etc/i3/config`。

## 2. Using i3

在整个指南中，关键字`$mod`将用于指配置的修饰符。默认情况下，这是Alt键(`Mod1`)，而Windows键(`Mod4`)是一个流行的替代方法，它很大程度上防止了与应用程序定义的快捷方式冲突。

### 2.1. Opening terminals and moving around

一个非常基本的操作是开设一个新的终端。默认情况下，这个键绑定是`$mod+Enter`，也就是默认配置中的Alt+Enter (`Mod1+Enter`)。按`$mod+Enter`键，将打开一个新的终端。它会填满你屏幕上的所有可用空间。

如果您现在打开另一个终端，i3将把它放在当前终端的旁边，将屏幕大小分成两半。根据您的显示器，i3将把创建的窗口放在现有窗口的旁边(宽显示)或下面的现有窗口(旋转显示)。

要在两个终端之间移动，可以使用你在编辑器`vi`中知道的方向键。但是，在i3中，homerow用于这些键(在`vi`中，为了与大多数键盘布局兼容，这些键要向左移动一个)。因此，`$mod+j`是向左，`$mod+k`是向下，`$mod+l`是向上，`$mod+;`是向右。因此，要在终端之间切换，使用`$mod+k`或`$mod+l`。当然，也可以使用方向键。

目前，您的工作区按照特定的方向(默认为水平)分割(它包含两个终端)。每个窗口都可以被水平分割或垂直分割，就像工作空间一样。术语是“window”，指实际包含X11窗口(如终端或浏览器)的容器，而“split container”指包含一个或多个窗口的容器。

要垂直分割一个窗口，在创建新窗口之前按`$mod+v`。水平分割，按`$mod+h`。

### 2.2 更改容器布局

分割容器可以具有以下布局之一

splith/splitv : 窗口的大小是为了使每个窗口在容器中获得相等的空间。splith水平分布窗口(窗口相邻)，splitv垂直分布窗口(窗口重叠)。

stacking : 只显示容器中聚焦的窗口。您将在容器的顶部获得一个窗口列表。

tabbed : 和堆叠的原理是一样的，但是顶部的窗口列表只有一条线，是垂直分开的。

To switch modes, press `$mod+e` for splith/splitv (it toggles), `$mod+s` for stacking and `$mod+w` for tabbed.

### 2.3 为一个窗口切换全屏模式

要在全屏模式下显示一个窗口或再次退出全屏模式，按`$mod+f`。

在i3中还有一个全局全屏模式，在该模式中，客户端将跨越所有可用输出(命令为全屏切换全局)。

### 2.4 打开其他应用程序

除了从终端打开应用程序，你还可以使用方便的`dmenu`，默认按`$mod+d`打开。只需键入您想要打开的应用程序的名称(或它的一部分)。要使其工作，相应的应用程序必须位于`$PATH`中。

另外，如果您有经常打开的应用程序，您可以创建一个键绑定来直接启动应用程序。有关详细信息，请参阅[配置]一节。

### 2.5 关闭窗口

如果一个应用程序没有提供关闭机制(大多数应用程序提供一个菜单，escape键或快捷键如`Control+w`来关闭)，你可以按`$mod+Shift+q`来杀死一个窗口。对于支持WM_DELETE协议的应用程序，这将正确地关闭应用程序(保存任何修改或进行其他清理)。如果应用程序不支持WM_DELETE协议，你的X server将杀死窗口的行为取决于应用程序。

### 2.6. Using workspaces

工作区是对一组窗口进行分组的一种简单方法。默认情况下，您处于第一个工作区中，正如左下角的工具栏所示。要切换到另一个工作空间，请按`$mod+num`，其中`num`是您要使用的工作空间的编号。如果工作空间还不存在，它将被创建。

一种常见的范例是将web浏览器放在一个工作空间中，将通信应用程序(`mutt`、`irssi`...)放在另一个工作空间中，将您使用的应用程序放在第三个工作空间中。当然，没有必要遵循这种方法。

如果您有多个屏幕，那么在启动时将在每个屏幕上创建一个工作区。如果您打开一个新的工作区，它将被绑定到您创建它的屏幕上。当您切换到另一个屏幕上的工作区时，i3将把焦点设置到该屏幕上。

### 2.7 将窗口移动到工作区

要将一个窗口移动到另一个工作空间，只需按`$mod+Shift+num`，其中`num`是目标工作空间的数量(如切换工作空间时)。与切换工作区类似，如果目标工作区还不存在，则将创建它。

### 2.8 Resizing

调整容器大小最简单的方法是使用鼠标:抓取边框并将其移动到所需的大小。

你也可以使用[绑定模式]来定义通过键盘调整大小的模式。要查看相关示例，请查看i3提供的[默认配置](https://github.com/i3/i3/blob/next/etc/config.keycodes)。

### 2.9 原地重启i3

要在原地重新启动i3(从而在出现bug时进入干净状态，或者升级到新的i3版本)，可以使用`$mod+Shift+r`。

### 2.10  退出i3

要干净地退出i3而不杀死X server，你可以使用`$mod+Shift+e`。默认情况下，会有一个对话框要求您确认是否真的要退出。

### 2.11 浮动

浮动模式与平铺模式相反。窗口的位置和大小不是由i3自动管理的，而是由您手动管理的。使用这种模式违背了平铺模式，但对于一些特殊情况很有用，比如“另存为”对话框窗口或工具栏窗口(GIMP或类似的)。这些窗口通常设置适当的提示，并在默认情况下以浮动模式打开。

你可以通过按`$mod+Shift+Space`来切换浮动模式。通过拖动窗口的标题栏与您的鼠标，您可以移动窗口周围。通过抓取边框并移动它们，可以调整窗口的大小。你也可以使用[floating_modifier]来做到这一点。另一种使用鼠标调整浮动窗口大小的方法是右键单击标题栏并拖动。

要使用键盘调整浮动窗口的大小，请参阅i3[默认配置](https://github.com/i3/i3/blob/next/etc/config.keycodes)提供的调整绑定模式。

浮动窗口总是在平铺窗口的顶部。

## 3 树

i3将关于X11输出、工作区和窗口布局的所有信息存储在一个树中。根节点是X11根窗口，后面是X11输出，然后是停靠区域和内容容器，然后是工作区，最后是窗口本身。在i3的早期版本中，我们有多个列表(输出、工作区)和每个工作区的表格。这种方法的使用、理解和实现都很复杂。

### 3.1 树由容器组成

我们树的构建块就是所谓的容器。容器可以托管窗口(即X11窗口，可以像浏览器一样实际看到和使用)。或者，它可以包含一个或多个容器。一个简单的示例是工作区:当您使用单个显示器、单个工作区启动i3并打开两个终端窗口时，您将得到这样一个树

### 3.2 方向和拆分容器

当使用树作为数据结构时，使用所谓的拆分容器来构建布局是很自然的。在i3中，每个容器都有一个方向(水平、垂直或未指定的)，方向取决于容器的布局(垂直用于splitv和stacking，水平用于splith和tabed)。因此，在我们的工作区示例中，工作区容器的默认布局是splith(现在大多数监视器都是宽屏的)。如果您将布局改为splitv(默认配置中的`$mod+v`)，然后打开两个终端，i3将这样配置您的windows

自从版本4以来，i3的一个有趣的新特性就是能够分割任何东西:让我们假设您在一个工作区上有两个终端(具有splith布局，即水平方向)，焦点在右边的终端上。现在您想在当前窗口下面打开另一个终端窗口。如果您只是打开一个新的终端窗口，由于splith布局，它将在右侧显示。相反，按`$mod+v`以使用splitv布局分割容器(使用`$mod+h`打开水平分割容器)。现在您可以打开一个新的终端，它将在当前终端下面打开

你可能已经猜到了:分叉的深度是没有限制的。

### 3.3 Focus parent

让我们继续以上面的例子为例。我们有一个终端在左边和两个垂直分裂的终端在右边，焦点是在右下角的一个。当您打开一个新的终端时，它将在当前终端下面打开。

那么，如何在当前终端窗口的右侧打开新的终端窗口呢？解决方案是使用焦点父容器，它将焦点放在当前容器的父容器上。在默认配置中，使用`$mod+a`在树中导航一个容器(您可以多次重复此操作，直到到达工作区容器)。在本例中，您将关注位于水平方向工作区内的垂直分隔容器。因此，现在将在垂直分隔容器的右侧打开新的窗口

### 3.4 隐式容器

在某些情况下，i3需要隐式地创建一个容器来实现您的命令。

其中一个示例是以下场景:使用一个显示器和一个工作区启动i3，在该工作区上打开三个终端窗口。所有这些终端窗口都直接连接到i3布局树中的一个节点，即工作区节点。默认情况下，工作空间节点的方向是水平的。

现在将其中一个终端向下移动(默认为`$mod+Shift+k`)。工作空间节点的方向将改变为垂直方向。您向下移动的终端窗口直接连接到工作区，并出现在屏幕的底部。创建了一个新的(水平的)容器来容纳其他两个终端窗口。当切换到选项卡模式(例如)时，您将注意到这一点。你最终会有一个标签表示分隔容器(例如，“H[urxvt firefox]”)，而另一个是你向下移动的终端窗口。

## 4 配置i3

这是真正的乐趣开始;-)。大多数事情都非常依赖于你理想的工作环境，所以我们不能为它们设定合理的默认值。

虽然没有使用编程语言进行配置，但i3在您通常希望窗口管理器执行的操作方面保持了相当的灵活性。

例如，您可以配置绑定以跳转到特定的窗口，您可以设置特定的应用程序以在特定的工作区上启动，您可以自动启动应用程序，您可以更改i3的颜色，并且您可以绑定您的键来做一些有用的事情。

要更改i3的配置，请将`/etc/i3/config`复制到`～/.i3/config`（如果喜欢XDG目录方案，则复制到`〜/.config/i3/config`），然后使用文本编辑器进行编辑。

在第一次启动时(以及以后所有启动时，除非您有配置文件)，i3将提供创建配置文件的功能。您可以告诉向导在配置文件中使用Alt (Mod1)或Windows (Mod4)作为修改器。此外，创建的配置文件将使用当前键盘布局的关键符号。要启动向导，使用命令i3-config-wizard。请注意，您必须没有`~/.i3/config`，否则向导将退出。

从i3 4.0开始，使用了一种新的配置格式。i3将尝试根据几个不同的关键字自动检测配置文件的格式版本，但是如果你想确保你的配置是用新的格式读取的，在配置文件中包括下面的行

~~~bash
# i3 config file (v4)
~~~

### 4.1 注释

可以并且建议在配置文件中使用注释来正确地记录您的设置，以便以后参考。注释以#开头并且只能用在一行的开头。

例子：

~~~bash
# This is a comment
~~~

### 4.2 字体

i3支持X核心字体和自由类型字体(通过Pango)来呈现窗口标题。

要生成X core字体描述，可以使用xfontsel(1)。要查看特殊字符(Unicode)，需要使用支持ISO-10646编码的字体。

FreeType字体描述由字体族、样式、重量、变体、扩展和大小组成。自由类型字体支持从右到左的呈现，并且通常包含比X核心字体更多的Unicode符号。

如果i3无法打开配置的字体，它将在日志文件中输出一个错误并返回到工作字体。

Syntax:
<div class="content">
<pre><tt>font &lt;X core font description&gt;
font pango:&lt;family list&gt; [&lt;style options&gt;] &lt;size&gt;</tt></pre>
</div>

Examples:
~~~bash
font -misc-fixed-medium-r-normal--13-120-75-75-C-70-iso10646-1
font pango:DejaVu Sans Mono 10
font pango:DejaVu Sans Mono, Terminus Bold Semi-Condensed 11
font pango:Terminus 11px
~~~

### 4.3 键盘绑定

键盘绑定使i3在按下特定键后执行命令(见下面)。i3允许您绑定按键代码或按键符号(您也可以混合绑定，尽管i3不能防止重叠的绑定)。

- 按键符号是对特定符号的描述,如a或b,但也包括一些更奇怪的符号，如下划线而不是。这些是您在Xmodmap中用于重新映射密钥的键。要获得键盘的当前映射，请使用`xmodmap -pke`。要交互式地输入一个键并查看它被配置为什么keysym，请使用`xev`。

- 键码不需要指定符号(对于某些笔记本上的自定义供应商热键很方便)，而且当切换到不同的键盘布局时(使用xmodmap时)，它们的含义不会改变。

我的建议是:如果您经常切换键盘布局，但又希望将绑定保持在键盘上相同的物理位置，那么请使用键码。如果你不想切换布局，想要一个干净简单的配置文件，使用键盘。

某些工具（例如import或xdotool）可能无法在KeyPress事件上运行，因为仍然抓住了键盘/指针。 在这些情况下，可以使用--release标志，它将在释放键后执行命令。

Syntax:

~~~bash
bindsym [--release] [<Group>+][<Modifiers>+]<keysym> command
bindcode [--release] [<Group>+][<Modifiers>+]<keycode> command
~~~

~~~bash
# Fullscreen
bindsym $mod+f fullscreen toggle

# Restart
bindsym $mod+Shift+r restart

# Notebook-specific hotkeys
bindcode 214 exec --no-startup-id /home/michael/toggle_beamer.sh

# Simulate ctrl+v upon pressing $mod+x
bindsym --release $mod+x exec --no-startup-id xdotool key --clearmodifiers ctrl+v

# Take a screenshot upon pressing $mod+x (select an area)
bindsym --release $mod+x exec --no-startup-id import /tmp/latest-screenshot.png
~~~

可用的修饰符

`Mod1-Mod5, Shift, Control`
- Standard modifiers, see xmodmap(1)

` Group1, Group2, Group3, Group4 `

- 当使用多个键盘布局(例如`setxkbmap -layout us`,ru)时，您可以指定在哪个XKB组(也称为布局)中激活键绑定。默认情况下，键绑定在Group1中进行翻译，并且在所有组中都是活动的。如果想要覆盖某个布局中的键绑定，请指定相应的组。为了向后兼容，组模式开关是Group2的别名。

### 4.4 鼠标绑定

鼠标绑定使i3在单击容器范围内的特定鼠标按钮时执行命令(参见[命令标准])。您可以以与键绑定类似的方式配置鼠标绑定。

Syntax:

<div class="content">
<pre><tt>bindsym [--release] [--border] [--whole-window] [--exclude-titlebar] [&lt;Modifiers&gt;+]button&lt;n&gt; command</tt></pre>
</div>

默认情况下，只有在单击窗口的标题栏时，绑定才会运行。如果给定了`--release`标志，它将在释放鼠标按钮时运行。

如果指定了`--whole-window`标志，那么当单击窗口的任何部分时，绑定也将运行，除了边框之外。若要在单击边框时运行绑定，请指定`--border`标志。

如果给定了`--exclude-titlebar`标志，则将不会考虑将titlebar用于键绑定。

Examples:

<div class="content">
<pre><tt># The middle button over a titlebar kills the window
bindsym --release button2 kill

# The middle button and a modifer over any part of the window kills the window
bindsym --whole-window $mod+button2 kill

# The right button toggles floating
bindsym button3 floating toggle
bindsym $mod+button3 floating toggle

# The side buttons move the window around
bindsym button9 move left
bindsym button8 move right</tt></pre>
</div>

### 4.5 绑定模式

通过使用不同的绑定模式，您可以拥有多组绑定。当切换到另一种绑定模式时，当前模式下的所有绑定都将被释放，只要您保持该绑定模式，就只有在新模式下定义的绑定有效。唯一预定义的绑定模式是`default`，即i3开始时的模式，所有未在特定绑定模式中定义的绑定都属于该模式。

使用绑定模式包括两个部分:定义绑定模式和切换到它。为了达到这些目的，有一个配置指令和一个命令，它们都被称为模式。指令用于定义属于特定绑定模式的绑定，而命令将切换到指定模式。

为了更容易维护，建议结合使用绑定模式和[变量]。下面是如何使用绑定模式的示例。

注意，为切换回默认模式，最好定义绑定。

注意，可以对绑定模式使用[pango_markup](https://i3wm.org/docs/userguide.html#pango_markup)，但是需要通过将`--pango_markup`标志传递给模式定义来显式地启用它。

<strong>Syntax:<\strong>

<div class="listingblock">
<div class="content">
<pre><tt># config directive
mode [--pango_markup] &lt;name&gt;

# command
mode &lt;name&gt;</tt></pre>
</div></div>

<strong>Examples:<\strong>

<div class="listingblock">
<div class="content">
<pre><tt># Press $mod+o followed by either f, t, Escape or Return to launch firefox,
# thunderbird or return to the default mode, respectively.
set $mode_launcher Launch: [f]irefox [t]hunderbird
bindsym $mod+o mode "$mode_launcher"

mode "$mode_launcher" {
    bindsym f exec firefox
    bindsym t exec thunderbird

    bindsym Escape mode "default"
    bindsym Return mode "default"
}</tt></pre>
</div></div>


