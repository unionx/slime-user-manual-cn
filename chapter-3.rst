三、使用Slime模式
==============

Slime的所有命令都通过slime-mode提供。它是一个与Emacs的lisp-mode配合使用的minor-mode。本章描述slime-mode及其相关事项。

3.1 用户界面须知
--------------

要方便地使用Slime，了解一些“全局的”用户界面特性是十分重要的。这一部分描述了最为重要的原则。

3.1.1 临时缓冲区
^^^^^^^^^^^^^^

某些Slime命令会创建临时缓冲区来显示结果。虽然这些缓冲区有它们自己的为了特定目的而使用的major-mode，但某些特定的约定是通用的。

可以通过按q键来关闭临时缓冲区。此操作会关闭缓冲区并且回复缓冲区显示之前的窗口配置。临时缓冲区也可以通过一般的命令例如kill-buffer来关闭，这样的话之前的窗口配置就不会被恢复。

按RET键被认为是“做最明显的有用的事情”。例如，在apropos缓冲区此操作会打印出当前光标处的符号的详细描述，而在XREF缓冲区此操作会显示当前光标处的索引的源代码。这样的行为是效仿Emacs的显示相关内容、补全结果等的缓冲区。

含有Lisp符号的临时缓冲区会使用slime-mode来补充它自己的特定的模式。这样就可以使用一般的Slime命令，例如描述符号、查找函数定义等等。

对这些“描述性的”缓冲区的初始聚焦由变量slime-description-autofocus确定。如果它是nil（默认的），这些描述性缓冲区不会自动聚焦，反之则会。

3.1.2 *inferior-lisp*缓冲区
^^^^^^^^^^^^^^^^^^^^^^^^^^

Slime在内部使用comint包来启动Lisp进程。这会产生一些用户可见的结果，有些是好的，另一些则不是。为了避免产生疑惑，理解其交互特性是很有用的。

*inferior-lisp*缓冲区包含有Lisp进程自己的top-level。这个与Lisp的直接连接对错误排查很有用，并且使用inferior-slime-mode可以达到某种程度上的Slime集成。许多人选择加载更好的集成模块SLIME REPL包（见8.2 REPL）而无视*inferior-lisp*缓冲区。（见8.1 加载扩展包 获得更多关于启动REPL的信息。）

3.1.3 多线程
^^^^^^^^^^^

如果Lisp支持多线程，对于每个请求Slime会生成一个新的线程，例如，C-x C-e会创建一个新线程来对表达式求值。但是对于从REPL来的请求则是一个例外：所有在REPL缓冲区里输入的命令都会在一个专用的REPL线程里求值。

多线程和特殊变量会导致一些复杂性。非全局的特殊绑定是在本地线程里的，也就是说，在一个线程里改变一个由let绑定的特殊变量的值不会影响到其它线程里相同名字的变量的值。这增加了改变新线程的打印和读取行为的困难程度。变量swank:*default-worker-thread-bindings*就是为了应付这种情况的：不需要改变一个变量的全局的值，而是增加swank:*default-worker-thread-bindings*的绑定，例如，使用下面的代码，新的线程会默认将浮点值读取为double。

::

   (push '(*read-default-float-format* . double-float)
         swank:*default-worker-thread-bindings*)


3.1.4 键绑定
^^^^^^^^^^

总体上我们会让我们的键绑定跟Emacs的键绑定配合良好。我们也使用了我们自己的某种不太寻常的约定：当键入一个三次按键的命令时，最后一次按键可以按Control也可以不按。例如，slime-describe-symbol命令的键绑定是C-c C-d d，但是按C-c C-d C-d也是同样的。我们将两种方式都绑定了，因为有些人喜欢三次按键都按着Control键，而有些人则不是。并且有两次按键作为前缀，我们不怕键不够用。

这条规则只有一个例外，希望不要让你中招。我们从来不在任何命令里绑定C-h键，所以C-c C-d C-h跟C-c C-d h做的事情是不一样的。这是因为Emacs内建的默认情况是，输入一个前缀，然后按C-h，会显示处所有以该前缀开始的键绑定。所以C-c C-d C-h命令实际上会显示出所有的文档命令。这个特性太有用了所以我们不会替换它！

“你是故意破坏Emacs超级牛逼的在线帮助机制吗？上帝都会震怒的！”

