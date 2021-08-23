**linux命令**

```shell
cd /   #1.返回根目录
cd ..  #2.返回上一级目录
pwd	   #3.显示当前目录
mkdir  #4.在当前文件夹下创建一个文件
vi a.txt  #5.创建一个名为a的文本文档
cat a.txt #6.查看a.txt的内容
ps -ef #显示当前所有进程环境变量及进程间的关系
ps -ef | grep java #查找指定进程java（grep是强大的文本搜索命令）
kill -9 8001	#杀死进程号PID为8001的进程
esc+:+wq #保存并退出,若想不保存推出,esc + : + q!
cp a.txt b.txt  #在当前文件夹下将a.txt复制一份并重命名为b.txt
mv a.txt b.txt	#将a.txt重命名为b.txt



##===================快捷键
Ctrl+c #在命令行下起着终止当前执行程序的作用
Ctrl+d #相当于exit命令，退出当前shell
Ctrl+s #挂起当前shell
Ctrl+q #解冻挂起的shell再不行就重新连接打开一个终端，reboot linux 或 kill 相关进程。
host键+c #虚拟机切换为窗口自适应模式(host键为右边的ctrl键)





```

