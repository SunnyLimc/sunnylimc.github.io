# First_post


# 概念

**内部类(inner class)** 是定义在一个**类**或**方法**中的类

而内部类中，也有几个分类：

- 非静态内部类（声明在类中的类）
- 静态内部类（声明在类中的静态类）
- 局部内部类（声明在方法中的类）
- 匿名内部类（声明在语句中的类）

这篇文章，就让我带大家了解这些内部类的特性，并浅谈它们的实现原理

<br>

***




# 非静态内部类

## 示例

> 非静态内部类的内容全部围绕示例展开，示例中的内部类为**非静态内部类**

```Java
class TestClass {

    public int username;
    
    private int password;

    class InnerClass {
        int status;
    }
}
```

## 实例化内部类

> 把内部类声明为public和默认的可见性是一样的，你可能会想看权限修饰符

### 外部类

```Java
class TestClass {
   ...
    public InnerClass instance() {
        return new InnerClass();
        // 等价的
        return this.new InnerClass();
    }
   ...
}
```

### 同包下其他类

必须使用**外部类的对象**才能对**非静态内部类**进行实例化

由于内部类对**同一个包下的其他类**是隐藏的，所以在使用时需先引用外部类，通过外部类来引用内部类，如`TestClass.InnerClass`

```Java
public static void main(String[] args) {
    TestClass testClass = new TestClass();
    TestClass.InnerClass inner = testClass.new InnerClass();
} 
```

### 非同包下其他类

除非内部类在被**声明为 public 的外部类**中，否则非同包下其他类无法访问

### 私有化内部类

```Java
class TestClass {
  ...
    private class InnerClass {
        int status;
    }
  ...
}
```

将内部类声明为**private**后，则只能从**外部类的方法**中访问内部类。因为我们无法从其他类中访问一个类的私有属性。

## 使用内部类访问外部对象

### 访问public属性

在实例化内部类时，编译器会为内部对象创建一个指向外部对象的常量隐式引用(变量)` TestClass this$0`

- 编译器会修改内部类的构造函数，且在实例化时自动把外部类的`this`指针传入内部类的构造函数，构造函数再为`this$0`这个隐式引用赋值

> 在正常的编码使用上，我们需要使用 `TestClass.this`，由编译器来帮我们完成转化为`this$0`的过程，具体可见下方示例

当内部对象需要访问外部对象的变量时，**如果内部类中没有声明与外部类同名的变量**，以此处的`username`为例，编译器会自动替换`username`为`TestClass.this.username`，从而让内部类能正确访问外部对象的变量

- 示例

```Java
class TestClass {
    public int username;
    
    class InnerClass {
        public void print() {
            System.out.println(username);
            // 等价的
            System.out.println(TestClass.this.username);
        }
        ...
     }
}
```

## 访问private属性

内部类与其他类不同的其中一个地方就是可以**直接访问**外部类的私有数据。但我们知道，用隐式引用`this$0`的方式肯定是没有足够的权限访问外部类的私有变量的

- 因为这只是拿到了外部类实例化的对象，但是除了使用反射外，我们不能直接访问对象的私有属性

所以 Java 使用了不一样的实现方法。以访问 TestClass 的私有变量 password 为例，若编译器检测到你访问的是对象的私有属性，编译器会自动为**外部类**添加一个静态的方法`int access$0(TestClass)`

- `int access$0(TestClass)`方法接受一个 TestClass 对象作为参数，并返回该对象的 password 属性，让内部类可以通过`TalkingClock.access$0(this$0)`来拿到外部类的 password 属性

这可能存在安全风险，因为这相当于**运行阶段**在外部类中直接加入了一个可以访问其私有属性的方法。

## 静态属性

### 静态方法

Java 没有对**非静态内部类**声明静态方法进行任何的限制

但如果声明的方法是静态方法，则其**只允许访问外部类的静态属性**

### 静态属性

**在JDK 16之前**，不允许在**非静态内部类**中声明**非常量的静态变量**

但在JDK 16及之后，就没有这个限制了

## 后记

如果你不需要对外部类对象的引用`(this$0)`，一般来说更推荐声明一个**静态内部类**

