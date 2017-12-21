# High-performance-coding
# 高性能编码

### 1.采用静态数据结构

   对于简单的数据类型，尽量采用静态的数据结构来表达，系统访问静态数据结构比非静态大约快15%～20％。
   如果是常量，建议使用final

    static int a =1;

### 2.循环中不要使用复杂表达式

   循环条件会被反复计算，导致性能变差

    for(int i=0;i＜arry.length;i++) 将arry.length 提成成员变量

### 3.移位操作代替乘除

   2倍数的乘除操作，可以使用移位来完成，提升性能。

    size/1024       size＞＞10

### 4.合理设置组建状态

   当某个组件暂时不需要被使用，但又不想直接销毁该组件，可以动态设置组件的状态来完成此目的。

    PackageManagerService   setComponentEnableSetting


### 5.主动停止长时间空闲的服务

   长时间空闲的服务将使所在进程一直处在 Services(oom_adj =8),进程不容易被杀掉，内存也较难及时让出。
   因此，我们建议这类服务在处理完请求后调用Service.StopSelft()将服务停止，让进程的oom_adj 降到Background


### 6.重要的应用可以适当的调整oom_adj的级别，降低被杀的概率

   相关代码AMS startProcessLocked

### 7.谨慎使用foreach

   PreferForinArrayList/ArrayList 的遍历应采用for循环。
   如果循环对象为arraylist，foreach和Iterator方式在循环过程中会进行数据锁定，导致性能稍微差(时间消耗)
    对于循环体已经实现了Iterator接口或者使用Array数据类型，建议使用增加循环语法结构

8.尽量使用库函数

大部分库函数都是人工优化编译的汇编程序，这些代码的效率远高于JIT(just in time)自动编译产生的代码

9.避免集合类的自动增长

集合类的构造器会创建一个默认的容量大小，在使用中，如果超过这个大小，就会重新分配内存，重构一个更大容量的对象，并且将原先的数据复制过来，再丢弃旧的数据，这样会浪费很多时间。在大多数情况下，可以在创建对象的时候指定大小，或者估计出所需要的最佳大小，这样就避免了在容量不够的时候自动增长，以提高性能。

10.尽量使用final修饰符

带有final修饰符的类是不可派生的。如果一个类是final的，则该类所有方法都是final的。java编译器会寻找机会内联(inline)所有的final方法。此举能够使性能平均提高50%

11.尽量使用基本数据类型代替对象

使用对象会有额外的内存开销

创建一个字符串，而且jvm的字符缓存池还会缓存这个字符串。

此时程序除了会创建一个“hello”字符串外，str所引用的string对象底层还包括一个char[]数组

12.尽可能避免使用内在的get/set方法

android编程中，虚方法的调用会产生很多代价，比实例属性查询的代价还要多。我们应该在外部调用的时候才使用get，set方法，在内部应该直接调用。

13.尽可能少的调用会访问I/O的接口

程序中访问IO的接口被频繁的调用会导致程序允许缓慢，尤其是在方法中调用同步访问IO的方法，需要等访问结束后返回，这类调用如果放在Android APP的UI线程中，也就是MainThread 会导致应用界面卡顿，即使是线程中单独执行，也会大大拖慢程序的相应速度

14.使用substring()代替replace()

String的replace操作需要执行正则表达式的匹配，如果该字符串的前缀是固定的，而我们需要的结果就是替换字符串的前缀，那么可以使用substring来实现。这种替换写法固定，但是在频繁调用的函数中对性能的提升明显。

15.谨慎使用异常，异常对性能不利

抛出异常首先要创建一个新的对象。Throwable接口的构造函数调用名为fillInStackTrace()的本地方法，此方法查找栈，因为在处理过程中创建了一个新的对象。异常只能用于错误处理，不应该用来控制程序流程

Android 平台基本设计

1.性能敏感场景小粒度开启硬件加速(禁用application级别的加速)

硬件加速要求分配额外的内存供GPU使用，这部分内存的消耗往往和分辨率的大小由直接关系，分辨率越大，消耗的内存越多。如果开启整个应用的硬件加速，虽然简单却造成系统资源的损失，所以有必要根据实际的需要，针对更小的颗粒Activity，Window，View进行更精细的控制，关闭整个应用的硬件加速，开启需要的Activity或者Window的硬件加速，以达到关闭其他不需要的Activity或者Window的硬件加速

