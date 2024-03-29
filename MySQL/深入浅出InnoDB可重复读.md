> 几个前提知识： 
>
> SQL标准的事务隔离级别包括：读未提交、读提交、可重复读、串行化。
>
> MySQL的事务是在引擎层实现的，而原生的MyISAM引擎不支持事务。

可重复读是指：一个事务进行过程中**读取**到的事务跟事务启动时看到的是一致的。（当然，当前事务执行的修改是可以读到的）

## 可重复读事务的实现逻辑

对于可重复读事务隔离级别，InnoDB会在事务启动时生成一个视图(一致性视图)，这个视图是整个库的快照。之后在事务运行过程中，在不加锁的情况下，读取的都是这个视图。最后再提交或是回滚。（这里需要注意的是，“读未提交”隔离级别下直接返回记录上的最新值，没有视图概念）

实现逻辑就是这么简单～

## 视图的实现逻辑

InnoDB有个undo log，会在更新语句执行后立刻记下当前操作，所以我们可以根据当前数据和undo  log就能回推出该事务对应的视图。

假设一个值从 1 被按顺序改成了 2、3、4，在回滚日志里面就会有类似下面的记录。

![image-20210225214245596](https://img.jooks.cn/img/20210225214245.png)

同一条记录在系统中可以存在多个版本，就是数据库的多版本并发控制（MVCC）。对于 read-view A，要得到 1，就必须将当前值依次执行图中所有的回滚操作得到。

## 数据更新实现细节

要明白整个事务的实现原理，或者是视图的实现原理，首先得对InnoDB的数据更新的细节有一定了解。

InnoDB里面每个事务有唯一的事务ID，叫做transaction id，是按申请顺序严格递增的。

每行的数据可以理解为有多个版本。每次事务更新数据的时候，都会生成一个新的数据版本，并且把当前事务的ID赋值到这个数据版本上，记为`row trx_id`。

怎么得到旧版本的数据？就是通过之前的undo log，后面会详细说。

## 视图的实现细节

在实现上，InnoDB为每个事务构造了一个数组，用来保存这个事务启动瞬间当前还未结束的事务的ID。

数组中，事务ID的最小值记为低水位，当前已经创建过的事务ID的最大值+1记为高水位。这个视图数组和高水位，就组成了当前事务的**一致性视图**。

我们根据低水位、高水位、数组就能知道事务启动的瞬间有哪些事务已经提交(可见)、有哪些事务还未提交(不可见)或者是还未启动(不可见)。

对于当前事务的启动瞬间来说，一个数据版本的row trx_id，有以下几种可能：

1. 如果低于低水位，那肯定是已经提交的，可见；或者就是低水位，那就是当前事务修改的记录，同样可见。
2. 如果是高于高水位，就以为着是当前事务之后创建的，那按照最终要实现的规则，看不见（或者说忽略）。
3. 如果在高水位和低水位之间，我们的任务是区分出哪些事务是在当前事务启动前结束的，哪些是之后结束的。而通过遍历之前记录的数组即可。在数组中，表示未提交，按规则忽略；不在数组中，表示已提交，读进来。

所以，我们事务中的select语句在执行中会有如下步骤：

1. 读取目标行当前版本的row trx_id，判断是否忽略；
2. 若忽略，则根据undo log获取前一个版本，然后回到1；
3. 直到读进来。

>  这里有个细节中的细节，从代码实现上来看，获取视图数组和高水位，是在事务系统的锁保护下做的，可以认为是原子操作，期间不能创建事务。

就是这样， InnoDB实现了"秒级创建快照"。

## 其它事项

这里要注意，update语句或者select语句加锁，都是当前读，即只能读最新版本的值。

- 因为你这时候如果要回到历史版本进行修改的时候，新的记录放哪里呢？所以，只能在最新版本基础上修改。



为什么表结构不支持"可重复读"？

- 这是因为表结构没有对应的行数据，也没有row trx_id，因此只能遵循当前读。



"读提交"级别下，视图是在每个SQL语句执行开始的时候创建的。

---



以上是我根据极客时间<<MySQL实战45讲>>专栏的第3讲和第8讲总结的。