<br>

***


# 静态内部类

### 示例

```Java
class TestClass {
    static class InnerClass {
      ...
    }
} 
```

### 特性

- 静态内部类中没有静态属性的限制
- 除了没有对外部类对象的引用之外，其他特性与非静态内部类一致

### 其他

- 在接口中声明的内部类自动是静态内部类

<br>

***

<br>

# 局部内部类

Java允许在方法中声明**局部内部类**(方法内部类)，供方法本身进行使用。

### 实例化

```Java
class TestClass {
    public void instance() {
        class Inner {
           ...
        }
        Inner instance = new Inner();
    }
} 
```

### 特性

- 局部内部类对外部是完全隐藏的。除了方法本身外，没有任何人知道局部内部类的存在

- 局部内部类能直接访问**对象的內部属性**，不过在**访问方法的局部变量**时会有所限制

- 局部内部类声明时不能带有权限修饰符(`public`、`private`)

### 访问方法的局部变量

**局部内部类**可以直接读取**方法**中声明的局部变量。

- 实例

```Java
class TestClass {
    public void instance() {
        int[] a = {1, 2}; // wrong
        final int[] a = {1, 2}; // correct
        int test; // 演示如何读取局部变量
        
        class Inner {
            public void edit() {
                a[0] = 1; // 由于数组是对象，所以写值的操作被允许
            }
        }
        new Inner().edit();
    }
} 
```

这里的原理跟**非静态内部类**访问 public 属性非常相似，编译器会为内部类生成**与方法局部变量的对应的内部类属性**并**修改构造函数**，让方法的局部变量在实例化内部类时通过构造函数传递到内部类的属性中

- 就如同这里方法的`test`变量，编译器会为内部类生成一个`final int test` 属性，然后通过构造函数将方法的`test`变量传递到内部类的`final int test`属性中，从而允许内部类进行访问

<br>


那能否**写入**方法中声明的局部变量呢?

> 这里的**能否写入**其实与传入的**方法变量的类型有关**，由于传入之后内部类中生成的是**final 类型**的属性，我们不难得出以下结论：

- 如果传入的是**基本类型**，基本类型的**值**是属性引用的对象，故值不可变
- 而如果传递的是一个**对象的引用**，则只是**对象的引用**不可变，而对象的内部属性是可变的 (如数组、ArrayList等)

<br>

***


# 匿名内部类

匿名内部类可以在不为类指定名字的情况下，直接**继承**或**实现**一个**类**或**接口**，并直接生成其对象

```Java
new ExtendClassName(ConstructParam){
    // 匿名内部类的内容 
}.FunctionName(FunctionParam);

// 或

new InterfaceName(){ 
 // 匿名内部类的内容 
}.FunctionName(FunctionParam);
```

## 继承

简而言之，就是匿名继承一个类，并直接获得一个其继承后的匿名对象

由于匿名内部类没有类名，也自然不能有任何构造器。**构造器的参数会传递到父类(super)里面**

> 注意如果将对象保存并向上转型，会丢失匿名内部类的方法

- 实例 — 向**构造器传递参数**、**调用父类方法**、输出**重写方法**

```Java
class TestClass {
    public void instance() {  
     // 用匿名内部类重写Date类方法并调用其输出
     // 只作演示，没有实际作用
      System.out.println( new Date(114513L) {
          @Override
          public long getTime() {
              return super.getTime() + 1;
          }
          
      }.getTime() );
    }
} 
```



## 实现接口

简而言之，就是匿名实现一个接口，并直接获得一个其实现后的匿名对象

由于在接口中不能存在构造函数，如果用匿名内部类实现接口，就不能有任何构造参数

## 其他
尽管没有办法使用构造器，但是仍然可以在类中使用由`{ }`包裹的构造区块

<br>

***

<br>


# 参考文献

[Java 核心技术卷1 - 豆瓣](https://book.douban.com/subject/34898994/)

# 附

谢谢您的阅读。第一次写作，难免出现错漏，还请大家多多包涵。

关于文章有任何的问题，可以在后台留言哦！