android:hardwareAccelerated  不声明默认打开  Window无法关闭


2.主动停止长时间空闲的服务


长时间空闲的服务将使所在进程一直处在B service


3.UI线程里尽量避免数据库读写等I╱O操作，至少延后I/O操作。

UI线程中做I/O操作，可能由于系统I/O负载过高，导致无法及时响应，造成UI线程无响应的情况。

4.性能比较敏感场合减少java对象的创建

java对象的创建有一定的系统开销，在性能比较敏感的地方，尽量减少java对象的创建。
android常见的性能敏感处理过程:
测量 onmeasure
布局 onlayout 
绘图 ondraw  dipatchdraw
touch事件处理  ontouchevent dispatchtouchevent()
Adapter: getview()  bindview()


5.缓存频繁使用的服务器数据到本地

用户在浏览在线资源的时候，如果在线资源更新的周期不会很频繁，可以采用缓存最近浏览过的在线资源的信息，以便用户下次再次浏览时，可以直接取缓存中的信息，而不必每次都从服务器侧去获取，从而提高界面加载速度，提升用户感受。

6.缓存频繁使用的成员变量到本地(成员变量)

for(**;i<this.mCount;**)

int a = this.mCount;
for(**;i<a;**)

对象的创建、使用和回收

1.避免创建不必要的对象

创建和销毁任何数据类型的对象，都将销毁系统资源。创建的对象越多，引起内存回收器的清理工作也将越频繁。这会影响到用户的体验。在程序中，应尽量避免创建不必要的数据对象。

String name = getSimpleName();
Log.d(TAG,“”+name);

优化后

