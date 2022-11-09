## 背景

在公司要给的一张表（5亿条数据）新建一条索引时，因为执行耗时较长，并且马上会有写入了。故联系了平台的同学将任务在中断下。


但发现任务停止了后，数据依旧有在跑。

google以及跟平台的同学沟通后，了解了一下平台执行ddl修改，原理其实是用到了pt-osc。在任务终止之后，还需要手动删除掉mysql触发器以及清除掉临时影子表。


```
Not dropping triggers because the tool was interrupted. To drop the triggers, execute:
DROP TRIGGER IF EXISTS hhl.pt_osc_hhl_t1_del;
DROP TRIGGER IF EXISTS hhl.pt_osc_hhl_t1_upd;
DROP TRIGGER IF EXISTS hhl.pt_osc_hhl_t1_ins;
Not dropping the new table hhl._t1_new because the tool was interrupted. To drop the new table, execute:
DROP TABLE IF EXISTS hhl._t1_new;
```

## pt-osc执行ddl操作原理

> 基于以上背景，故了解了一下pt-ost的原理。

### 执行原理

1.  创建一个和要执行 alter 操作的表一样的新的空表结构(是alter之前的结构)
2.  在新表执行alter table 语句
3.  在原表中创建触发器3个触发器分别对应insert,update,delete操作
4.  以一定块大小从原表拷贝数据到临时表，拷贝过程中通过原表上的触发器在原表进行的写操作都会更新到新建的临时表
5.  Rename 原表到old表中，在把临时表Rename为原表
6.  如果有参考该表的外键，根据alter-foreign-keys-method参数的值，检测外键相关的表，做相应设置的处理
7.  默认最后将旧原表删除
    

### 使用限制

1.  原表必须要有主键或者唯一索引（不含NULL）
2.  原表上不能有触发器存在
3.  使用前需保证有足够的磁盘容量，因为复制原表需要一倍的空间
    

### 使用问题

#### 执行过程是否支持DML

*   支持。
*   ALTER过程采用Copy Table To New Table的方式，新建一个表格，然后在原表上创建3个触发器：DELETE\\UPDATE\\INSERT触发器，一旦新表，拷贝数据到新表的过程中，如果原表数据发生变化，则会通过触发器更新到新表上。  
*   如果数据修改的时候，还没有拷贝过来，修改后再拷贝则是覆盖，正确；如果是已经拷贝过来，再修改，也是正确。
    

#### 是否存在锁表情况

*   创建新表后，按照每一个chunk的大小拷贝数据到新表，每次SELECT都是share mode，带S锁，但是每个chunk都比较小，所以锁时间不大。
*   但是在rename过程时，是不支持dml操作的，但是该操作也不会长时间锁表。
    

## mysql触发器

> 上文提到了触发器，我确实也是第一次听到这个概念，故也简单了解了一下。

### 定义

*   触发器（trigger）是MySQL提供给程序员和数据分析员来保证数据完整性的一种方法，它是与表事件相关的特殊的存储过程，它的执行不是由程序调用，也不是手工启动，而是由事件来触发，比如当对一个表进行操作（insert，delete， update）时就会激活它执行。
*   它可以在操作者对表进行「增删改」 之前（或之后）被触发，自动执行一段事先写好的 SQL 代码。
    

#### 关键要素

*   监视地点，即table
*   监视事件，即insert、update、delete
*   触发时间，即before、after
*   触发事件，既insert、update、delete
    

#### 语法

*   查看已有触发器：show triggers
*   删除已有触发器：drop trigger triggerName
*   创建触发器：create trigger triggerName
    

## ddl跟dml解释

*   ddl：date definition language，数据库定义语言。指CREATE、ALTER、DROP等改变表结构，数据类型，表之间的链接、约束等操作。
*   dml：data manipulation language，数据操作语言。指SELECT、UPDATE、INSERT、DELETE这些用于对数据库数据的操作。
    

## 其他开源产品

*   在线变更工具除了pt-osc之外，还有另外一款开源产品gh-ost，两者原理区别简单来提是前者是通过触发器实现，后者是通过binlog。
    *   具体可以查看该blog：[https://www.cnblogs.com/zping/p/8876148.html](https://www.cnblogs.com/zping/p/8876148.html)
        
    

## 参考

*   [https://www.cnblogs.com/xinysu/p/6758170.html#\_label2](https://www.cnblogs.com/xinysu/p/6758170.html#_label2)
*   [https://blog.csdn.net/u010046908/article/details/112661389](https://blog.csdn.net/u010046908/article/details/112661389)
*   [https://zhuanlan.zhihu.com/p/147736116](https://zhuanlan.zhihu.com/p/147736116)