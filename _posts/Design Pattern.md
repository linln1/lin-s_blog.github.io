---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/29 18:26:39 -->


<div id="toc"></div>
#Java笔记
##Design Pattern

###创建型模式
####Factory Method
    定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使一个类的实例化延迟到其子类。
    ┌─────────────┐      ┌─────────────┐
    │   Product   │      │   Factory   │
    └─────────────┘      └─────────────┘
        ▲                    ▲
        │                    │
    ┌─────────────┐      ┌─────────────┐
    │ ProductImpl │<─ ─ ─│ FactoryImpl │
    └─────────────┘      └─────────────┘
    public interface NumberFactory{
        Number parse(String s);
        static NumberFactory getFactory(){
            return imp1;
        }
        static NumberFactory imp1 = new NumberFactoryImp1();
    }//工厂接口
    public class NumberFactoryImp1 implements NumberFactory{
        public Number parse(String s){
            return new BigDecimal(s);
        }
    }//工厂实现类
    NumberFactory factory = NumberFactory.getFactory();
    Number result = factory.parse("s");

    //静态工厂方法
    public class NumberFactory{
        public static Number parse(String s){
            return new BigDecimal(s);
        }
    }

    eg: Integer
    public final class Integer{
        public static Integer valueOf(int i){
            if (i>=IntegerCache.low && i<=IntegerCache.high){
                return IntegerCache.cache[ i + (-IntegerCache.low)];
            }
            return new Integer(i);
        }
    }

####Abstarct Factory
                                    ┌────────┐
                                ─ >│ProductA│
    ┌────────┐    ┌─────────┐   │  └────────┘
    │ Client │─ ─>│ Factory │─ ─
    └────────┘    └─────────┘   │┌────────┐
                    ▲         ─ >│ProductB│
            ┌───────┴───────┐    └────────┘
            │               │
        ┌─────────┐     ┌─────────┐
        │Factory1 │     │Factory2 │
        └─────────┘     └─────────┘
            │   ┌─────────┐ │      ┌─────────┐
             ─ >│ProductA1│  ─ >   │ProductA2│
            │   └─────────┘ │      └─────────┘
                ┌─────────┐     ┌─────────┐
            └ ─>│ProductB1│ └ ─>│ProductB2│
                └─────────┘     └─────────┘
    不但工厂是抽象的，产品是抽象的，而且有多个产品需要创建，因此，这个抽象工厂会对应到多个实际工厂，每个实际工厂负责创建多个实际产品
    多个供应商负责提供一系列类型的产品

    Service : Markdown --> HTML && Word
    public interface AbstractFactory{
        HtmlDocument createHtml(String md);
        WordDocument createWord(String md);
    }
    //产品也是抽象的
    public interface HtmlDocument{
        String toHtml();
        void save(Path path) throws IOException;
    }

    public interface WordDocument{
        String toWord();
        void save(Path path) throws IOException;
    }
    因为实现它们比较困难，我们决定让供应商来完成。


####Builder
    将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示
    是使用多个“小型”工厂来最终创建出一个完整对象



####Prototype
    用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象
    创建过程很简单，就是把现有数组的元素复制到新数组。如果我们把这个创建过程封装一下，就成了原型模式
    String[] dupilicated = Arrays.copyOf(original, original.length);

    对于普通类,Java Obeject提供了一个clone()方法,意图就是复制一个新的对象出来,需要实现一个Cloneable接口来表示一个对象时可复制的
    public class Students implements Cloneable{
        private int id;
        private String name;
        private int score;

        public Object clone(){
            Student std = new Student();
            std.id = this.id;
            std.name = this.name;
            std.score= this.score;
            return std;
        }
    }//这种方法的缺点是，每次clone之后都要强制转型

    更好的时定义一个copy()方法,明确返回类型
    public class Student{
                    private int id;
        private String name;
        private int score;

        public Student copy(){
            Student std = new Student();
            std.id = this.id;
            std.name = this.name;
            std.score= this.score;
            return std;
        }
    }
####Singleton
    保证一个类仅有一个实例，并提供一个访问它的全局访问点
    目的是为了保证在一个进程中，某个类有且仅有一个实例
    自然不能让调用方使用new Xyz()来创建实例,构造方法是private
    在类的内部，是可以用一个静态字段来引用唯一创建的实例的
    public class Singleton{
        private static final Singleton INSTANCE = new Singleton();

        private Singleton(){

        }
        //提供一个静态方法，直接返回实例
        public static Singleton getInstance(){
            return INSTANCE;
        }
    }

    或者不需要静态方法，直接public static final Singleton INSTANCE;


###结构型模式:
    如何组合各种对象以便获得更好、更灵活的结构
####适配器
    将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类可以一起工作

    public class Task implements Callable<Long>{
        private long num;
        public Task(long num){
            this.num = num;
        }

        public Long call() throws Exception{
            long r = 0;
            for(long n = 1; n <= this.num; n++){
                r += n;
            }
            System.out.println("Result" + r);
            return r;
        }
    }

    Callable<Long> callable = new Task(123450000L);
    Thread thread = new Thread(callable);
    thread.start();

    Thread接收Runnable接口，但不接收Callable接口
    Thread thread = new Thread(new RunnableAdapter(callable));

    public class RunnableAdapter implements Runnable{
        private Callable<?> callable;

        public RunnableAdapter(Callable<?> callable){
            this.callable = callable;
        }

        public void run(){
            try{
                callable.call();
            }catch(Exception e){
                throw new RuntimeException(e);
            }
        }
    }

    public BAdapter implements B {
        private A a;
        public BAdapter(A a) {
            this.a = a;
        }
        public void b() {
            a.a();
        }
    }
####桥接
    将抽象部分与它的实现部分分离，使它们都可以独立地变化
    桥接模式就是为了避免直接继承带来的子类爆炸

####组合
    将对象组合成树形结构以表示“部分-整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性

####装饰器
    动态地给一个对象添加一些额外的职责。就增加功能来说，相比生成子类更为灵活。
    Decorator模式的目的就是把一个一个的附加功能，用Decorator的方式给一层一层地累加到原始数据源上，最终，通过组合获得我们想要的功能
    InputStream input = new GZIPInputStream( // 第二层装饰
                        new BufferedInputStream( // 第一层装饰
                            new FileInputStream("test.gz") // 核心功能
                        ));
                ┌───────────┐
                │ Component │
                └───────────┘
                    ▲
        ┌────────────┼─────────────────┐
        │            │                 │
    ┌───────────┐┌───────────┐     ┌───────────┐
    │ComponentA ││ComponentB │...  │ Decorator │
    └───────────┘└───────────┘     └───────────┘
                                        ▲
                                ┌──────┴──────┐
                                │             │
                            ┌───────────┐ ┌───────────┐
                            │DecoratorA │ │DecoratorB │...
                            └───────────┘ └───────────┘
    public abstract class NodeDecorator implements TextNode{
        protected final TextNode target;

        protected NodeDecorator(TextNode target) {
            this.target = target;
        }

        public void setText(String text) {
            this.target.setText(text);
        }
    }


####外观Facade
    为子系统中的一组接口提供一个一致的界面。Facade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用
    相当于中介
    很多Web程序，内部有多个子系统提供服务，经常使用一个统一的Facade入口

####享元
    运用共享技术有效地支持大量细粒度的对象
    
####代理

    