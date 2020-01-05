# 条例三:使用私有构造函数或枚举类型实现单例属性

　　单例就是只实例化一次的类[Gamma95]。单例对象通常表示无状态对象，如函数(条例24)或内部唯一的系统组件。
**测试单例类的客户端代码是困难的**，因为除非单例类实现一个用作其类型的接口，否则无法用模拟实现代替单例。

　　有两种实现单例的常用方式。两者都基于保持构造函数私有并导出公共静态成员以提供对唯一实例的访问。

在一种方式中，成员是final修饰的字段：
	
```java
//具有public final字段的单例
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}

    public void leaveTheBuilding() {}
}
```

 　　私有构造函数只在初始化公共静态最终字段Elvis.INSTANCE时调用一次。没有公共和受保护的构造函数确保了单例:  
一旦初始化Elvis类将只存在一个Elvis实例——不多也不少。客户端无论如何都不能改变这一点。  

　　但有一点需要注意：特权客户端可以借助`AccessibleObject.setAccessible`方法以反射(条例65)的形式调用私有构造函数。
如果您需要防御这种攻击，请修改构造函数使其被要求创建第二个实例时抛出异常。

在实现单例的第二种方式中，公共成员是一个静态工厂方法：
```java
//具有静态工厂的单例
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {}
}
```
  对`Elvis.getInstance`的所有调用都返回相同的对象引用，且不会再创建任何其它的`Elvis`实例。(上面提到的注意点这边同样适用)
	
公共字段方式
- 主要优点是API清楚地表明该类是单例：公共静态字段是final的，因此它将始终包含相同的对象引用。  
- 第二个优点是它更简单。  

静态工厂方法方式
- 一个优点是它使您可以灵活地更改关于类是否为单例的想法，而无需更改其API。工厂方法返回
唯一的实例，但是可以对其进行修改，比如，为每个调用它的线程返回一个单独的实例。  　  
- 第二个优点是，如果应用程序需要，可以编写一个*通用单例工厂*(条例30)。  
- 使用静态工厂的最后一个优点是可以将*方法引用*用作supplier实例，比如`Elvis::instance`是`Supplier<Elvis>`的一个实例。  
除非要用到这些优点中的一个，否则优先使用公共字段方式。
	
　　这两种方式不管采用哪一种实现的单例类如果要序列化(第12章)，仅在类的声明中添加`implements Serializable`是不够
的。要维持单例的保证，需要声明所有的实例字段为`transient`并提供一个readResolve方法(条例89)。否则，每次一个序列化
的实例反序列化时，都会创建一个新实例，以我们的这个例子来说，会出现假的Elvis。为了防止这种情况发生，将`readResolve`
方法添加到`Elvis`类中:
```java
//readResolve方法用来保存单例属性
private Object readResolve() {
   // 返回真正的Elvis,让垃圾回收器处理Elvis的模仿者
   return INSTANCE;
}
```

实现单例的第三种方式是声明一个单元素的枚举
```java
//Enum单例，首选方法
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {}
}
```
　　这种方式类似于公共字段方式，但更简洁，无偿提供序列化机制，并提供了防止多重实例化的保证，即使面对复杂的序列化或反射攻击。
这种方式可能感觉有点不自然，但是单元素枚举类型通常是实现单例的最佳方式。  
　　注意，如果你的单例必须扩展除了Enum外的超类，那么你不能使用这种方式来实现单例（尽管你可以声明一个枚举来实现接口）
