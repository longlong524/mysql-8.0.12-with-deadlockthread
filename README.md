

#  为mysql增加了一个deadlock thread 定期检测死锁，并且关闭同步检测死锁的功能。

# 设计初衷
mysql的同步检测死锁耗费过多的cpu，极端情况还会引起大量不必要的锁竞争。
如果关闭死锁检测，锁的等待时间过长会使得业务长时间等待，设置等待时间不是一个根本的解决办法。

所以我们引入了一个单独的线程来进行死锁检测。

# 优势
1. 耗费更少CPU，减少mutex等资源竞争。死锁本来就是比较罕见的，并不是每次加锁都会死锁，所以很多死锁检测是没有必要的。
2. 更快检测到死锁。比关闭死锁检测等待锁超时这种策略能更快的检测到死锁。

# 使用
linux原生的patch。8.0.13的patch，8以上的版本应该都可以用。
完成之后无需任何设置。innodb每隔1000ms检测一次死锁。还有不要设置innodb_deadlock_detect变量。


# 性能测试
1500个线程同时更新一行，测试本patch、关闭死锁检测、打开死锁检测，测试版本为mysql8.0.13
结果：
patch与关闭死锁检测的TPS基本持平（150），比打开死锁检测高一倍（80左右）。必需使用大线程数量才能测出来，不然三者基本持平。
