# java多线程并发领域有三大块内容：

   1. synchronizer（同步器）
   
   2. 同步容器（concurrentHashMap之流）
   
   3. 线程池


# 一、相关知识

   1. synchronized
   
      1. synchronized方法被一个线程执行时，其他线程可以执行非synchronized方法，不需要那把锁，不影响
      
      2. synchronized锁定的是对象，不是代码
      
      3. 如果用synchronized修饰一个成员方法，那他锁住的是this；如果修饰一个static方法，那个锁住的是该类的class对象

   2. 
      1. 匿名内部类写法
      ```
         new Thread(new Runnable()){
           public void run(){
             ....
           }
         }
      ```
      
      2. lambda表达式写法
      ```
         new Thread(() -> {
			
		   }).start();
      ```

   3. 脏读

      1. 写的时候加锁，读没有加锁，可能会读到在写的过程中还没有完成的数据

   4. 一个同步方法可以调用另一个同步方法，一个线程已经拥有某个对象的锁，再次申请的时候仍然会得到，对象的锁，也就是说synchronized获得的锁是可重入的，本身支持重入锁
      ```
         synchronized a(){
           b();
         }
         synchronized b(){}
      ```

      * 重入锁的另一个情形：子类同步方法可以调用父类同步方法

   5. 写一个程序模拟死锁

      1. 线程a执行的方法需要锁定o1，再锁o2

      2. 线程b执行的方法需要锁定o2，再锁o1

      3. 线程a锁完o1线程b就开始执行，锁o2，就产生死锁了

   6. 程序中出现异常，默认情况下锁会被释放，所以要非常小心处理同步业务逻辑中的异常，所以要try/catch做出对异常的处理
   
   7. synchronized代码块中的语句越少越好，只有某些语句需要同步，就给对应语句加synchronized，不需要给整个方法加synchronized，细粒度锁比高粒度锁执行效率高不少
   
   8. 锁定某对象o，如果o的属性改变，不影响锁的使用，如果o变成另一个新对象，那锁将锁住新对象。代码：
   
      ```
         public class T {
            Object o = new Object();

            void aa(){
               synchronized(o){
                  while(true){
                     System.out.println(Thread.currentThread().getName());
                  }
               }
            }

            public static void main(String[] args) {
               final T t = new T();

               new Thread(new Runnable() {
                  @Override
                  public void run() {
                     t.aa();
                  }
               }).start();

               try {
                  Thread.sleep(100);
               } catch (InterruptedException e) {
                  e.printStackTrace();
               }

               t.o = new Object();

               new Thread(new Runnable() {
                  @Override
                  public void run() {
                     t.aa();
                  }
               }).start();

            }
         }
      ```
      解析：main方法起了两个线程，线程a先启动，锁定对象o，如果没有t.o = new Object()，线程b是执行不了的，因为o被线程a锁定了，当执行t.o = new Object()把o对象换成一个新的Object之后，原来那个对象o的锁就被释放了，这时线程b发现t对象里的对象o，也就是这个新的Object对象还没被上锁，他就将获得这个新o对象的锁，锁住o对象，方法得以执行，而线程a也会继续运行，一开始锁住的那个老的o对象还在堆内存里，只是不上锁了，现在上锁的是那个new出来的Object，值得注意的是：synchronized锁住的是那个new出来的在堆内存的Object，而不是栈里面的o变量
      **用final修饰，禁止修改**
      
   9. 不要用字符串常量作为锁定对象，容易发生死锁，而且非常隐蔽，难以发现      
   
      ```
         public class T {
            String s1 = "hello";
            String s2 = "hello";

            void m1(){
               synchronized (s1) {

               }
            }

            void m2(){
               synchronized (s2) {

               }
            }
         }
      ```

# 二、什么是原子性   

   1. 定义：一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行
   
      * 举例：比如从账户A向账户B转1000元，那么必然包括2个操作：从账户A减去1000元，往账户B加上1000元。假如从账户A减去1000元之后，操作突然中止。这样就会导致账户A虽然减去了1000元，但是账户B没有收到这个转过来的1000元。所以这2个操作必须要具备原子性才能保证不出现一些意外的问题。
      
   2. java的原子性
   
      1. 在Java中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。
      
      2. Java内存模型只保证了基本读取和赋值是原子性操作，如果要实现更大范围操作的原子性，可以通过synchronized和Lock来实现
      
   3. AtomicXXX
   
      1. Atomic开头的类(例如AtomicInteger)，他的方法都具有原子性，这种比synchronized效率高得多
      
      2. 但是多个方法一起执行并不构成原子性，就是两个原子性操作的中间还是可能会有別的线程插进来进行操作，代码：
      
         ```
            AtomicInteger a = new AtomicInteger(1);
            if(a.get() < 100){
               a.addAndGet(1);
            }
         ```
         解析：a的get()方法和a.addAndGet()方法具有原子性，但是凑在一起就不构成原子性了，因为有可能当线程a判断if的a.get()为99时，还没来得及执行a.addAndGet()，线程b就插进来执行if的a.get()了这时候线程b获得的a.get()也是99，接下来也会执行a.addAndGet()，这时有可能是线程a/b先执行a.addAndGet方法，就假设线程a先执行，执行完后变量a更新进主内存，接下来轮到线程b执行a.addAndGet方法，这时他在主内存拿到的a变量已经是100了，但因为之前已经判断过if里面的a.get()<100了，所以还是会执行a.addAndGet()，再刷进主内存，这时a的值就是101了，出错了

