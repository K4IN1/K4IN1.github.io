---
title: Ubuntu下使用terminator或其他终端软件并替换默认终端和右键菜单
date: 2024-07-05 10:12:31
tags:
- Ubuntu
- 终端
categories:
- 教程

---

# Ubuntu下使用terminator或其他终端软件并替换默认终端和右键菜单  

Ubuntu默认所使用的gnome-terminal，虽然简洁好用，但是不能分多个窗口  
我个人在使用过程中会出现输入卡顿，补全卡顿等问题，有时候很烦心  
使用一个外观更加现代，功能更加丰富，性能强大的Terminal，可以提高我们的效率

<!-- more -->

## 请注意区分

Terminal与Shell是不同的

- Terminal是用来显示Shell，与Shell交互的界面  
- Shell则是系统内核Core外层，与Core进行交互的一层  

在GUI环境下理论上不存在真正的Terminal，所以你看到这些TerminalGUI往往都会带有Simulator的字样  
在22.04版本中，需要使用```Ctrl+Alt+F3/4/5/6```来进入不同Ternimal，这个才是Ubuntu真正的终端。  
退出请尝试```Ctrl+Alt+F1/2/7```，或在终端输入用户名密码登陆后尝试```starx```命令  

## 安装你选择的终端软件

例如安装Terminator

```shell
sudo apt install terminator
```

或者从.deb .AppImage 等软件包安装
请确保你知道软件安装的位置

## 替换默认终端的快捷键

使用`Ctrl+Alt+T`就可以快捷打开终端
有的终端软件在安装之后就会自动替换该快捷键
如果没有替换，请使用如下指令

```shell
gsettings set org.gnome.desktop.default-applications.terminal exec /usr/bin/terminator
gsettings set org.gnome.desktop.default-applications.terminal exec-arg "-x"
```

请把`exec`后的路径更换成你自己终端的路径
要还原初始，请使用

```shell
gsettings reset org.gnome.desktop.default-applications.terminal exec
gsettings reset org.gnome.desktop.default-applications.terminal exec-arg
```

也可以通过间接启动的方式  

1.打开Gnome-terminal，转到你的Profile预设，选择Command命令选项卡  
2.勾选运行自定义命令而不是Shell
3.在自定义命令中键入运行你的Termimal 例如`terminator`  

如果需要恢复，请使用`gnome-terminal --preferences`打开选项卡，并取消勾选上述选项

## 替换右键菜单

如果是按照上述第二种方法，此时右键菜单应该已经可用  
选择在此打开Terminal即可打开你的自定义终端  
如果没有生效，可以使用这个nautilus[插件](https://github.com/Stunkymonkey/nautilus-open-any-terminal)  
这个插件会增加一个新的右键菜单，在当前目录下启动你的终端  
该插件同时也支持Caja文件管理器，几乎支持所有的主流终端软件，但是需要自己做一些简单的配置  
建议从源代码安装，安装完成后`nautilus -q`重新启动文件管理器即可  
要配置该拓展，需要使用命令行或者dconf-editor  
通过编辑`com.github.stunkymonkey.nautilus-open-any-terminal`中的四个不同参数即可  

```shell
gsettings set com.github.stunkymonkey.nautilus-open-any-terminal terminal alacritty
gsettings set com.github.stunkymonkey.nautilus-open-any-terminal keybindings '<Ctrl><Alt>t'
gsettings set com.github.stunkymonkey.nautilus-open-any-terminal new-tab true
gsettings set com.github.stunkymonkey.nautilus-open-any-terminal flatpak system
```

建议参考Github原文章的参数  

2024.7.5

15:43  
