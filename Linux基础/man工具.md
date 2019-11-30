Linux有很多常用命令，每个命令还有其相关的参数。咱们都是人，记忆能力有限，对于命令的记忆能力也非常有限，不能全部记住，如果谁能记住全部命令，那可真是神人了。在这种情况下，man工具便发挥了其作用。

man命令语法 ： man [-fnaw] command

-f 显示command的所有手册

-n 显示指定章节手册

-a 显示所有章节手册

-w 显示手册所在路径

在运维的口中有句话“有问题找男人（man）”，咱们现在可以找“男人”试一下：

假设我们不知道vi编辑器怎么用。则可以使用以下代码查看手册：

```
bash# man vi
VIM(1)                      General Commands Manual                     VIM(1)

NAME
       vim - Vi IMproved, a programmers text editor

SYNOPSIS
       vim [options] [file ..]
       vim [options] -
       vim [options] -t tag
       vim [options] -q [errorfile]

       ex gex
       view
       gvim gview vimx evim eview
       rvim rview rgvim rgview

DESCRIPTION
       Vim  is a text editor that is upwards compatible to Vi.  It can be used
       to edit all kinds of plain text.  It is especially useful  for  editing
       programs.

       There  are a lot of enhancements above Vi: multi level undo, multi win‐
       dows and buffers, syntax highlighting, command line  editing,  filename

```

有童鞋会说，为啥我安装的系统是中文版的，但是man还是英文。同学，你醒醒，Linux是老外做的，man也是老外写的，如果看不懂，咱们先补补英语吧（虽然博主的英语也很差，但是读这东西够了）。