# 三、volatile

   1. 作用：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
   
   2. volatile如何实现保证可见性
   
      代码：
   
      ```
         public class T {
          /*volatile*/ boolean flag = true;

          void m(){
            while(flag){}
            System.out.println("end");
          }

          public static void main(String[] args) {
            final T t = new T();
            new Thread(new Runnable(){
              @Override
              public void run() {
                t.m();
              }
            }).start();
            try {
              Thread.sleep(10);
            } catch (InterruptedException e) {
              e.printStackTrace();
            }
            t.flag = false;
          }
        }
      ```
      1. 解析：
         
         每个线程都有自己的一个内存区域，然后多个线程跟主内存进行交互，对象也放在主内存里面，每个线程都有一个缓存区，线程操作主内存的对象中的变量时，是把变量从主内存拷贝一份放到自己的缓存区进行修改，改完了再写回去主内存，接下来看上面的代码：
      
         main方法里起一个线程a执行m()方法，这个线程先把flag从主内存读到缓存区中，然后判断flag，因为这时候flag==true，所以是死循环，这种情况下cpu很忙，腾不出时间来从主内存获取最新的flag的值，所以当main线程把t.flag更新为false时，那个线程没有从主内存获取最新的flag，一直还是用那个最初的flag值去判断，就跳不出循环，线程就不能结束了
         
         1. 如果在while循环里面写一个System.out输出语句，这样在while循环执行期间，cpu可能会有一段空闲去主内存读取最新的flag值，就有可能读到被main线程修改过的flag，就有可能跳出循环
         
      2. 如果用volatile会发生什么
      
         1. 使用volatile关键字会**强制**将修改的值立即写入**主存**，在线程main修改flag值时，包括2个操作，修改main线程工作内存中的值，然后将修改后的值写入主存，这样的修改会使得线程a的工作内存中缓存变量flag的缓存行无效（反映到硬件层的话，就是CPU的L1或者L2缓存中对应的缓存行无效），然后线程a读取时，发现自己的缓存行无效，它会等待缓存行对应的主存地址被更新之后，然后去对应的主存读取最新的值。
      
   3. volatile和synchronized的区别

      1. synchronized既有可见性，又有原子性，而volatile只保证可见性，不能保证多个线程共同修改变量时所带来的不一致问题，不能代替synchronized

      2. synchronized效率比volatile低不少，所以只需要保证可见性的时候就不要用synchronized

      3. volatile不能保证原子性的例子：

         1. 两个线程操作volatile变量v，线程a拿到变量v给加到10，刷进主存，线程b开始运行，这时它在主存拿到v，拷贝到缓存区，v的值是10，然后给他加到20，然后写回去，但是在他写回去主内存之前，线程a已经把v加到30，也已经写到主内存了，这时线程b把v(值是20)写进主内存的时候就会把30覆盖掉，因为：

            1. 线程b从缓存区读取v的动作发生在线程a把v=30更新进主存之前，所以不会导致他的缓存行失效，所以线程b就直接在缓存区拿v了，加到20再写主存

            2. 写进主存时是不会判断主内存中v的值还是不是最初他从主内存中拿v时候的值的，所以说volatile只能保证可见性，不能保证原子性

            3. **总结：** 在一个线程尚未完成把修改的volatile变量 **更新进主存之前**，另一个线程**已经获得**该volatile变量的值，并且做出修改，那就会出现不能保证原子性的情况

            3. 这种情况就要使用synchronized了，先让线程a把v加完到30，线程b再操作 三个程序12-14
            
# 四、wait/notify

   1. 线程之间通信用的，wait释放锁，notify不释放锁，所以wait必须在synchronized代码块里面调用
   
   2. 线程notify完了之后自己得再做wait释放锁，让其他线程继续执行
   
   3. wait/notify能完成的很多，但是能不要用就不要用，太麻烦，需要非常小心，在多线程中用就相当于用汇编在编程一样
   
   4. 永远使用notifyAll，不要使用notify

# 五、CountDownLatch

   1. 就是一个门闩，await()方法相当于上门闩，给一个初始值，每调用一次countDown()方法值减1，当初始值变为0时，门闩打开
            