Log.d(TAG,getSimpleName();


2.采用在需要的时候才开始创建的策略

有些分支可能不需要创建对象，尽量避免创建无用的对象

String str =“abc”
if(i ==1) list.add(str);

改为if( i==1)｛ String str =“abc”;  list.add（str）｝


3.避免在循环中创建对象

每次循环都会创建对象都需要重新申请内存，对象使用完再释放。应该尽量避免在循环中创建对象。

for（）｛
String str= getString（）;
｝

String str = null;
for（）｛
str = getString（）;
｝


4.尽量重用已有对象

过多的创建对象会消耗系统大量的内存，由于系统不仅要花大量时间生成对象，以后可能还要花时间对这些对象进行垃圾回收和处理。StringBuffer Message

5.保证过期的对象的及时回收

过分的创建对象会消耗系统的大量内存，严重时，会导致内存泄露，因此保证过期的对象的及时回收具有重要意义。JVM的GC并非十分智能，因此建议在对象使用完毕后，手动设置为null。例如服务绑定和解绑。

6.性能敏感场景禁止执行GC操作。

在界面启动，滑动，拖动过程中执行GC操作，会造成明显的卡顿和迟滞感，所以在这些性能敏感的处理过程中禁止执行GC操作

7.使用BitmapFactory.Options.inBitmap处理Bitmap对象，避免重复申请释放大片内存

Android 的3.0提供BitmapFactory.Options.inBitmap属性。如果设置此选项，即采取选项对象解码方法将尝试加载内容重用的现有位图。这意味着该位图的内存进行再利用，从而提升性能，并减少大内存分配和释放。但是，inBitmap使用一定的限制，特别是Android 4.4之前，只有同等大小的位图支持


布局

1.新增无用户交互的视图不增加不必要的图层，避免性能下降

在一个应用窗口上根据新的视图，这个视图如果没有用户交互的需求，不需要单独相应用户事件，例如类似悬浮显示信息，不要采用通过WindowManager新增一个图层的做法。新增图层，往往会导致显示性能下降，尤其是在性能敏感的滑动过程中，会非常明显的。


2.不允许4xUI过度绘制，尽量减少3x过度绘制

过度绘制会增加UI复杂度，增加GPU绘图面积，增加GPU负载，增大GPU内存消耗，导致性能下降，内存增大

3.使用merge标签来减少视图层级结构

在Android layout文件中需要一个顶级容器来容纳其他组件，而不能直接放置多个组件，merge可以去除一层framelayout

4.简化布局层次提高页面启动性能

简化layout的层次关系，尽量减少layout的嵌套层次，建议用relativelayout或gridlayout使页面布局扁平化

5.通过重复利用convertView避免不必要的inflation

6.使用drawingcache提高滑动的平滑行

android系统中的view大多比较繁杂，在绘图操作的过程中，由于需要遍历view tree，对每一个view执行draw操作的复杂度很高，导致draw的时间很长，复杂界面的滑动不够流畅。Android 提供了cache的机制来提高绘图的效率，在android的原始版本的launcher中已经使用了这个机制，但是由于其方式是在拖动开始的时候创建drawing cache，导致滑动的前两帧还是不够流畅。这个问题可以通过始终维护更新缓存来解决。
setChildrenLayersEnabled

7.根据实际显示的高度设定listview的高度

Listview 的显示过程中会根据设定的高度来计算执行几次bindview，实测中发现当列表界面比较复杂的时候，bindview的调用和执行时间相对较长，从而使得拖慢界面的显示速度。在设计界面的layout时:

（1）listview的高度一定不能是wrap_content,否则bindview的调用次数会等于加载的list条数。
（2）尽量根据实际展现给用户的高度来设置listview的高度，减少bindview调用的次数，例如拨号盘界面，初始进入的时候只有上半屏显示通话记录列表，则list的初始高度设置为半屏高度就可以了，其实采用相对布局使其浮动在拨号盘上。

8.listview中的图片尽量采用合适的尺寸，需要缩放的提前进行缩放缓存

运行时的放大/缩小图片是很耗时的，尤其是listview中如果有图片，尽量采用合适的尺寸，避免在显示过程中进行缩放。如果必须进行缩放的情况，尽量提前缩放缓存。

如果场景允许，可以在快速滑动时不显示图片，当快速滑动列表时（SCROLL＿STATE＿FLING），item中的图片获取需要消耗资源的view可以不显示出来，在其他两种状态的时候显示。

9.用ViewStub代替layout,在需要时再加载需要的布局

ViewStub是一个轻量级的view，他是一个看不见，不占布局位置，占用资源非常小的控件。可以为ViewStub指定一个布局，在Inflate布局的时候，只有viewstub会被初始化，然后当viewstub被设置为可见的时候，或者调用了viewstub.inflate()的时候，ViewStub所指向的布局就会被inflate和实例化，然后ViewStub的布局属性都会传给它所指向的布局。这样，就可以使用ViewStub来方便的在运行时，要还是不要显示某个布局


资源

1.避免在滚动屏幕时播放动画

滚到屏幕时播放动画会严重影响帧率，比如Marquee，AnimationDrawable等。如果必须使用动画，请不要使用绘画cache。
listview.setdrawableCacheEnabled(false)

2.尽量使用ViewHolder缓存视图

说明:在滚动listview的时候也许会重复频繁的调用findviewbyid，这样会降低性能。尽管Adapter会因为循环机制返回一个创建好的view，但有时仍然需要查找到这些组件并更新它，为了避免这样的重复，建议可以使用ViewHolder的方式缓存

3.图片大小按需要大小加载

我们在使用BitmapFactory进行本地文件Decode的时候，不一定需要按照原始图片的大小加载，我们先判断下图片的大小，如果是过大的图片则需要减少加载尺寸，满足实际场景的需求即可。

4.更快的屏幕方向变换

在手机屏幕方向变换的时候，会销毁当前activity，并重新create，此时activity的所有数据都需要重新获取。那当这些数据是比较大的时候，例如图片，声音，重新获取的方式是一种非常不明智的做法。为了解决这个问题，我们可以考虑使用如下方法，onRetainNonConfigurationInstance()。该方法允许你在屏幕变换的时候进行数据的传递，你可以在这里传输所需要的数据，然后在onCreate里调用getLastNonConfigurationInstance()获取数据

5.优先考虑点9图片资源 svg


6.小图拼大图

多线程与同步

1.单线程尽量使用HashMap,ArrayList

单线程应尽量使用HashMap,Arraylist,除非必要，否则不推荐使用HashTable,Vector他们使用了同步机制，而降低了性能

2.广播接收处尽量少使用同步代码
