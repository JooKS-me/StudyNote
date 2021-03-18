```shell
select [id]  #选择数据库，默认为0
key *     #查看所有key
flushall  #清空所有key
set x y   #设置key为x，value为y
get x     #获取x的value
move x [id]   #将x移动到数据库id中
expire x [time]   #给x设置过期时间，单位为秒
ttl x      #查看x还有几秒过期
type x     #查看键的类型
exists x   #检查key是否存在
```

## String

```shell
strlen x    #查询x的value的长度
append x y  #给x的value末尾加上y

incr x    #给x增加1
decr x    #给x减少1
incrby x n  #给x增加n
decrby x n  #给x减小n

getrange x start end   #截取字符串x的start到end（从0开始，-1表示最后一个）
setrange x offset str  #把x第offset位起的字符串(长度与str相同)替换为str

setex x seconds value  #给x设置值value和过期时间seconds
setnx x value    #如果x不存在，则设置x的值为value

mset k1 v1 k2 v2 k3 v3 ... #批量设置
msetnx k1 v1 k2 v2 k3 v3 ... #如果不存在则批量设置，是原子性的操作！
mget k1 k2 k3  #批量取值

getset k v  #先获取value再设置value 
```

## List

```shell
lpush list e1 e2 e3   #在左边依次放入
rpush list e1 e2 e3   #在右边依次放入
lpop [nums 可选] #懂的都懂
rpop [nums 可选]

rpoplpush source target #从一个列表出，从另一个列表进去

lset list index "other" #将下标中的值替换成其他值，若不存在则报错

lindex index  #按索引取值
lrange start end      #从左边开始截取

lrem list count v   #移除list中值为v的count个元素

ltrim list start end  #截取指定长度的list

linsert key BEFORE|AFTER pivot element   #将element插入pivot的前面或者后面
```

 ## Set

```shell
sadd myset 123  #给myset添加123，若没有myset则新建
smembers myset  #显示myset的所有成员
sismember myset target  #判断myset中是否有target
scard myset     #获取当前set的元素个数
srem myset xxx   #从myset中移除xxx                                   
srandmember myset [count 可选]   #从myset中随机取元素                                 
spop myset [count 可选]   #从myset随机pop                                  
sdiff s1 s2  #s1与s2的差集
sinter s1 s2  #s1与s2的交集
sunion s1 s2  #s1与s2点并集                                         
```

## 事务

```shell
multi   #开启事务
...
...
...
exec/discard   #提交/放弃
```

redis事务没有原子性

> 只要命令输入错了，整个事务都会放弃

> 但是如果是运行时有错(语法上没问题)，事务中其他语句会正常执行，该语句抛出异常    

## 监视器锁

watch x              给x加锁

unwatch x         给x解锁