---
layout:     post
title:      QT之toggled()的使用
subtitle:   toggled(),checked
date:       2020-05-08
author:     wukurua
header-img: img/qt/post-bg-qt.png
catalog: true
tags:
    - Qt
---

> 需求:密码输入框中密码暗文显示,按按钮密码明文显示,再一次则又暗文显示

## 一、实现 ##


>LoginWindow.cpp

	//定义QPushButton
    btnPwdShow=new QPushButton("",this);
	//设置控件id名以使用qss选择器
    btnPwdShow->setObjectName("btnPwdShow");
	//设置此按钮为可检查
    btnPwdShow->setCheckable(true);

	//点击按钮切换密码明文暗文显示和按钮图片
	connect(btnPwdShow,SIGNAL(toggled(bool)),this,SLOT(changePwdEchoMode(bool)));

	//槽函数
	void LoginWindow::changePwdEchoMode(bool checked)
	{
	    if(checked)
	    {
	        etPwd->setEchoMode(QLineEdit::Normal);//明文模式
	    }
	    else
	    {
	        etPwd->setEchoMode(QLineEdit::Password);//暗文模式
	    }
	}

>qss

	QPushButton#btnPwdShow{
	    border-image: url(":/img/view_off.png");
	}
	
	QPushButton#btnPwdShow:checked{
	    border-image: url(":/img/view.png");
	}

## 二、解释 ##
Qt相关文档对`toggled`信号的解释:

	[signal] void QAbstractButton::toggled(bool checked)

>This signal is emitted whenever a checkable button changes its state. checked is true if the button is checked, or false if the button is unchecked.
>
>This may be the result of a user action, click() slot activation, or because setChecked() is called.

当可选中的按钮被更改其可选中状态,即`checked`时，就会发出此信号。如果按钮被选中，则`checked`属性为真；如果按钮未被选中，则为假。

按钮被选中从而发出此信号可能是因为用户操作、激活`click()`槽函数或调用`setChecked()`的结果。

官方文档给出了使用此属性的例子:实现一个槽函数,它可以对刚被选中的按钮发出的信号作出反应,但同时在按钮未被选中时,忽略按钮发出的信号：

	void MyWidget::reactToToggle(bool checked)
	{
	 if (checked) {
	    // Examine the new button states.
	    ...
	 }
	}

那么checked是什么呢?文档里这样写到:

`checked : bool`

>This property holds whether the button is checked
>
>Only checkable buttons can be checked. By default, the button is unchecked.

此属性用来表示是否选中按钮.

只能可选中的按钮才能被选中。默认情况下，按钮是未选中状态。

Access functions:

	bool isCheckable() const//查看按钮是否为可选中状态
	
	void setCheckable(bool)//设置按钮是否可选中

所以,**在使用此属性之前必须要使用`setCheckable(true)`设置此按钮为可选中的**.


## 三、toggled()triggered()的区别 ##

- toggle有开关的意思,开关可以有两个状态:合上和断开.于是更准确的译法应该是切换，**在两个状态间进行转换**.在Qt中,checkable按钮或是图标的槽函数应该用toggled()事件来激活,也是这个道理..

- trigger更有触发的意思.这个单词还有另一个意思就是板机.枪械上用来发射子弹的那种.我们很容易想到板机是没有开/关两种状态的,不能说让它一直关上,一直发射子弹.在Qt中,一般的按钮(uncheckable)的激活方式即是triggered().菜单的激活方式就是triggered().