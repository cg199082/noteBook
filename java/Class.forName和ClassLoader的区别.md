# Class.forName和ClassLoader的区别

参考文档：  
在Java的反射中，Class.forName和ClassLoader的区别：https://www.cnblogs.com/jimoer/p/9185662.html

## 解释
在java中Class.forName()和ClassLoader都可以对类进行加载。ClassLoader就是遵循双亲委派模型最终调用启动类加载器的类加载器，实现的功能是：通过一个类的全限定名来获取此类的二进制字节流，获取到二进制字节流后放到jvm中，class.forName()方法实际上也是调用的ClassLoader来实现的。

Class.forName(string className)的源代码实现如下：  
```
    @CallerSensitive
    public static Class<?> forName(String className)
                throws ClassNotFoundException {
        Class<?> caller = Reflection.getCallerClass();
        return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
    }
```

最后调用的是forName0这个方法，在这个方法中第二个参数代表是否对加载的类进行初始化，在这里被设置为true代表会对类进行初始化，也就是代表会执行类中的静态代码块，以及对静态变量的赋值操作。  
当然也可以调用Class.forName（string Name, boolean initialize,ClassLoader loader）方法手动的设置是否初始化类，以及设置类加载器，其源码如下：  
```
 @CallerSensitive
    public static Class<?> forName(String name, boolean initialize,
                                   ClassLoader loader)
        throws ClassNotFoundException
    {
        Class<?> caller = null;
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            // Reflective call to get caller class is only needed if a security manager
            // is present.  Avoid the overhead of making this call otherwise.
            caller = Reflection.getCallerClass();
            if (sun.misc.VM.isSystemDomainLoader(loader)) {
                ClassLoader ccl = ClassLoader.getClassLoader(caller);
                if (!sun.misc.VM.isSystemDomainLoader(ccl)) {
                    sm.checkPermission(
                        SecurityConstants.GET_CLASSLOADER_PERMISSION);
                }
            }
        }
        return forName0(name, initialize, loader, caller);
    }

```

## 测试
假如有一个含有静态代码块，静态变量，赋值给静态变量的静态方法类如下：  
```
public class ClassForName {

    //静态代码块
    static {
        System.out.println("执行了静态代码块");
    }
    //静态变量
    private static String staticFiled = staticMethod();

    //赋值静态变量的静态方法
    public static String staticMethod(){
        System.out.println("执行了静态方法");
        return "给静态字段赋值了";
    }
}
```

分别测试Class.forName和ClassLoader调用：  
```
public class MyTest {
    @Test
    public void test44(){

        try {
            Class.forName("com.test.mytest.ClassForName");
            System.out.println("#########分割符(上面是Class.forName的加载过程，下面是ClassLoader的加载过程)##########");
            ClassLoader.getSystemClassLoader().loadClass("com.test.mytest.ClassForName");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
}
```

运行结果：  
```
执行了静态代码块
执行了静态方法
#########分割符(上面是Class.forName的加载过程，下面是ClassLoader的加载过程)##########
```

进一步证实他们的区别就是是否对类进行初始化。

## 应用场景
在spring框架中的IOC实现就是使用了ClassLoader。
Class.ForName()应用到的场景是：经常使用它来加载JDBC驱动，主要的原因是在JDBC规范中明确要求Driver（数据库驱动）类必须向DriverManager注册自己。而mysql的驱动正是在静态代码块中进行注册的，因此使用Class.forName正好可以满足这样的使用需求：  
```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {  
    // ~ Static fields/initializers  
    // ---------------------------------------------  
  
    //  
    // Register ourselves with the DriverManager  
    //  
    static {  
        try {  
            java.sql.DriverManager.registerDriver(new Driver());  
        } catch (SQLException E) {  
            throw new RuntimeException("Can't register driver!");  
        }  
    }  
  
    // ~ Constructors  
    // -----------------------------------------------------------  
  
    /** 
     * Construct a new driver and register it with DriverManager 
     *  
     * @throws SQLException 
     *             if a database error occurs. 
     */  
    public Driver() throws SQLException {  
        // Required for Class.forName().newInstance()  
    }  
}
```