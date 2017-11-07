# High-performance-coding

### 1.采用静态数据结构

对于简单的数据类型，尽量采用静态的数据结构来表达，系统访问静态数据结构比非静态大约快15%～20％。
如果是常量，建议使用final

static int a =1;

### 2.循环中不要使用复杂表达式

循环条件会被反复计算，导致性能变差

例:for(int i=0;i＜arry.length;i＋＋)

### 3.循环中不要使用同步方法

方法的同步需要消耗大量资源，循环中调用同步的方法，会导致性能下降，必须要同步的话将同步移出循环。

### 4.移位操作代替乘除

2倍数的乘除操作，可以使用移位来完成，提升性能。

size/1024       size＞＞10

### 5.合理设置组建状态

当某个组件暂时不需要被使用，但又不想直接销毁该组件，可以动态设置组件的状态来完成此目的。

PackageManagerService   setComponentEnableSetting

### 6.listview中图片尽量采用合适的尺寸，需要缩放的提前进行缩放缓存

运行时的缩放图片很耗时，避免在显示过程中进行缩放。尽量提前。

### 7.主动停止长时间空闲的服务

长时间空闲的服务将使所在进程一直处在 Services(oom＿adj =8),进程不容易被杀掉，内存也较难及时让出。
因此，我们建议这类服务在处理完请求后调用Service.StopSelft()将服务停止，让进程的oom＿adh 降到Background

### 8.建议在静态初始化时读取只读系统属性

ro开头的只读属性，可以视为常量    static final

### 9.重要的应用可以适当的调整oom_adj的级别，降低被杀的概率

相关代码AMS startProcessLocked

### 10.谨慎使用foreach

PreferForinArrayList/ArrayList 的遍历应采用for循环。
如果循环对象为arraylist，foreach和Iterator方式在循环过程中会进行数据锁定，导致性能稍微差(时间消耗)
