---
layout:     post
title:      "Activity中需要注意的知识点"

date:       2017-2-12 21:40:41
author:     "Wengdada"
header-img: "img/post-bg-sublime3-markdown.jpg"
catalog: true
tags:  
    - Android
    - Activity
---




>本文主要是讲一些Activity中需要注意的知识点，如果对Activity不熟悉的可以参考下以下这篇文章，**[ Activity你真的熟悉吗？看了才知道 ](http://www.jianshu.com/p/c21216bf5f82)**,下面开始正文。

1. 典型情况下的生命周期：由用户参与情况下Activity所经过的生命周期的改变；

2. 异常情况下的生命周期：Activity被系统回收或者设备的Configuration发生改变从而导致Activity被销毁重建；

3. 用户打开新的Activity或者切换到桌面的时候，回调如下：onPause->onStop，但是若Activity采用的是<font color="#F27F05" >透明的主题</font>，则当前的Activity不会回调onStop；

4. onPause表示Activity正在停止，正常情况下onStop会被调用，此时若快速回到当前的Activity，则onResume会被调用。但是这种情况属于极端情况，用户很难重现这一场景。

5. onStart,onStop是从Activity是否可见的角度来回调的，而onResume，onPause则是从Activity是否位于前台这个角度来进行回调的，除了这种区别，在实际的使用中并没有其他明显的区别。
6. 启动Activity的源码相当复杂，可以简单理解如下，启动Activity的请求会由Instrumentation来处理，然后它通过Binder相ActivityManagerService发请求，ActivityManagerService内部维护着一个Activitystack并负责栈内的Activity的状态的同步，ActivityManagerService通过ActivityThread去同步Activity的状态从而完成Activity生命周期方法的调用。

7. 新的Activity启动之前需要调用栈顶Activity的onPause后新的Activity才能启动。所以不能在Activity中做重量级的操作，<font color="#F27F05" >应当尽量在onStop中做操作使得新的Activity尽快显示出来并切换到前台</font>。

8. Activity在异常情况下终止时系统会回调onsaveinstancestate来保存当前Activity的状态。这个调用方法是在onStop之前但是和onPause没有既定的时序关系，可能在onPause之前或之后调用。当Activity重新创建的时候会在onStart之后调用onRestoreInstanceState,利用保存的bundle对象来取出保存的数据并恢复，当然也可以利用onCreate方法中的bundle对象来恢复。<font color="#F27F05" >官方建议采用onRestoreInstanceState来恢复数据</font>，因为该方法被调用，其参数中的bundle对象一定是有值的，然而onCreate是正常情况下调用的话，其参数中的bundle对象是空的，需要额外的判断。

9. 在onSaveInstanceState和onRestoreInstanceState方法中，系统默认做了一定的恢复工作，默认保存Activity的试图结构，并在Acitivity重启后恢复这些数据，比如文本框中的数据，ListView滚动的位置等，每个View都有onSaveInstanceState和onRestoreInstanceState这两个方法。关于保存和恢复View层次结构，系统的工作流程是这样的，当Activity发生意外终止时，Activity会调用OnSaveIntanceState去保存数据，然后Activity会委托Windows去保存数据，接着Windows再委托它上面的顶级容器去保存数据。顶层的容器是一个ViewGroup，一般来说时DecorView。最后再有顶层的容器一一通知子元素来保存数据，这样这个数据保存就完成了。这就是安卓中的委托机制，View的绘制和事件的分发都是用的委托机制。

10. Activity按照优先级从高到低，可分为如下三种：前台Activity——正在和用户交互的Activity，优先级最高；可见但非前台Activity——比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法与用户直接交互；后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低。当系统内存不足时，系统就会按照上述优先级去杀死目标Activity所在的进程，并在后续通过onSaveInstanceState和onRestoreInstanceState来存储和恢复数据。如果一个进程中没有四大组件在执行，这个进程很容易被杀死，较好的方法是将后台工作放入Service中保证有一定的优先级，不容易被系统杀死。如果当某项内容发生改变后，不想Activity重新创建，就可以给Activity指定configChanges属性，多个值用“|”连接起来。android:configChanges="orientation | keyboardHidden"。

11. singleTask：栈内复用模式。只要Activity中一个栈中存在，多次启动Activity都不会重新创建实例，和singleTop一样也会回调onNewInent。具体一点，当一个具有singleTask模式的Activity A请求启动后，系统先会寻找是否存在A想要的任务栈。如果不存在对应任务栈，就重新创建一个任务栈，然后创建A的实例后，把A放到任务栈中。如果存在A所需的任务栈，那么系统将判断该任务栈中是否有实例A。如果有实例A，那么系统就将A调到栈顶并调用其onNewIntent方法（会清空A之上的Activity）。如果没有实例A，那么系统就创建实例A并压入栈中。
sdjkfjdskf


12. sdfdsf

<meta http-equiv="refresh" content="10">