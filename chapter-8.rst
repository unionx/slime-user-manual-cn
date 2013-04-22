八、扩展包
========

在3.0版中，我们将一些功能移到了单独的包里。本章讲述如何加载扩展包的模块，并且描述了这些包的功能。

8.1 加载扩展包
-------------

默认情况下扩展包并没有被加载。你必须稍微设置一下，这样Emacs就可以知道在哪里找到这些扩展包、加载哪些扩展包。总的来说，你应该调用slime-setup函数，并将需要用的包的名字作为一个列表传给它。例如，加载slime-scratch和slime-editing-commands包的设置如下：

::

   (setq inferior-lisp-program "/opt/sbcl/bin/sbcl") ; your Lisp system
   (add-to-list 'load-path "~/hacking/lisp/slime/")  ; your SLIME directory
   (require 'slime-autoloads)
   (slime-setup '(slime-scratch slime-editing-commands))

启动Slime后，这些扩展包的命令应该都可用了。

这里要特别提到REPL和slime-fancy扩展包。许多用户认为REPL（见 8.2 REPL）很必要，而slime-fancy（见 8.20 slime-fancy）加载了REPL包和其它几乎所有常用的包。所以，如果你不知道怎样启动，试试：

::

   (slime-setup '(slime-repl)) ; repl only

如果你喜欢你见到的，试试：

::

   (slime-setup '(slime-fancy)) ; almost everything

8.2 REPL：“顶层环境”
------------------

Slime使用定制的读-写-打印循环（REPL，也被称作“顶层环境”，或者是监听者）。REPL界面使用Emacs Lisp写成，因此与传统的基于comint的Lisp交互相比，它与Emacs更加契合：

* 可以通过SLDB调试REPL里表达式的状况信号

* 打印出的不同的返回值可以通过不同的Emacs faces（颜色）加以区分

* Emacs通过不同的符号管理REPL提示符。这确保了Lisp的输出被插入到正确的位置，并且不会和其它的用户输入搞混。

加载REPL需要在你的.emacs文件里调用(slime-setup '(slime-repl))。

* C-c C-z 或 M-x slime-switch-to-output-buffer

  选择输出缓冲区，一般是在另一个窗口里

* C-c C-y 或 M-x slime-call-defun

  在REPL里调用在当前光标处定义的函数

8.2.1 REPL命令
^^^^^^^^^^^^

* RET 或 M-x slime-repl-return

  如果输入完成，则在Lisp里对当前输入求值。如果没有，则开启一个新行并缩进。如果给出了前缀参数，则不检查输入完整性而直接求值。

* C-RET 或 M-x slime-repl-closing-return

  关闭所有未匹配的括号并对当前输入求值。也绑定到了M-RET。

* TAB 或 M-x slime-indent-and-complete-symbol

  缩进当前行并进行符号补全。

* C-j 或 M-x slime-repl-newline-and-indent

  新开一行并缩进。

* C-a 或 M-x slime-repl-bol

  到行首，但是在REPL提示符之后。

* C-c C-c 或 M-x slime-interrupt

  以SIGINT中断Lisp进程。

* C-c M-o 或 M-x slime-repl-clear-buffer

  清空当前缓冲区，只留一个提示符。

* C-c C-o 或 M-x slime-repl-clear-output

  从缓冲区里清空之前表达式的输出和结果

8.2.2 输入引导
^^^^^^^^^^^^

输入导航（历史）命令是模仿coming-mode的。如果你习惯了类似Bash的键绑定，那么要注意了： M-p和M-n使用当前的输入作为搜索样本，并且类似Bash一样只有在当前行是空的情况下才工作。C-<up>和C-<down>像Bash里的up和down键效果一样。

* C-<up> 或 M-x slime-repl-forward-input 和 C-<down> 和 M-x slime-repl-backward-input

  去到前一个/后一个历史输入

* M-n 或 M-x slime-repl-next-input 和 M-p 或 M-x slime-repl-previous-input

  使用当前的输入作为搜索样本，在输入历史里向前/向后搜索相关输入。如果M-n/M-n被连续按两次，第二次调用会使用相同的搜索样本（即使当前输入已经改变）。

* M-s 或 M-x slime-repl-next-matching-input 和 M-r 或 M-x slime-repl-previous-matching-input

  使用正则表达式在输入历史里向前/向后搜索

* C-c C-n 或 M-x slime-repl-next-prompt 和 C-c C-p 或 M-x slime-repl-previous-prompt

  在REPL缓冲区的当前和前一个提示符之间移动。在有之前输入的一行里按RET会把那一行复制到最新的提示符处。

slime-repl-wrap-history变量控控制了环绕行为，即是如果到了末尾那么是否应该重新跳转到历史的最开头。

8.2.3 快捷命令
^^^^^^^^^^^^

“快捷命令”是一组通过名称调用的REPL命令。要调用一个快捷命令，你需要先在REPL提示符后输入一个逗号，然后再输入命令的名称来执行。

快捷命令处理一些类似于切换目录和加载编译Lisp系统的事务。快捷命令在下面列出，或者你可以通过help快捷命令来交互式地列出来。

* change-directory (aka !d, cd)

  改变当前目录

* change-package (aka !p, in, in-package)

  改变当前包

* compile-and-load (aka cl)

  编译（如果）并加载一个Lisp文件

* defparameter (aka !)

  定义一个新的全局的特殊的变量

* disconnect

  关闭所有连接

* help (aka ?)

  显示帮助

* pop-directory (aka -d)

  弹出当前目录

* pop-package (aka -p)

  弹出包栈的顶端元素

* push-directory (aka +d, pushd)

  将一个新的目录推到目录栈里

* push-package (aka +p)

  将一个包推到包栈里

* pwd

  显示当前目录

* quit

  退出Lisp

* resend-form

  再次发送最后的形式

* restart-inferior-lisp

  重启*inferior-lisp*并重新连接Slime

* sayoonara

  退出所有Lisp并关闭所有Slime缓冲区

8.3 多REPL
---------

slime-mrepl扩展包为多监听者缓冲区提供了支持。M-x slime-open-listener命令创建一个新的缓冲区。在多线程Lisp里，每一个监听者都与一个单独的线程相连。在单线程Lisp里，创建多监听者缓冲区也是可以的，但是其命令都是在同一个进程里顺序执行的。

8.4 inferior-slime-mode
-----------------------

inferior-slime-mode是一个用来与*inferior-lisp*缓冲区一起使用的次模式。它提供了一些Slime命令，例如符号补全和文档查询。它也跟踪Lisp进程的当前目录。将以下代码加入.emacs配置来使用它：

::

   (slime-setup '(inferior-slime-mode))

* M-x inferior-slime-mode

  打开或关闭inferior-slime-mode

inferior-slime-mode-map变量包含了额外的键绑定

8.5 混合补全
----------

slime-c-p-c扩展包提供了不同的符号补全算法，它通过中划线分割的符号名 [#f1]_ 的单词子串来进行“并行”的补全。形式上来讲，“a-b-c”可以补全任何匹配“^a.*-b.*-c.*”正则表达式的符号（“圆点”匹配任何除了中划线之外的东西）。下面的例子会给你更直观的感觉：

* m-v-b补全为multiple-value-bind。

* w-open稍有歧义：它可以补全with-open-file或with-open-stream。它会扩展到最长的相同匹配（with-open-）然后光标会停留在有歧义的第一个字符处，在这里就是最后一个单词处。

* w--stream扩展为with-open-stream

slime-c-p-c-unambiguous-prefix-p变量定义了在补全符号后光标应该置于何处。例如f-o可能的补全是finish-output和force-output，默认情况下光标会移动到f后面，因为这里是明确的前缀。如果f-o are finish-output and force-output是nil，光标会到插入的文本的最后，在这里就是在o之后。

除此之外，slime-c-p-c也为字符名提供补全（对很多可以识别Unicode的Lisp实现来讲通常很有用）：

::

   CL-USER> #\Sp<TAB>

在这里Slime会将其补全为#\Space，但在一个可以识别Unicode的实现里，就可能会有以下的补全：

::

   Space                              Space
   Sparkle                            Spherical_Angle
   Spherical_Angle_Opening_Left       Spherical_Angle_Opening_Up

slime-c-p-c扩展包也提供了对关键字的大小写敏感的补全。例如：

::

   CL-USER> (find 1 '(1 2 3) :s<TAB>

在这里Slime会补全为:start，而不是将所有以:s开头的关键字列出来。

* C-c C-s 或 M-x slime-complete-form

如果有的话，将当前光标处的函数的参数列表列出来并插入缓冲区。更加一般地，此命令给不完全的形式的缺失参数提供了一个模板。对于发现泛函数的额外参数，处理make-instance、defmethod和其它很多函数来说有特殊的代码，例如：

::

   (subseq "abc" <C-c C-s>
            --inserts--> start [end])
   (find 17 <C-c C-s>
            --inserts--> sequence :from-end from-end :test test
            :test-not test-not :start start :end end
            :key key)
   (find 17 '(17 18 19) :test #'= <C-c C-s>
             --inserts--> :from-end from-end
             :test-not test-not :start start :end end
             :key key)
   (defclass foo () ((bar :initarg :bar)))
   (defmethod print-object <C-c C-s>
              --inserts-->   (object stream)
              body...)
   (defmethod initialize-instance :after ((object foo) &key blub))
   (make-instance 'foo <C-c C-s>
                   --inserts--> :bar bar :blub blub initargs...)

8.6 模糊补全
-----------

slime-fuzzy扩展包提供了另一种符号补全方式。

[最好有人描述一下这种算法到底是做什么的]

它尝试一次性补全整个符号，而不是只补全一部分。例如，“mvb”会补全为“multiple-value-bind”，“norm-df”会补全为“least-positive-normalized-double-float”。

这种算法尝试以不同的方式扩展每一个字符，然后以下列的方式将所有可能的补全排序列出。

根据在字符串里的位置，字母会被赋予一个权值。字符串最开头，或者是前缀字母之后的字母的权值是最高的。分隔符之后的字符，例如#\-，权值是次高的。字符串最后或者是后缀字母之前的字母有中等权值，其它地方的字母的权值最低。

如果一个字母在另一个匹配字母之后，它在此处的可能性就比之前字母的可能性低，所以就会使用之前的可能性。

最后，一个偏好因子会作用于一些常用的较短的匹配，其它的东西都是一样的。

* C-c M-i 或 M-x slime-fuzzy-complete-symbol

  根据当前光标处的缩写列出所有可能的补全。如果你将变量slime-complete-symbol-function的值设为这个命令，则可以通过M-TAB使用模糊补全。

8.7 slime**autodoc**mode
------------------------

Autodoc模式是一个用来自动显示光标附近符号的相关信息的minor-mode。对于函数名，参数列表会被显示，对于全局变量，则显示它的值。Autodoc是通过Emacs的eldoc-mode来实现的。

该模式可以通过你~/.emacs文件里的slime-setup调用来默认开启：

::

   (slime-setup '(slime-autodoc))


* M-x slime-arglist NAME

  显示函数NAME的参数列表

* M-x slime-autodoc-mode

  根据参数的值开启或关闭autodoc-mode。当没有参数时，触发该模式。

如果变量slime-use-autodoc-mode被设置（默认情况），Emacs会启动一个计时器，否则信息只会在按SPC之后显示。

8.8 ASDF
---------

ASDF是一个流行的“系统构建工具”。slime-asdf扩展包提供了一些命令来从Emacs里加载和编译这些系统。ASDF本身没有被包含在Slime里，你必须自己把它加载到Lisp里。还有，你必须在连接之前加载ASDF，否则你会收到关于符号缺失的错误。

* M-x slime-load-system NAME

  编译并加载ASDF系统。默认的系统名字是从当前目录下第一个符合*.asd的文件里获得的。

* M-x slime-open-system NAME &optional LOAD

  打开系统里的所有文件，如果LOAD不是nil的话则加载进来。

* M-x slime-browse-system NAME

  使用Dired浏览系统里的所有文件。

该扩展包也加载了一些新的REPL快捷命令（见 8.2.3 快捷命令）；

* load-system

  编译（根据需要）并加载一个ASDF系统

* compile-system

  编译（但不加载）一个ASDF系统

* force-compile-system

  重新编译（但不加载）一个ASDF系统

* force-load-system

  重新编译并加载一个ASDF系统

* open-system

  打开系统里的所有文件

* browse-system

  使用Dired打开系统里的所有文件

8.9 导航条
---------

slime-banner扩展包在当前REPL缓冲区安装一个位于窗口顶端的横条。开始的时候还会播放一段动画。

通过将slime-startup-animation设置为nil，你可以关闭动画，而slime-header-line-p可以设置横条。

8.10 编辑命令
-----------

slime-editing-commands扩展包提供了一些命令来编辑Lisp表达式。

* C-c M-q 或 M-x slime-reindent-defun

  重新缩进当前的defun，或者重排当前段落。如果光标在一段注释里，那么光标附近的文本会被当做一个段落，然后用fill-paragraph重排。否则，它会被当做Lisp代码，当前defun会被重新缩进。如果当前defun有没匹配的括号，在重新缩进前会尝试修复。

* C-c C-] 或 M-x slime-close-all-parens-in-sexp

  补全当前光标处未闭合的S表达式的括号。插入足够多的右括号，使得跟它的左括号数量匹配。删除多余的左括号，将结尾处的括号格式化为Lisp形式。

  如果REGION是true，对该区域操作。否则对顶层环境光标前的表达式操作。

* M-x slime-insert-balanced-comments

  在包含光标的表达式里插入对称的注释。如果该命令被重复调用（多次调用之间没有其它命令了），注释逐渐从里面的表达式向外扩展。如果调用的时候有前缀参数，S表达式的参数列表会有一个对称的注释。

* M-C-a 或 M-x slime-beginning-of-defun

* M-C-e 或 M-x slime-end-of-defun

8.11 更好的检查器
---------------

有一个默认检查器的替代物，由slime-fancy-inspector扩展包提供。该检查器更加了解CLOS对象和方法。它提供很多用来使Lisp代码检查对象的行为。例如，为了展示一个泛函数，检查器会以纯文本的形式显示其文档，而对于每个方法则会列出它的超链接和一个你可以调用的“除去该方法”行为。它的键绑定跟默认检查器是一样的。

8.12 对象描述
-----------

在Slime里，一个“对象描述” [#f2]_ 指的是跟一个Lisp对象有关的一块文本。右键点击文本会弹出操作该对象的一个菜单。有些操作，例如查看，对所有对象都适用，但对象也可以有自己特有的操作。例如，路径对象有Dired相关的操作。

更加重要的是，可以使用所有标准的Emacs命令来剪切和粘贴这些描述（也就是Lisp对象，而不仅仅是打印出来的样子）。通过这种方式，可以剪切和粘贴REPL里之前计算出来的结果。这对不可读对象来说十分重要。

slime-presentations扩展包在REPL里安装这种对象描述，也就是求值命令的结果会被显示出来。使用这种方法，相关描述会生成标准Common Lisp REPL历史变量的用法。例如：

::

   CL-USER> (find-class 'standard-class)
   #<STANDARD-CLASS STANDARD-CLASS>
   CL-USER>

在缓冲区里描述会以红色显示。使用标准的Emacs命令，描述可以被复制进REPL内的一个新的输入里：

::

   CL-USER> (eql '#<STANDARD-CLASS STANDARD-CLASS> '#<STANDARD-CLASS STANDARD-CLASS>)
   T

当你复制了一个不完整的描述，或者编辑描述里的文本，该描述会变为纯文本，丢失与Lisp对象之间的关联。在缓冲区里，这会通过其颜色从红色变回黑色来表示，而且不能撤销。

对象描述也可以在查看器（所有可以查看的部分都是对象描述）和调试器（所有的本地变量都是对象描述）里使用。这样就可以使用出现在调试窗口里的对象来在REPL里求值。这比使用M-x sldb-eval-in-frame更加方便。警告：从查看器和调试器而来的对象只在相关窗口打开的时候才是可用的。否则的话会引起错误或者混淆。

对于某些Lisp实现，你还可以安装slime-presentation-streams包，它让对象描述适用于*standard-output*流和其它流。这意味着不只是计算的结果，而是某些对象都可以通过与对象描述相关联来打印到标准输出（作为计算的副作用）。目前所有的不可读对象和路径都被作为对象描述打印出来。

::

   CL-USER> (describe (find-class 'standard-object))
   #<STANDARD-CLASS STANDARD-OBJECT> is an instance of
       #<STANDARD-CLASS STANDARD-CLASS>:
     The following slots have :INSTANCE allocation:
       PLIST                   NIL
       FLAGS                   1
       DIRECT-METHODS          ((#<STANDARD-METHOD
                                   SWANK::ALL-SLOTS-FOR-INSPECTOR
                                   (STANDARD-OBJECT T)>

这也使得可以复制粘贴、查看这些对象。

除了标准Emacs命令，还有一些键盘命令，一个menu-bar菜单，一个上下文菜单来操作对象描述。我们在下面解释了这些键盘命令，它们也可以通过menu-bar访问。

* C-c C-v SPC 或 M-x slime-mark-presentation

  如果光标在描述内，将其移到描述的最前并标记其末尾。这样就可以复制该描述。

* C-c C-v w 或 M-x slime-copy-presentation-at-point-to-kill-ring

  如果光标在描述内，将该描述复制到kill ring里。

* C-c C-v r 或 M-x slime-copy-presentation-at-point-to-repl

  如果光标在描述内，将该描述复制到REPL里。

* C-c C-v d 或 M-x slime-describe-presentation-at-point

  如果光标在描述内，显示相关对象的注释。

* C-c C-v i 或 M-x slime-inspect-presentation-at-point

  如果光标在描述内，在Slime查看器里查看该对象。

* C-c C-v n 或 M-x slime-next-presentation

  将光标移到缓冲区里的下一个描述处。

* C-c C-v p 或 M-x slime-previous-presentation

  将光标移到缓冲区里的上一个描述处。

相关的操作也可以在每一个描述的上下文菜单里找到。在一个描述处单击mouse-3打开上下文菜单会，会显示可用的命令。对于某些对象，某些特别的命令也是可用的。用户可以通过给swank::menu-choices-for-presentation定义方法来定义特殊的命令。

警告：对于没有弱哈希表的Lisp实现，所有跟对象描述相关联的对象都被垃圾回收保护起来。如果你的Lisp进程因此变得太大，使用C-c C-v M-o（slime-clear-presentations）断开这些关联，这会清空REPL缓冲区，并且断开所有对象描述的关联。

警告：对象描述可能让新用户迷惑。

::

   CL-USER> (cons 1 2)
   (1 . 2)
   CL-USER> (eq '(1 . 2) '(1 . 2))
   T

可能有人会期望结果是nil，因为这看起来像是两个新创建的cons在相互比较，而忽视了它们的对象身份。但是在上例中，对象描述(1 . 2)是被两次复制到REPL里的，所以eq确实是作用在相同的对象上的，也就是之前输入到REPL里的cons对象。

8.13 打印窗口
-----------

打印窗口是一个特殊的Emacs窗口，用来代替显示区域（mini缓冲区）来显示Slime命令的信息。这是一个可选的特性。跟显示区域相比，打印窗口的优势是可以显示更多的文本，可以被滚动，而且当你按键时内容不会消失。所有可能的较长的信息都会被发送到打印窗口，例如参数列表、宏展开等等。

* M-x slime-ensure-typeout-frame

  保证打印窗口存在，如果需要就新建一个。

如果打印窗口关闭那么会重新使用显示区域。

如果要在启动时自动创建一个打印窗口，需要加载slime-typeout-frame扩展包。（见 8.1 加载扩展包）

slime-typeout-frame-properties变量指定了打印窗口的长度和其它可能的特性。它的值会传给make-frame。

8.14 TRAMP
---------

slime-tramp扩展包提供了一些为TRAMP进行文件名转换的函数。（见 7.1.3 设置路径名翻译）

8.15 文档链接
-----------

对于某些错误信息，SBCL包含了ANSI标准或者SBCL用户手册相关的参考。slime-references扩展包将这些参考变为可以点击的链接。这使得在HyperSpec里找到这些参考相关的章节更加容易。

8.16 交叉引用和类查看器
-------------------

slime-xref-browser扩展包提供了一个基础的类查看器。

* M-x slime-browse-classes

  该命令需要一个类的名字，它会显示出类的所有继承关系。

* M-x slime-browse-xrefs

  该命令显示一个符号及其交叉引用，即它的调用者。以该符号为根的引用树会在之后显示出来。

8.17 高亮编辑
----------

slime-highlight-edits是一个用来高亮显示Lisp源代码里被修改了的部分的minor模式。这对于快速找到那些需要重新编译（用C-c C-c）的函数十分有用。

* M-x slime-highlight-edits-mode

  打开或关闭slime-highlight-edits-mode

8.18 空白缓冲区
------------

由slime-scratch扩展包提供的Slime的空白缓冲区，模仿Emacs的*scratch*缓冲区。如果slime-scratch-file被设置，它被用来备份空白缓冲区，使其变得可持久。它跟其它的Lisp缓冲区是一样的，除了绑定到C-j的命令。

* C-j 或 M-x slime-eval-print-last-expression

  对光标前的表达式求值，并将结果插入到当前缓冲区里。

* M-x slime-scratch

  创建一个*slime-scratch*缓冲区。在此缓冲区里你可以输入Lisp表达式并用C-j来求值，类似于Emacs的*scratch*缓冲区。

8.19 slime**sprof
-----------------

slime-sprof扩展包用来集成SBCL的静态分析器，sb-sprof。

slime-sprof-exclude-swank变量控制是否显示swank函数，默认值是nil。

* M-x slime-sprof-start

  开始分析。

* M-x slime-sprof-stop

  停止分析。

* M-x slime-sprof-browser

  报告分析结果

下面的命令在slime-sprof-browser模式里定义：

* RET 或 M-x slime-sprof-browser-toggle

  打开或折叠函数的详细信息（调用者、调用）

* v 或 M-x slime-sprof-browser-view-source

  查看函数源码

* d 或 M-x slime-sprof-browser-disassemble-function

  拆开该函数

* s 或 M-x slime-sprof-toggle-swank-exclusion

  标记swank函数使其不在报告里

8.20 slime**fancy
----------------

slime-fancy包是一个用来加载那些最受欢迎的包的元包。

脚注

.. rubric:: Footnotes


.. [#f1] 这种类型的补全由Chris McConnell在completer.el里被构建。该包跟ILISP绑定在一起。

.. [#f2] 对象描述是来自于Lisp机的特性。可以通过定义present方法来适用于不同的设备，例如将对象绘到位图显示屏或者将文本写到字符流。
