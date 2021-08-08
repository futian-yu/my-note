##### **Linux查看服务器信息常用命令：**

1.查看cpu的使用率：

```
[root@li676-235 ~]# top -bn 1 -i -c
top - 14:19:51 up 138 days, 7:15, 1 user, load average: 0.20, 0.33, 0.39
Tasks: 115 total, 1 running, 114 sleeping, 0 stopped, 0 zombie
Cpu(s): 4.5%us, 3.8%sy, 0.0%ni, 91.0%id, 0.6%wa, 0.0%hi, 0.0%si, 0.0%st
Mem: 1014660k total, 880512k used, 134148k free, 264904k buffers
Swap: 262140k total, 34788k used, 227352k free, 217144k cached
PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND 
12760 root 20 0 15084 1944 1632 R 2.0 0.2 0:00.01 top -bn 1 -i -c
```

![image-20210203173102157](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20210203173102157.png)

- top命令可以看到总体的系统运行状态和cpu的使用率 ，

- %us：表示用户空间程序的cpu使用率（没有通过nice调度）。

- top命令下面查出的进程的CPU百分比可能会超过100%，这说明该进程占用了多个cpu，多核同时处理。



2. 查看CPU信息（核数，一般我们说的几个就是表示有几个cpu）

```
（1）cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l 单纯查看核数
（2）cat /proc/cpuinfo |grep "model name" && cat /proc/cpuinfo |grep "physical id"	查看cpu型号和核数（cpu个数）
（3）cat /proc/cpuinfo 查看CPU信息（较为详细的）  
	cat /proc/cpuinfo| grep "cpu cores"| uniq	查看每个物理CPU中core的个数(即核数)
```

![image-20210203211228057](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20210203211228057.png)



3.查看内存信息

```
cat /proc/meminfo
```

![image-20210203210356422](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20210203210356422.png)







