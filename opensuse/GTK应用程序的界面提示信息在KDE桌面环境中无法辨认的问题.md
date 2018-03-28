GTK应用程序的界面提示信息在KDE桌面环境中无法辨认的问题
============================================================

有些GTK应用程序界面上出现的提示信息框里的内容是白色文字显示在非常淡的灰色背景上，无法辨认，解决方法如下：
	
	修改~/.gitrc-<version>文件，在其结尾添加下面代码，
	
	style "color-chooser-tooltips"
	{
	bg[NORMAL] = "#FFFFAF"
	fg[NORMAL] = "#000000"
	}
	widget "gtk-tooltip*" style "gnome-color-chooser-tooltips"
	
