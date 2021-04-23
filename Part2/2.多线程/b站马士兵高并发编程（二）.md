# 一、ReentrantLock

   1. lock方法
   
      1. lock.lock()方法尝试获得一把锁，如果这把锁被别的线程占用了，他就一直阻塞在那里等那把锁，直到别的线程释放
      
      2. 在想要释放锁时，lock一般要在finally里面手动释放锁
      
      3. 代码
         
          ```
             public class ReentrantLock1 {
	
               Lock lock = new ReentrantLock();

               void m1(){
                    try{
                         lock.lock(); //等价于synchronized(this)
                         for(int i = 0; i < 5; i++){
                              TimeUnit.SECONDS.sleep(1);
                              System.out.println(i);
                         }
                    } catch(InterruptedException e){
                         e.printStackTrace();
                    } finally {
                         lock.unlock();
                    }
               }

               void m2(){
                     lock.lock();  //如果没得到锁，线程就会阻塞在这里
                     System.out.println("m2....");
                     lock.unlock();
               }

               public static void main(String[] args) {
                    ReentrantLock1 r1 = new ReentrantLock1();
                    new Thread(new Runnable() {
                         public void run() {
                              r1.m1();
                         }
                    }).start();

                    try {
                         TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                         e.printStackTrace();
                    }
                    new Thread(new Runnable() {
                         public void run() {
                              r1.m2();
                         }
                    }).start();
               }

          }
          ```
	  
   2. tryLock
   
      1. 线程可以指定在等一把锁多长时间后就不等了，继续执行，而不是一直像synchronized一样阻塞在那里等，体现了比synchronized灵活的地方
      
      2. 该方法返回值是boolean，即表示到了指定时间之后是否得到锁
      
      3. 代码
      
         ```
            //假设线程1占用锁5秒，线程2调用tryLock()方法并设置3秒的等待，3秒过后如果没得到锁就不等了，不阻塞，继续执行线程
            
            void m2(){
                boolean locked = false;

                try{
                     locked = lock.tryLock(3, TimeUnit.SECONDS);
                     System.out.println("m2...."+locked);
                } catch(InterruptedException e){
                     e.printStackTrace();
                } finally{
                     if(locked){
                          lock.unlock();
                     }
                }
          }
         ```
	 
   3. lockInterruptibly      
   
      1. 使用lock方法时，如果没得到锁，他就一直阻塞在那里，別的线程调用它的interrupt方法也没办法中断该线程
      
      2. 如果使用lockInterruptibly方法，別的线程调用interrupt就可以强行中断这个线程了，防止别的线程一直不释放锁，他一直等，阻塞在那里
      
      3. 代码
      
         ```
            //线程1一直占着锁不放，线程2调用lockInterruptibly方法，在那等线程1释放锁，但是这时线程1一直释放锁的，所以主线程在过了5秒之后
            //调用interrupt方法把线程2强行中断，这时线程2的lockInterruptibly方法会抛一个异常
            
            public class ReentrantLock3 {
	
               Lock lock = new ReentrantLock();

               void m1(){
                    try{
                         lock.lock(); //synchronized(this)
                         TimeUnit.SECONDS.sleep(2000....);      //让他在这里sleep很长一段时间
                    } catch(InterruptedException e){
                         e.printStackTrace();
                    } finally {
                         lock.unlock();
                    }
               }

               void m2(){
                     try{
                          lock.lockInterruptibly();  //如果被别的线程调用interrupt方法强行中断，这里会抛异常
                          System.out.println("t2 start");
                     } catch(InterruptedException e){
                          System.out.println("t2 interrupted");
                     } finally{
                          lock.unlock();
                     }
               }

               public static void main(String[] args) {
                    ReentrantLock3 r3 = new ReentrantLock3();
                    new Thread(new Runnable() {
                         public void run() {
                              r3.m1();
                         }
                    }).start();

                    Thread t2 = new Thread(new Runnable() {
                         public void run() {
                              r3.m2();
                         }
                    });

                    t2.start();

                    try {
                         TimeUnit.SECONDS.sleep(5);   //main线程过了5秒之后就中断线程2
                    } catch (InterruptedException e) {
                         e.printStackTrace();
                    }
                    t2.interrupt();
               }

          }
         ```	 
	 
   4. ReentrantLock可以指定为公平锁，即哪个线程等的时间长，就能得到锁
   
      1. 代码
   
         ```
            public class ReentrantLock4 {
	
				Lock lock = new ReentrantLock(true);  //这里的true参数是把lock指定为公平锁

				void m1(){
					for(int i = 0; i < 1000; i++){
						try{
							lock.lock(); 
							System.out.println(Thread.currentThread().getName()+"-start");
						} finally {
							lock.unlock();
						}
					}
				}

				public static void main(String[] args) {
					ReentrantLock4 r3 = new ReentrantLock4();
					new Thread(new Runnable() {
						public void run() {
							r3.m1();
						}
					}).start();
					new Thread(new Runnable() {
						public void run() {
							r3.m1();
						}
					}).start();
				}

			}
         ```
	 
      2. 上面代码的执行结果一般都是线程1执行一次，紧接着线程2执行一次，然后又是线程1，然后线程2....一直到for循环结束
      
   5. ReentrantLock和synchronized的区别
  
      1. **为什么synchronized有一对大括号把代码括起来而lock没有？** 
     
         1. 必须再次强调一遍：synchronized是锁的对象，而不是锁代码，只是说synchronized把锁住对象之后要干的活丢到括号里面，所以会有他锁的是代码的错觉。那为什么要把代码丢到括号里呢？因为jvm在synchronized修饰的代码跑完之后会自动释放锁，所以需要知道他代码什么时候跑完，所以需要括号把代码括起来。
        
         2. 再看看lock，一样也是锁住对象，但是jvm不会像synchronized一样在代码跑完之后帮你释放锁，所以他必须手动释放锁，所以lock也就不需要像synchronized一样定义要跑的代码，就不用花括号包代码，如果硬要类比于synchronized的花括号里的代码，个人理解：在lock.lock和unlock之间执行的代码就相当于synchronized花括号中的代码。
     
         3. 这里关注的是锁，而不是代码，因为synchronized有修饰代码范围，所以学lock的时候很容易产生理解上的偏差，把重点跑到代码去，其实应该关注的重点是拿没拿到这把锁

      2. ReentrantLock可以替代synchronized，可以完成他的功能。
     
      3. ReentrantLock比synchronized灵活（tryLock）
      
      4. synchronized默认是不公平锁，ReentrantLock可以指定公平锁      
      
