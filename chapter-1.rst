一、简介
========

SLIME 即“Emacs的高级交互式Lisp开发模式”。

SLIME扩展了Emacs，使其可以交互式地编辑Common Lisp。所有的特性都基于slime-mode这个Emacs的minor-mode，它补充了标准lisp-mode。lisp-mode支持编辑Lisp源代码，而slime-mode则提供了与Lisp进程进行交互的功能，包括编译、调试、查找文档等等。

slime-mode开发环境效仿Emacs原生的Emacs Lisp环境。我们也借鉴了某些类似的系统（例如ILISP），当然还包括我们自己的想法。

SLIME由两部分组成：用Emacs Lisp写的用户界面，和用Common Lisp写的服务器端。这两部分通过套接字连接在一起，并且使用一个类似于RPC的协议通信。

服务器端使用的Lisp主要是可移植Commom Lisp。所有跟特定Lisp实现相关的特性都由一个接口定义好，然后由不同的Lisp实现提供。这使得SLIME非常容易移植。