此建议十分有用。Emacs的在线帮助机制是你最快捷、最完整和最新的关于键绑定的信息来源。它们是你的朋友：

* C-h k <key> 或者 M-x describe-key <key>

  描述当前缓冲区的绑定到<key>的函数。

* C-h b 或 describe-bindings

  列出当前缓冲区的所有键绑定。

* C-h m 或 describe-mode

  显示所有当前缓冲区可用的major-mode的命令，然后是所有的minor-mode的命令。

* C-h l 或 view-lossage

  按顺序显示出你刚刚按了的所有按键的序列。

注意：在本文档里C-h指定的意思是一个“典型键绑定”（canonical key），它可能是Ctrl-h或者是F1。或者是普通情况下你的.emacs文件里配置的help-command函数的绑定。下面是一种情形：

::

   (global-set-key [f1]   'help-command)
   (global-set-key "\C-h" 'delete-backward-char)

在这种情况下，在本文档中任何地方你看到的C-h你都可以用F1来代替。

你可以像这样用global-set-key函数在你的~/.emacs文件里全局地更改默认的键绑定：

::

   (global-set-key "\C-c s" 'slime-selector)

这会绑定slime-select函数到C-c s。

或者，如果你只是想在特定的slime模式下新建或改变键绑定，你可以像这样在你的~/.emacs文件里使用define-key函数：

::

   (define-key slime-repl-mode-map (kbd "C-c ;") 'slime-insert-balanced-comments)


这会在REPL缓冲区里绑定slime-insert-balanced-comments函数到C-c ;键绑定。

3.2 求值命令
----------

这些命令每一个都以不同的方式来对一个Common Lisp表达式求值。一般来说它们模仿Emacs Lisp的求值命令。默认情况下它们会在显示区显示出结果，但是一个前缀参数会让结果插入到当前缓冲区中。

* C-x C-e 或 M-x M-x slime-eval-last-expression

  对光标前的表达式求值并且将结果显示到显示区

* C-M-x 或 M-x slime-eval-defun
  
  对当前toplevel的形式进行求值并将结果打印到显示区。“C-M-x”会特别对待“defvar”。正常来讲，如果定义的变量已经有一个值了，“defvar”表达式不会做任何事情。但是“C-M-x”命令无条件的将“defvar”表达式里定义的值初始化并赋予指定的值。这个特性十分便于调试Lisp程序。

如果带数字参数地执行C-M-x或者C-x C-e，它会将结果插入到当前缓冲区，而不是将其打印到显示区。

* C-c : 或 M-x slime-interactive-eval

  从迷你缓冲区读取一个表达式并求值

* C-c C-r 或 M-x slime-eval-region

  对区域进行求值

* C-c C-p 或 M-x slime-pprint-eval-last-expression

  对光标前的表达式进行求值并将结果漂亮地打印在一个新的缓冲区里

* C-c E 或 M-x slime-edit-value

  在一个叫做“Edit <form>”的新缓冲区里编辑一个可以setf的形式的值。这个值会被插入一个临时缓冲区以便编辑，然后用C-c C-c命令来提交设置于Lisp中。

* C-x M-e 或 M-x slime-eval-last-expression-display-output
  
  对光标前的表达式求值并将结果打印在显示缓冲区里。如果表达式会写一些内容到输出流的话这会很有用。

* C-c C-u 或 M-x slime-undefine-function

  用fmakunbound来取消当前光标处函数的定义。

3.3 编译命令
----------

Slime有许多很好的命令来编译函数、文件和包。好的地方在于，很多Lisp编译器生成的提示和警告会被拦截，然后直接注释给Lisp源文件缓冲区里相应的表达式。（试一试看会发生什么。）

* C-c C-c 或 M-x slime-compile-defun

  编译光标处的top-level形式。被选择的区域会闪一下以给出回应，表明是哪一部分被选择了。若给了一个（正的）前缀参数的时候，形式会以最小调试设置来编译。若是一个负的前缀参数，编译速度会被优化。区域里的代码在编译之后将要被执行，总的来说，此命令将该区域写入一个文件，编译该文件，然后加载结果代码。

* C-c C-k 或 M-x slime-compile-and-load-file
  
  编译和加载当前缓冲区的源文件。如果编译步骤失败了，那么文件不会被加载。编译是否失败并不总是那么容易判断的：某些情况下你可能会在加载阶段进入调试器。

* C-c M-k 或 M-x slime-compile-file

  编译（但不加载）当前缓冲区的源文件。

* C-c C-l 或 M-x slime-load-file

  加载Lisp文件。此命令用到了Common Lisp的LOAD函数。

* M-x slime-compile-region

  编译选中的区域。

Slime通过在源代码的形式下加下划线来表示有提示信息。可以通过将鼠标置于文本处或者下面这些选择命令来阅读带有提示信息的编译器消息。

* M-n 或 M-x slime-next-note

  将光标移到下一个编译器消息处并显示消息。

* M-p 或 M-x slime-previous-note

  将光标移到上一个编译器消息处并显示消息。

* C-c M-c 或 M-x slime-remove-notes

  删除缓冲区里的所有提示信息。

* C-x ‘ 或 M-x next-error

  访问下一个错误消息。实际上这不是一个Slime命令，Slime会创建一个隐藏的缓冲区，然后大部分的编译模式的命令（见info “emacs”文件的“Compilation Mode”节点）都会类似批处理编译器一样地编译Lisp。

3.4 补全命令
----------

补全命令的作用是根据光标处已有的东西来补全一个符号或者形式。典型的补全假设一个确定的前缀，给出的选择也只是可能发生的分支。模糊补全会做更多的尝试。

* M-TAB 或 M-x slime-complete-symbol

  补全光标处的符号。注意，Slime里有三种模式的补全；默认的模式跟正常的Emacs补全类似（见6.1 slime-complete-symbol-function）

3.5 查找定义（“Meta-Point”命令）
----------------------------

Slime提供了熟悉的M-.命令。对于广泛函数来讲此命令会找出所有的方法，而在某些系统上它会做一切其它事情（例如根据DEFSTRUCT定义来追踪结构访问器）。

* M-. 或 M-x slime-edit-definition
  
  跳至光标处符号的定义处

* M-, 或 M-* 或 M-x slime-pop-find-definition-stack

  回到M-.命令执行的光标处。如果M-.被执行了多次，那么此命令会多重地回溯。

* C-x 4 . 或 M-x slime-edit-definition-other-window

  类似slime-edit-definition，但是会跳到另一个窗口来编辑其定义。

* C-x 5 . 或 M-x slime-edit-definition-other-frame
  
  类似slime-edit-definition，但是会跳到另一个框架来编辑其定义。

* M-x slime-edit-definition-with-etags

  使用ETAGES的表来寻找当前光标处的定义。

3.6 文档命令
----------

Slime的在线文档命令效仿了Emacs的例子。这些命令都以C-c C-d为前缀，并且允许更改其键绑定或者取消更改（见 3.1.4 键绑定）

* SPC 或 M-x slime-space
  
  Space键插入一个空格键，并且也查找并显示出当前光标处函数的参数列表，如果有的话。

* C-c C-d d 或 M-x slime-describe-symbol

  描述当前光标处的符号。

* C-c C-d f 或 M-x slime-describe-function

  描述当前光标处的函数。

* C-c C-d a 或 M-x slime-apropos

  对于一个正则表达式执行一个合适的搜索，来搜索所有的Lisp符号名称，并且显示出相应的文档字符串。默认情况下所有包的外部变量都会被搜索，你可以用一个前缀参数来指定特定的包或者是否包含未导出的符号。

* C-c C-d z 或 M-x slime-apropos-all

  类似slime-apropos但是默认包含所有内部符号。

* C-c C-d p 或 M-x slime-apropos-package

  显示包内所有符号的合适的结果。这个命令是用来在一个较高层次浏览包的。加上包名补全，它可以差不多被当作是一个Smalltalk类似的图像浏览器。

* C-c C-d h 或 M-x slime-hyperspec-lookup

  在《Common Lisp Hyperspec》里查找当前光标处的符号。它使用常用的hyperspec.el来在浏览器里显示相应的部分。Hyperspec可以在网络上或者在common-lisp-hyperspec-root处，默认打开的浏览器通过browse-url-browser-function指定。

  注意：这里就是一个C-c C-d h跟C-c C-d C-h不同的例子

* C-c C-d ~ 或 M-x hyperspec-lookup-format

  在《Common Lisp Hyperspec》里查找一个foramt格式控制符。

* C-c C-d # 或 M-x hyperspec-lookup-reader-macro

  在《Common Lisp Hyperspec》里查找一个读取宏。

3.7 交叉引用命令
--------------

Slime的交叉引用命令是基于Lisp系统的支持的，而此特性在不同Lisp实现上差异颇大。对于那些没有内置XREF支持的Lisp系统，Slime需要一个可移植的XREF包，这个包是从CMU AI Repository获得并且与Slime绑定在一起的。

所以这些命令的操作对象都是当前光标处的符号，如果没有，则会要求用户输入。如果有前缀参数，则总会要求用户输入符号。你可以输入以下所示的键绑定，或者将最后一个键加上control。见 3.1.4 键绑定。

* C-c C-w c 或 M-x slime-who-calls

  显示该函数的调用者。

* C-c C-w c 或 M-x slime-who-calls

  显示该函数调用了的函数。

* C-c C-w r 或 M-x slime-who-references

  显示对全局变量的引用。

* C-c C-w b 或 M-x slime-who-binds

  显示对全局标量的绑定。

* C-c C-w s 或 M-x slime-who-sets

  显示对全局标量的赋值。

* C-c C-w m 或 M-x slime-who-macroexpands

  显示某个宏扩展之后的结果。

* M-x slime-who-specializes

  显示一个类所有已知的方法。

当然也有所谓的“列出调用者/被调用者”命令。这些操作会在一个很底层的层次上搜寻堆上的函数对象，来确定所有调用的情况。只有某些Lisp系统有此功能，并且在无法获得精确的XREF信息时，这些功能可以作为备用。

* C-c < 或 M-x slime-list-callers

  列出一个函数的所有调用者。

* C-c > 或 M-x slime-list-callees

  列出一个函数所有调用的函数。

3.7.1 XREF缓冲区命令
^^^^^^^^^^^^^^^^^^

XREF缓冲区可用的命令。

* RET 或 M-x slime-show-xref

  在另一个窗口里显示当前光标处的符号的定义。不离开XREF缓冲区。

* Space 或 M-x slime-goto-xref

  在另一个窗口里显示当前光标处的符号的定义并且关闭XREF缓冲区。

* C-c C-c 或 M-x slime-recompile-xref

  重新编译当前光标处的定义。

* C-c C-c 或 M-x slime-recompile-all-xrefs

  重新编译所有定义。

3.8 宏扩展命令
------------

* C-c C-m 或 M-x slime-macroexpand-1

  将光标处的表达式宏展开一次。如果带有一个前缀参数，则使用macroexpand代替macroexpand-1。

* C-c M-m 或 M-x slime-macroexpand-all

  将光标处的表达式完全宏展开。

* M-x slime-compiler-macroexpand-1

  显示光标处的编译宏展开的sexp。

* M-x slime-compiler-macroexpand

  反复展开光标处的编译宏的sexp。

更多的minor-mode命令及相关讨论见5.2 slime-macroexpansion-minor-mode。

3.9 分解命令
----------

* C-c M-d 或 M-x slime-disassemble-symbol

  分解光标处的函数定义。

* C-c C-t 或 M-x slime-toggle-trace-fdefinition

  触发对光标处函数的跟踪。若有前缀参数，则读取附加信息，例如跟踪某个指定的方法。

* M-x slime-untrace-all

  停止跟踪所有函数。

3.10 中止/恢复命令
---------------

* C-c C-b 或 M-x slime-interrupt   

  中断Lisp进程（发送SIGINT）。

* M-x slime-restart-inferior-lisp

  重启inferior-lisp进程。

* C-c ~ 或 M-x slime-sync-package-and-default-directory

  从Emacs到Lisp同步当前包到工作目录。

* C-c M-p 或 M-x slime-repl-set-package

  设置REPL的包。

* M-x slime-cd

  设置Lisp进程所在的当前目录。这也改变了REPL的当前目录。

* M-x slime-pwd

  打印出Lisp进程的当前目录。

3.11 检查命令
-----------

Slime查看器是一个基于Emacs的对普通INSPECT函数的替代选择。Slime查看器会在一个Emacs缓冲区中结合使用文本和对其它对象的超链接来展示一个对象。

Slime查看器可以轻易地为你的程序里的对象做定制化。详细用法见swank-backend.lisp里的inspect-for-emacs广泛函数。

* C-c I 或 M-x slime-inspect

  查看输入在一个迷你缓冲区里的表达式的值。

在查看器里可以使用的标准命令有：

* RET 或 M-x slime-inspector-operate-on-point

  如果光标处是一个值，那么对这个值递归地调用查看器查看它。如果光标处是一个命令，则调用它。

* d 或 M-x slime-inspector-describe

  描述光标处的槽。

* v 或 M-x slime-inspector-toggle-verbose

  在冗余模式和简洁模式之间切换。默认值由swank:*inspector-verbose*指定。

* l 或 M-x slime-inspector-pop

  回到前一个对象（从RET返回）。

* n 或 M-x slime-inspector-next

  l的逆操作。也绑定到了空格键。

* g 或 M-x slime-inspector-reinspect

  再次查看。

* q 或 M-x slime-inspector-quit

  关闭查看缓冲区。

* p 或 M-x slime-inspector-pprint

  在另一个缓冲区里打印出光标处的对象。

* . 或 M-x slime-inspector-show-source

  查看光标处的对象的源码。

* > 或 M-x slime-inspector-fetch-all

  取得所有查看器的内容并且移到其最后。

* M-RET 或 M-x slime-inspector-copy-down

  将光标之后所有在“*”变量里的值存储起来。这些对象可以之后在REPL中访问。

* TAB, M-x slime-inspector-next-inspectable-object 或 S-TAB, M-x slime-inspector-previous-inspectable-object

  分别是跳至下一个或者前一个可查看对象。
  
3.12 分析命令
-----------

所有的分析命令都是给予CMUCL的分析器。它们都是对函数的简单包装，然后打印一些信息到输出缓冲区。

* M-x slime-toggle-profile-fdefinition   

  触发对一个函数的分析。

* M-x slime-profile-package

  分析一个包里的函数。

* M-x slime-profile-by-substring

  分析所有名字含有某个子串的函数。

* M-x slime-unprofile-all

  停止所有分析。

* M-x slime-profile-report

  报告分析数据。

* M-x slime-profile-reset

  重置分析数据。

* M-x slime-profiled-functions

  显示当前所有正在分析的函数。

3.13 遮盖命令
-----------

* C-c C-a, M-x slime-nop 和 C-c C-v, M-x slime-nop

  此键绑定由inf-lisp遮盖。

3.14 语义缩进
-----------

Slime会自动地决定如何缩进你的Lisp程序里的宏。为了达到这个目的，Lisp端会查看系统内所有的宏并且将有&body参数的宏报告给Emacs。然后Emacs会对这些宏的缩进特殊处理，通常情况是，将其第一个参数缩进四个空格，而body参数缩进两个空格。

这仅仅“够用”。如果你是那种很幸运的人，那么你要阅读本节剩下的内容。

为了简化实现，Slime并不区分宏和其他简单的符号名，除了不同的包名。这使得Slime和Emacs自己的缩进方式兼容得很好。但是，如果你有些宏和某些简单符号有相同的名字，那么它们的缩进会相同，使用它们的参数列表里任一缩进方式。你可以找出有哪些符号有缩进冲突：

::

   (swank:print-indentation-lossage)


如果有冲突让你很恼火，不要崩溃，只要用你喜欢的缩进方式覆盖elisp符号的common-lisp-indent-function属性就可以了。Slime不会覆盖你的定制变量，它只是尝试给你最好的默认设置。

更加巧妙的是，有个不那么完美的缓存机制来保证良好的性能。

理想情况下，在每次Emacs操作之后，Lisp会自动查看所有符号的缩进的改变情况。但是若要每次都执行，那么效率就太低了。所以，Lisp通常只会查看那些Emacs使用到的属于本地包的符号，而所有的请求都是从它们那里来的。这使得取得在交互环境下定义的宏的缩进变得十分高效。为了查看剩下的那些，当有新的Lisp包被创建的时候——例如新系统被加载，所有的符号都会被查看。

你可以使用M-x slime-update-indentation来强制要求所有符号的缩进信息都被查看。

3.15 根据读取器的结果字符化
----------------------

Slime会自动对读取器条件判断表达式进行求值，例如#+linux，在源代码缓冲区里，所有会被当前Lisp连接所忽略掉的代码都会呈现为灰色以示不被读取。

