## OS层面

- top
- vmstat 1 3
- iostat -d 1 3
- pidstat -r     可以追踪到进程 

## JDK层面

- jps
  - 查看当前系统的java进程信息
  - 显示main函数所在类的名字
  - -m可以显示进程的参数
  - -l显示程序的全路径
  - -v显示传递给Java的main函数参数
- jstat -gc pid
  - 查看JVM堆信息
- jinfo -flags pid
  - 查看虚拟机参数
  - jinfo -flag  [+/-] \<name> 设置某些参数（布尔值）
  - jinfo -flag \<name>=\<value> 设置某些参数

- jstack -l \<vmid>
  - 查看线程拥有的锁，分析线程死锁的原因

- jstatd 

  ![image-20210226223858145](https://img.jooks.cn/img/20210226223858.png)

![image-20210226223943926](https://img.jooks.cn/img/20210226223943.png)

- jcmd

![image-20210226224115813](https://img.jooks.cn/img/20210226224115.png)

## JMX

![image-20210226225817866](https://img.jooks.cn/img/20210226225817.png)

![image-20210226225940229](https://img.jooks.cn/img/20210226225940.png)

![image-20210226230125572](https://img.jooks.cn/img/20210226230125.png)

