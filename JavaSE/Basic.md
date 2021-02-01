[TOC]
在线API：[API中文手册](https://www.matools.com/api/java8)或[菜鸟编程](https://www.runoob.com/manual/jdk1.6/java/lang/package-summary.html)
# 1. 基本程序设计
## 关键字和保留字
1. 关键字必须全部小写
2. 保留字：`const`和`goto`
## 基本数据类型
1. long的默认值是`0L`，float的默认值是`0.0f`。
2. 浮点型的计算会导致精度丢失，此时使用Math中的BigDecimal。
3. 计算机的值以二进制存储，有最小值-1=最大值（整型）
4. 局部变量需要手动初始化，其他变量可以自动初始化为默认值
## 强制类型转换
1. 把小数赋给一个整型变量会丢失小数点后的精度，编译不通过，需要强制类型转换
2. 格式：`(要转换的数据类型)变量/对象`
# 2. 数组
数组分为两个部分：声明和初始化
声明：`数据类型[] 数组名;`
new的过程中开辟了空间和初始化默认值

```java
public class Main{
    public static void main(String[] args) {
        int[] nums = new int[]{};
        System.out.println(nums);
        if( nums.length ==0) System.out.println(-2);
        if ( nums == null) System.out.println(-1);
        else System.out.println(0);

        int[] arr1 = new int[5];
        System.out.println(arr1[0]);
        String[] arr2 = new String[5];
        System.out.println(arr2[0]);
    }
}
```
结果：

```java
[I@1b6d3586
-2
0
0
null
```

1. 静态初始化
 `= new 数据类型[]{元素1,元素2,元素3...}`或 ` = {元素1,元素2,元素3...};`后一种仅能在声明时同时使用
2.  动态初始化
`数组名 = new 数据类型[数组长度];`
3. 对象数组
除基本数据类型外，Java其他所有类型都是引用数据类型，如String等，由于引用类型的默认值是null，若直接使用，会报空指针异常。相应的，对象数组的每个对象都要使用构造方法实例化。
4. 二维数组
 `数据类型[][] 变量名 = new 数据类型[m] [n];`
 `数据类型[][] 变量名 = new 数据类型[][]{{元素…},{元素…},{元素…}};`
 第二维可以不给出，若不给出，堆中还是null，注意空指针异常

```java
 		int[][] num = new int[2][];
        num[0]= new int[]{1, 2, 3};
        System.out.println(Arrays.toString(num[0]));
        System.out.println(Arrays.deepToString(num));
        num[1][1]=4;
        System.out.println(Arrays.toString(num));
```
结果：

```java
[1, 2, 3]
[[1, 2, 3], null]
Exception in thread "main" java.lang.NullPointerException
	at Test.main(Test.java:9)

Process finished with exit code 1
```
5. 打印
`Arrays.toString()`（一维）
`Arrays.deepToString(数组)`（多维）
# 3. 方法
## Java的方法传参是值传递
对基本数据类型，传的是值的拷贝；对引用数据类型，传的是引用的拷贝，对引用本身而不是引用的对象进行修改，不会做出实际改动。
```java
class TestIt
{
    public static void main ( String[] args )
    {
        int[] myArray = {1, 2, 3, 4, 5};
        ChangeIt.doIt( myArray );
        for(int j=0; j<myArray.length; j++)
            System.out.print( myArray[j] + " " );
    }
}
class ChangeIt
{
    static void doIt( int[] z ) 
    {
        z = null ;
    }
}
```
结果：

```java
1 2 3 4 5 
Process finished with exit code 0
```
## 可变参数
1. 可变参数用于形参列表中，并且只能出现在形参列表的最后
2. `数据类型...  变量名`
3. 调用可变参数的方法时，编译器为该可变参数隐含创建一个数组，在方法体中以数组的形式访问可变参数

```java
public static void main ( String[] args )
    {
        System.out.println(sum(1,2,3,4));
    }

    public static int sum(int... n){
        int sum = 0;
        for (int i = 0; i < n.length; i++) {
            sum += n[i];
        }
        return sum;
    }
```
## 方法的重载和重写
|              | 重载（overload） |            重写（override）            |
| :----------: | :--------------: | :------------------------------------: |
| 发生的类不同 |   发生在同类中   |            发生在子父类之间            |
|    方法名    |     必须相同     |                必须相同                |
|   参数列表   |     必须不同     |                必须相同                |
|  权限修饰符  |      不影响      |  重写的方法访问权限必须大于等于原方法  |
|     异常     |      不影响      |      重写的方法不能抛出更多的异常      |
|  返回值类型  |      不影响      | 重写的方法的返回值类型必须和原方法兼容 |
1. 被static、final、private修饰的父类方法无法被重写
# 4. 对象和类
类是对象的模板，对象是类的具体实例。
## static
1. static修饰成员，称之为静态成员，包括静态成员变量和静态成员方法。随着类加载完毕，静态成员就存在，并且能够使用了。静态成员领先于对象存在，不依赖于对象而存在。
2. 静态成员变量和方法被所有类对象共享，访问方式是`类名.方法名()`和`类名.成员名()`
3. 一个类中，静态方法无法直接调用非静态的方法和属性，也不能使用this，super关键字。一个非静态的方法，可以访问静态的成员。
## 代码块
~~局部代码块，~~ 构造代码块，静态代码块，~~同步代码块（多线程）~~ 
1. 构造代码块依赖于构造方法的执行而执行，每次new对象都会执行。先执行this调用的构造器（若有），后顺序执行成员变量和代码块的初始化语句，最后执行构造器
2. 静态代码块和静态成员一样，随着类加载而执行，用于给静态成员变量赋值，只执行一次
## 访问权限
1. 成员

|              | public | protected | 默认 | private |
| ------------ | ------ | --------- | ---- | ------- |
| 同一类中     | √      | √         | √    | √       |
| 同一包其他类 | √      | √         | √    |         |
| 不同包子类   | √      | √         |      |         |
| 不同包其他类 | √      |           |      |         |
2. 类
public：全包可见；default：包内可见
# 5. 继承
## 父类与子类
1. 定义：`public class 类名 extends 父类名{}`
2. 父类成员变量会被子类的同名成员变量隐藏，静态成员变量会被静态同名成员变量覆盖
3. `父类 对象名 = new 子类()`自动类型转换。同时调用成员变量时调用的是父类的，调用成员方法时调的是子类的。
常见的由此引发的令人困惑的点：
 

```java
        Collection collection = new ArrayList();
        Iterator it = collection.iterator();
        InputStream inputStream = new Socket().getInputStream();
```
因为我们都知道，接口（上面的Collection，Iterator），抽象类（Inputstream）都是不能实例化的，那为什么还能这么写。这是因为等号左边的是声明，实际赋给的对象是右边给的ArrayList类的对象、FileInputstream类的对象。
## final
1. final修饰的类无法被继承，final修饰的方法无法被覆盖。
2. final修饰普通成员变量，表示常量，创建对象以后，它们的值就不可改变了。因此赋值方式共有三种：显式初始化，构造代码块和构造器。
3. final修饰静态成员变量，表示全局常量，类加载完毕后，它们的值就不可改变了。赋值方式共有两种：显式初始化和静态代码块。静态常量的声明格式一般全部大写，用下划线隔开。
4. 对于基本数据类型而言，final修饰后，值不能改变。对于引用数据类型而言，final修饰后，引用的地址就不可变了，但引用对象的状态可以改变。
5. 常量只能被赋值一次，进行诸如i++等重新赋值行为都是不可以的。
## 多态
子类对象，通过父类对象提供的窗口，执行自己覆盖过的方法。父类的引用，指向不同的实际对象时，调用方法，结果不同。
1. 条件：继承和重写
2. final，static，private，构造方法无法多态

## 抽象类
 1. 抽象类：

```java
[访问权限修饰符] abstract class 类名{}
```
   抽象方法：

```java
 [权限修饰符] abstract 返回值类型 方法名();
```
2. 抽象类无法被实例化；抽象方法没有方法体，只有方法声明。
3. 如果一个类包含抽象方法，那么该类必须是抽象类。抽象类同时也可以有正常的方法和成员变量。
4. 抽象类的子类只有重写抽象类的所有抽象方法才能成为具体类，否则必须定义为抽象类。

## 接口
1.   `[访问权限修饰符] interface 接口名{}`
2. 继承：`class SubClass extends SuperClass implements InterfaceA{ }` 
3. 一个接口能继承另一个接口，接口的继承使用extends关键字，子接口继承父接口的方法。
4. 接口可以多继承，不同接口间用`,`隔开
5. 
| 编号 | **区别点** |                          **抽象类**                          |                 **接口**                  |
| :--: | :--------: | :----------------------------------------------------------: | :---------------------------------------: |
|  1   |    定义    |                       包含抽象方法的类                       |         抽象方法和全局常量的集合          |
|  2   |    组成    |           构造方法、抽象方法、普通方法、常量、变量           | 常量、抽象方法、(jdk8:默认方法、静态方法) |
|  3   |    使用    |                   子类继承抽象类(extends)                    |         子类实现接口(implements)          |
|  4   |    关系    |                    抽象类可以实现多个接口                    |  接口不能继承抽象类，但允许继承多个接口   |
|  5   |    对象    |                 不能创建对象，但是有构造方法                 |       不能创建对象，也没有构造方法        |
|  6   |    局限    |                      抽象类不能被多继承                      |       接口之间能多继承，能被多实现        |
|  7   |    思想    |                  作为模板或对共性抽象，is-a                  |     作为标准或对共性能力抽象，like-a      |
|  8   |    选择    | 如果抽象类和接口都可以使用的话，优先使用接口，因为避免单继承的局限 |   
## 内部类
