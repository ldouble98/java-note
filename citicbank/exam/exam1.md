# 双重检查锁的单例模式
```java
public class Singleton {
        private static volatile Singleton singleton;

        private Singleton() {
        }

        public static Singleton getInstance() {
            if (singleton == null) {
                synchronized (Singleton.class) {
                    if (singleton == null) {
                        singleton = new Singleton();
                    }
                }
            }
            return singleton;
        }

    }
```
# HashMap 取值  一个value有三个abc字符串
```java

public class CountABC {
    public static void main(String[] args) {
        HashMap<Integer, String> map = new HashMap<>();
        map.put(1, "abcdabcabc");
        map.put(2, "abcdabc2abc");
        map.put(3, "abcdabc3232abc");
        map.put(4, "123abcdabcabc213123");
        map.put(5, "asdsfasdf");
        map.put(6, "asdabc,dfasabcabcasds");
        System.out.println(countABC(map));
    }

    private static int countABC(HashMap<Integer, String> map) {
        int i = 0;
        // HashMap 遍历方式
        // Iterator iterator = map.keySet().iterator();
        // while (iterator.hasNext()) {
        //     Integer key = (Integer) iterator.next();
        //     String value = map.get(key);
        //
        // }
        // // 遍历方式2  
        Iterator<Map.Entry<Integer, String>> iterator1 = map.entrySet().iterator();
        while (iterator1.hasNext()) {
            Map.Entry<Integer, String> entry = iterator1.next();
            String value1 = entry.getValue();
            int j;
            //  我想的一个方法是每遇到一个abc就把字符串截取一下。
            for (j=0; j < 3; j++) {
                int result = value1.indexOf("abc");
                if (result != -1) {
                    value1 = value1.substring(result + 3);
                } else {
                    break;
                }
            }
            if (j == 3) {
                i++;
            }
        }
        // 遍历方式3  jdk1.8
        // map.forEach((key,value)->{
        //     Integer key3 = key;
        //     System.out.println(key);
        //     String value3 = value;
        //     System.out.println(value3);
        //
        //
        // });
        return i;
    }
}
```
# 编译错误与运行时错误
    编译错误 一般指逻辑错误，比如缺少括号，缺分号等，还有一类如局部变量没赋初始值、调用不存在的方法等
    一般编译器如Eclipse、idea会帮我们识别这类错误。
    运行时错误 一般指我们出现的exception，如NullPointerException，ArrayIndexBoundOutOfRangeException
    
# 选择题

- 子类调用从父类继承过来的方法

    这里要区分`继承`和`多态`的概念，
    `多态`分为`运行时多态`和`编译时多态`。 `方法重载`都是`编译时多态`。根据实际参数的数据类型、个数和次序，
    Java在编译时能够确定执行`重载方法`中的哪一个。`方法覆盖`表现出`两种`多态性，当`对象`引用`本类实例`时，为`编译时多态`，否则为`运行时多态`。
    `运行时多态`是多指父类指向子类的引用，并且`子类重写父类`的方法
    ，当父类指向子类的引用调用子类独有的方法时会报`编译时错误`，
   
- 不管构造方式是什么修饰符，当我们初始化这个类的实例时，都会调用他的构造方法
  
   `单例`模式的`构造方法`就是使用`private`修饰符修饰， Singleton模式，指的是`一个类`，在`一个JVM`里，只有`一个实例`存在。
   `单例`模式又分为`懒汉式`和`饿汉式`。`懒汉式`是延时加载，只有使用的时候才会加载；并且有`线程安全`的考量。`饿汉式`是立即加载，论是否会用到这个对象，都会加载

- 初始化顺序
    
    父类静态变量-父类静态初始化块-子类静态变量-子类静态初始化块
    父类初始化快，构造方法-子类初始化快，构造方法
    
    静态代码块只会`初始化一次`，初始化子类时`也会调用`父类的`非静态代码块`和`构造方法`
    
    静态变量是所有`对象共享`的
    
- i++;++i;return i++; return ++i; i=i++;i=++i;
    
    i++: 先赋值后运算
    
    ++i: 先运算后赋值
        
        TODO 待办
    
    
    
        
