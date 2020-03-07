---
title: synchronized的不同效果实验
date: 2020-02-10 22:57:56
tags:
categories:
---
示例：synchronized方法。锁住的是this。多个线程同时使用一个对象，这个对象里面若有线程执行了这个对象的同步方法，此时其他线程不能使用该同步方法以及其他的同步方法，但是可以使用普通方法。
public class TestSyn02 {
    public static void main(String[] args) {
        Test01 test = new Test01();
        new Thread(test, "1").start();
        new Thread(test, "2").start();
        new Thread(test, "3").start();
    }
}
class Test01 implements Runnable{
    static int num = 0;
    public synchronized void synFaction1(){
        for (int i = 0; i < 10; i++) {
            try {
                Thread.sleep(100);
                System.out.println("1" + "----->" +Thread.currentThread().getName() +"----->" + i);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public synchronized void synFaction2(){
        for (int i = 0; i < 10; i++) {
            try {
                Thread.sleep(100);
                System.out.println("2" + "----->" +Thread.currentThread().getName() +"----->" + i);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public void synFaction3(){
        for (int i = 0; i < 10; i++) {
            try {
                Thread.sleep(100);
                System.out.println("3" + "----->" +Thread.currentThread().getName() +"----->" + i);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    @Override
    public void run() {
        switch (num){
            case 0:{ num++; synFaction1(); break;}
            case 1:{ num++; synFaction2(); break;}
            case 2:synFaction3();
        }
    }
}
运行的结果：
3----->3----->0
1----->1----->0
1----->1----->1
3----->3----->1
3----->3----->2
1----->1----->2
2----->2----->0
2----->2----->1
2----->2----->2
	1. 如果给静态方法加synchronized，那么同步锁加在了类上，类中其他加了同步锁的静态方法都不能用，但是未加同步锁的静态方法和所有的实例方法可以使用。也就是说，静态方法的同步锁和实例方法的同步锁不是同一个锁，互不干涉。
public class TestSyn03 {
    public static void main(String[] args) {
        Test02 test = new Test02();
        Test02 test2 = new Test02();
        new Thread(test,"1").start();
        new Thread(test2,"2").start();
        new Thread(test,"3").start();
        new Thread(test,"4").start();
        new Thread(test,"5").start();
    }
}
class Test02 implements Runnable{
    public synchronized static void synFaction1(){
        for (int i = 0; i < 3; i++) {
            try {
                Thread.sleep(100);
                System.out.println("同步——第一个对象静态同步方法");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public synchronized static void synFaction2(){
        for (int i = 0; i < 3; i++) {
            try {
                Thread.sleep(100);
                System.out.println("同步——第二个对象静态同步方法");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public static void synFaction3(){
        for (int i = 0; i < 3; i++) {
            try {
                Thread.sleep(100);
                System.out.println("非同步——静态非同步方法");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public synchronized  void synFaction4(){
        for (int i = 0; i < 3; i++) {
            try {
                Thread.sleep(100);
                System.out.println("同步——实例同步方法");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public synchronized  void synFaction5(){
        for (int i = 0; i < 3; i++) {
            try {
                Thread.sleep(100);
                System.out.println("非同步——实例非同步方法");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    @Override
    public void run() {
        switch (Thread.currentThread().getName()){
            case "1":synFaction1(); break;
            case "2":synFaction2(); break;
            case "3":synFaction3(); break;
            case "4":synFaction4(); break;
            case "5":synFaction5(); break;
        }
    }
}
显示结果：
同步——第二个对象静态同步方法
非同步——静态非同步方法
非同步——实例非同步方法
非同步——实例非同步方法
同步——第二个对象静态同步方法
非同步——静态非同步方法
非同步——实例非同步方法
非同步——静态非同步方法
同步——第二个对象静态同步方法
同步——实例同步方法
同步——第一个对象静态同步方法
同步——第一个对象静态同步方法
同步——实例同步方法
同步——实例同步方法
同步——第一个对象静态同步方法
	3. 使用synchronized方法块，锁住this
	如示例中，在run方法中用同步块锁住this，在main线程直接调用test1的testSyn1()方法和线程1中通过run()方法调用test1的testSyn1()方法，此时两者交替输出，似乎this没有被锁住。但当我们同时调用test2的run方法时，一个运行完了，另一个才开始运行。
	这是因为main线程在调用test1.testSyn1()时，没有进行锁的判断，所以可以直接运行。而调用run()方法时，因为run方法里有同步锁，需要先判断这个对象是否被锁住，因此会变成阻塞
public class TestSyn04 {
    public static void main(String[] args) {
        Test03 test1 = new Test03();
        test1.name = "test";
        Test03 test2 = new Test03();
        test2.name = "test2";
        new Thread(test1, "线程1").start();
        for (int i = 0; i < 5; i++) {
            test1.testSyn1();
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        new Thread(test2, "线程2").start();
        test2.run();
    }
}
class Test03 implements Runnable{
    String name;
    public void testSyn1(){
        System.out.println(Thread.currentThread().getName()+"\t实例" +name);
    }
    @Override
    public void run() {
        synchronized (this){
            for (int i = 0; i < 5; i++) {
                testSyn1();
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
运行结果：
	线程1	实例test
	main	实例test
	线程1	实例test
	main	实例test
	main	实例test
	线程1	实例test
	main	实例test2
	main	实例test2
	main	实例test2
	线程2	实例test2
	线程2	实例test2
	线程2	实例test2

	4. 使用synchronized方法块锁住类、锁住对象，省略
	5. 同步方法和同步块同时使用
public class TestSyn05 {
    public static void main(String[] args) {
        Test04 test = new Test04();
        new Thread(test).start();
        test.testSyn1();
    }
}
class Test04 implements Runnable{
    public synchronized void testSyn1(){
        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(100);
                System.out.println(Thread.currentThread().getName());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    @Override
    public void run() {
        synchronized (this){
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(100);
                    System.out.println(Thread.currentThread().getName());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
结果：
main
main
main
Thread-0
Thread-0
Thread-0
这个例子里，同步方法和同步块分别在两个线程里调用，而同步块锁住的是this。我们发现两者发生了互斥的结果，这反应了同步方法其实锁住了this，即当前调用的对象。
同理，对类的同步锁也是如此，即同步静态方法其实锁住了这个类。
通过上面的分析，我们可以分析出以下的同步过程：
同步实例方法就是锁住this的同步块，而同步静态方法就是锁住类.class的同步块，但同时，类与这个类的对象直接不是一个同步锁，两者互不影响。
而普通方法，或者没有判断锁的代码依然可以调用被锁住的类，因为运行这些代码的时候，没有经过判断这个类或者对象是否被锁住，形不成阻塞，所以依旧可以使用。