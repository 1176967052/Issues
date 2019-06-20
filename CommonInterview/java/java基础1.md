1. 重载和重写的区别
   
   + 重载：发生在同一个类中，方法名必须相同，参数的类型，个数，顺序不同，方法的返回值和修饰符可以不同，发生在编译时
   + 重写：发生在父子类中，方法名，参数列表必须相同，返回值的范围小于父类，访问修饰符的范围大于父类，抛出异常的范围小于父类，如果父类方法访问修饰符是private，这子类不能重写该方法
   
2. String和StringBuilder、StringBuffer的区别
  
   + 可变性
     + String类的修饰符是final关键字的字符数组 private final char value[]，所以String对象是不可变的。而StringBuilder和StringBuffer继承自AbstractStringBuilder类，使用的是字符数组保存字符串，没有用final关键字修饰，所以是可变的
   + 线程安全性
     + String中对象是不可变的，可以理解为常量，线程安全。StringBuilder对继承自AbstractStringBuilder的基本方法没有加同步锁，所以是非线程安全的。而StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，是线程安全的
   + 性能及执行效率
     + String是不可变的，每次对String类型的数据改变的时候，都会生成一个新的String对象，然后指针指向新的String对象。StringBuilder和StringBuffer是对对象本身进行操作，不是生成新的对象并改变对象的引用，但StringBuffer是线程安全的，会增加性能消耗。
      
    总结下来：（1） 少量数据操作用String （2）单线程操作大量数据用StringBuilder （3）多线程大量数据用StringBuffer

3. 装箱与拆箱

   + 装箱：将基本类型用对应的引用类型包装起来
   + 拆箱：将包装类型转换为基本数据类型

4. ==和equals()方法的区别
   
   == : 作用是判断两个对象是否相等，即两个对象是不是同一个对象。（基本数据类型==比较的是值，引用数据类型比较的是内存地址）
   
   equals() : 作用也是判断两个对象是否相等，分两种情况：
   + 类没有覆盖equals方法 ，则等价于通过"=="比较两个对象
   + 类覆盖equals方法，则比较两个对象的内容是否相等
   
   说明：String类中的equals方法是被重写过的，因为Object的equals方法比较的是对象的内存地址。而String的equals方法比较的是对象的值。当创建String类型对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的对象的值是否相同，如果有就把他赋给当前引用，如果没有就在常量池中重新创建一个String对象

5. final关键字总结
   
   final关键字主要使用在三个地方：变量，方法，类
   + 对于一个final变量，如果是基本数据类型，则其数值一旦初始化之后便不能改，如果是引用数据类型的变量，则其初始化以后便不能再指向另一个对象
   + 当final修饰一个类时，表明这个类不能被继承。final类中的所有成员方法都会隐式的指定为final方法
   + 使用final方法的原因有两个。第一个是把方法锁定，以防止任何继承类修改它的含义。第二个是早期java版本中，会将final方法转为内嵌调用。类中所有的private方法都隐式的指定为final
  
6. Object类中常见的方法总结
  
   Object类是一个特殊的类，是所有类的父类。
   
   + public final native Class<?> getClass() //native方法，用于返回当前运行时对象的Class对象，使用了 final关键字修饰，故不允许子类重写。
   + public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的 HashMap。
   + public boolean equals(Object obj) //用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用 户比较字符串的值是否相等。
   + protected native Object clone() throws CloneNotSupportedException //naitive方法，用于创建并返 回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true， x.clone().getClass() == x.getClass() 为true。Object本身没有实现Cloneable接口，所以不重写clone 方法并且进行调用的话会发生CloneNotSupportedException异常。
   + public String toString() //返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个 方法。
   + public final native void notify() //native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监 视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
   + public final native void notifyAll() //native方法，并且不能重写。跟notify一样，唯一的区别就是会唤 醒在此对象监视器上等待的所有线程，而不是一个线程。
   + public final native void wait(long timeout) throws InterruptedException //native方法，并且不 能重写。暂停线程的执行。注意:sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。
   + public final void wait(long timeout, int nanos) throws InterruptedException //多了nanos参 数，这个参数表示额外时间(以毫微秒为单位，范围是 0-999999)。 所以超时的时间还需要加上nanos毫秒。
   + public final void wait() throws InterruptedException //跟之前的2个wait方法一样，只不过该方法一直 等待，没有超时时间这个概念
   + protected void finalize() throws Throwable { }//实例被垃圾回收器回收的时候触发的操作

7. static关键字
   
   static关键字主要使用在四个地方：成员变量和成员方法，静态代码块，静态内部类，静态导包
   + 成员变量和成员方法：被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。调用格式：类名.静态变量名 类名.静态方法名()   
   + 静态代码块: 静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。 该类不管创建多少对象，静态代码块只执行一次
   + 静态内部类（static修饰类的话只能修饰内部类）： 静态内部类与非静态内部类之间存在一个最大的区别: 非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：1. 它的创建是不需要依赖外围类的创建。2. 它不能使用任何外围类的非static成员变量和方法。
   + 静态导包(用来导入类中的静态资源，1.5之后的新特性): 格式为：import static 这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法。

8. this关键字
   
    this关键字用于引用类的当前实例
    ```
      class Manager {
          Employees[] employees;
           
          void manageEmployees() {
              int totalEmp = this.employees.length;
              System.out.println("Total employees: " + totalEmp);
              this.report();
          }
          void report() { }
      }
    ```
    在上面的示例中，this关键字用于两个地方：
    + this.employees.length：访问类Manager的当前实例的变量。
    + this.report（）：调用类Manager的当前实例的方法。
    
    此关键字是可选的，这意味着如果上面的示例在不使用此关键字的情况下表现相同。 但是，使用此关键字可能会使代码更易读或易懂。
    
9. super关键字

   super关键字用于从子类访问父类的变量和方法
   
   使用 this 和 super 要注意的问题：
   + 在构造器中使用 super（） 调用父类中的其他构造方法时，该语句必须处于构造器的首行，否则编译器会报错。另外，this 调用本类中的其他构造方法时，也要放在首行。
   + this、super不能用在static方法中。
   
   简单解释一下：
   
   被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享。而 this 代表对本类对象的引用，指向本类对象；而 super 代表对父类对象的引用，指向父类对象；所以， this和super是属于对象范畴的东西，而静态方法是属于类范畴的东西。

10. transient关键字
    
    对于不想进行序列化的变量，使用transient关键字修饰。
    
    transient关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被transient修饰的变量值不会被持久化和恢复。transient只能修饰变量，不能修饰类和方法。

11. 获取用键盘输入常用的的两种方法
    方法1：通过 Scanner
    ```
    Scanner input = new Scanner(System.in);
    String s  = input.nextLine();
    input.close();  
    ```
    方法2：通过 BufferedReader
    ```
    BufferedReader input = new BufferedReader(new InputStreamReader(System.in)); 
    String s = input.readLine(); 
    ```
    