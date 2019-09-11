# JDK动态代理原理分析

参考文档：
Java动态代理实现及原理分析：https://www.jianshu.com/p/23d3f1a2b3c7

# 代理模式
给某个对象提供一个代理对象，并由代理对象控制对于原对象的访问。

# 动态代理
运行时动态生成代理类

# 动态代理需要什么
1. 业务接口（interface）  
业务的抽象表示
2. 业务具体实现类（concreteManager）  
实现业务接口执行具体的业务操作
3. 业务代理类（$proxy在运行时候动态生成类）  
进行业务代理，调用业务代理操作
4. 业务代理操作类（proxyHandler）  
实现了InvocationHandler接口的类，代理方法的直接调用者，通过InvocationHandler的Invoker方法直接发起代理
5. 客户端调用类（client）  
发起业务

# 具体实现
* 业务接口ICook
```
public interface ICook {
     void dealWithFood();
     void cook();
}
```

* 业务实现类CookManager
```
public class CookManager implements ICook {
    @Override
    public void dealWithFood() {
        System.out.println("food had been dealed with");
    }

    @Override
    public void cook() {
        System.out.println("cook food");
    }
}
```

* 业务代理类DynamicProxyHandler
```
public class DynamicProxyHandler implements InvocationHandler{
    Object realCookManager;
    DynamicProxyHandler(ICook realCookManager){
        this.realCookManager = realCookManager;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("invoke start");
        System.out.println(method.getName());
        method.invoke(realCookManager,args);
        System.out.println("invoke end");
        return null;
    }
}
```

* 客户端
```
public class Main {
    public static void main(String[] args){

        CookManager cookManager = new CookManager();
        DynamicProxyHandler dynamicProxyHandler = new DynamicProxyHandler(cookManager);
        ICook iCook =(ICook)Proxy.newProxyInstance(dynamicProxyHandler.getClass().getClassLoader(),cookManager.getClass().getInterfaces(), dynamicProxyHandler);
        //打印一下代理类的类名
        System.out.println(iCook.getClass().getName());
        iCook.dealWithFoot();
        iCook.cook();
    }
```

* 输出结果
```
com.sun.proxy.$Proxy0
invoke start
dealWithFoot
food had been dealed with
invoke end
invoke start
cook
cook food
invoke end
```

## 输出结果分析
输出的业务代理类名称为$proxy0，DynamicProxyhandler中的invoke方法被调用了，并且method.invoke方法会调用实现类中的具体实现方法，到这里我们其实已经完成了代理操作了，并且在DynamicProxyHandler的invoke中我们还可以添加自己的操作，比如打印日志之类的，这里其实就是一个简单的应用层hook的实现了。我们可以在客户端发起调用的时候使用代理中的方法替换掉原有的具体实现方法并对其进行扩展。这样我们在不改变原有实现类的情况下增强了原有类的功能，符合开闭原则。

# 通过源码具体分析
首先从Proxy类中的newProxyInstance这个函数入手，为了更好地理解其原理，以下是精简后的代码：  
```
 public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h){
     //所有被实现的业务接口
      final Class<?>[] intfs = interfaces.clone();
     //寻找或生成指定的代理类
      Class<?> cl = getProxyClass0(loader, intfs);
      //通过反射类中的Constructor获取其所有构造方法
      final Constructor<?> cons = cl.getConstructor(constructorParams);
      //通过Constructor返回代理类的实例
      return cons.newInstance(new Object[]{h});
}
```

该方法入参包含三个参数：  
1. loader：ClassLoader是一个抽象类，其作用是将字节码文件加载进入虚拟机并生成相应的class文件，这里得到的loader是其子类APPClassLoader（负责加载应用层字节码）的一个实例
2. interfaces：就是需要被实现的那些业务接口
3. h：是InvocationHandler接口的实例，具体的代理操作就放在这个InvocationHandler的invoke函数中。

接下来看一下生成业务代理类的getProxyClass0（loader，intfs）的实现
```
private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
      // proxyClassCache会缓存所有的代理类，如果缓存中有这个业务代理类，则会从缓存中取出，否则从ProxyClassFactory中生成
        return proxyClassCache.get(loader, interfaces);
    }
```

ProxyClassFactory是Proxy中的内部类，缓存中如果没有这个代理类则会调用ProxyClassFactory中的apply方法生成  
```
   private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>> {
        // 这两个常量就是代理类名字的由来
        private static final String proxyClassNamePrefix = "$Proxy";
        private static final AtomicLong nextUniqueNumber = new AtomicLong();
        
         @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
        //这里就生成了我们要的字节码形式的代理类
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
        //defineClass0是个native方法        
        return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
        }
        
}
```

到这里业务代理类就生成了，我们再回到newProxyInstance方法中，他会将InvocationHandler的实例h传入到这个业务代理实例中。
```
 return cons.newInstance(new Object[]{h});
```
由于业务代理类是以字节码存在与内存中的，我们想看到起原貌可以将其保存下来然后反编译查看其源码。  
```
   byte[] proxyClassFile =  ProxyGenerator.generateProxyClass(iCook.getClass().getName(),cookManager.getClass().getInterfaces());
   saveToFile(proxyClassFile);
```

最终我们的代理类$Proxy0类是这样的：  
```
public final class $Proxy0 extends Proxy implements ICook {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m4;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void cook() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void dealWithFoot() throws  {
        try {
            super.h.invoke(this, m4, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            m3 = Class.forName("com.company.ICook").getMethod("cook", new Class[0]);
            m4 = Class.forName("com.company.ICook").getMethod("dealWithFoot", new Class[0]);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

这个流程终于变得清晰了：  
当我们将业务接口ICook和业务代理操作类DynamicProxyHandler传入到Proxy中后，Proxy会为我们生成了一个实现了ICook接口并集成了Proxy的业务代理类$Proxy0  
在我们具体调用方法ICook.dealWithFood（）时他其实调用了$Proxy0中的dealWithFood方法，然后在调用Proxy类的invoke方法，所以DynamicProxyHandler中的Invoke方法才是最终执行的方法，这个方法给了我们扩展的可能，并且最终我们实现了代理对象访问源对象的目的，也就是最终通过$Proxy0代理了CookManager对象。