# 二、生产者消费者题

   1. wait在99.9%的情况下都和while一起使用，不要和if一起使用  
   
   2. 如果用if，当两个线程一起wait，阻塞在那里，这时別的线程notifyAll了，这俩线程都被叫醒了，但是线程1快一点，先拿到锁，先继续执行，线程2还是阻塞在 那里，等线程1代码跑完，释放锁，线程2拿到锁继续跑，但这时有可能线程2的if条件在线程1跑完代码之后已经不成立了，如果线程2继续跑的话就会出错了，所以需要把if换成while，让线程2需要继续跑的时候再回去while检测一下条件是否成立。
   
   3. 要用notifyAll而不是notify，如果只叫醒一个线程的话很可能又是叫醒生产者线程，然后她一看满了，又wait，又没通知消费者执行，程序就卡了
   
   4. synchronized/wait/notify一起用，lock/await/singleAll一起用
   
   5. 代码：面试题：写一个固定容量的同步容器，拥有put和get方法，能支持2个生产者线程和10个消费者线程的阻塞调用
   
      ```
         public class MyContainer1<T> {

			final private List<T> list = new ArrayList<>();

			synchronized void put(T t){
				while(list.size() == 10){
					try {
						this.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}

				list.add(t);
				System.out.println(Thread.currentThread().getName()+" put--"+list.size());
				this.notifyAll();
			}

			synchronized T get(){	
				while(list.size() == 0){
					try {
						this.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}

				T t = list.get(list.size() - 1);
				list.remove(t);
				System.out.println(Thread.currentThread().getName()+" get--"+list.size());
				this.notifyAll();

				return t;
			}

			public static void main(String[] args) {
				MyContainer1<String> mc = new MyContainer1<>();

				for(int i = 0; i < 10; i++){
					new Thread(new Runnable() {
						public void run() {
							for(int j = 0; j < 5; j++){
								mc.get();
							}
						}
					}, "c"+i).start();
				}

				for(int i = 0; i < 2; i++){
					new Thread(new Runnable() {
						public void run() {
							for(int j = 0; j < 25; j++){
								mc.put(Thread.currentThread().getName());
							}
						}
					}, "p"+i).start();
				}
			}
		}
      ```
      
# 三、Condition      
  （二）04
  
# 四、ThreadLocal--线程本地变量

   1. 线程私有的变量，可以看做每个线程从主内存复制一份变量下来，自己改，不会影响到其他线程的变量，效率较高
   
   2. 代码
   
      ```
         public class ThreadLocal01 {
	
			ThreadLocal<Person> tl = new ThreadLocal<>();

			public static void main(String[] args) {
				ThreadLocal01 tl1 = new ThreadLocal01();

				new Thread(new Runnable() {
					public void run() {
						try {
							TimeUnit.SECONDS.sleep(2);
						} catch (Exception e) {
							e.printStackTrace();
						}
						System.out.println(tl1.tl.get());  //到时候这里会输出null
					}
				}).start();

				new Thread(new Runnable() {
					public void run() {
						tl1.tl.set(new Person());
					}
				}).start();
			}

			static class Person{
				String name = "zhangsan";
			}
		}
      ```
     
