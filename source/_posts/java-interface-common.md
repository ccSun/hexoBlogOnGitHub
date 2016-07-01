title: Java-Interface-Marker
categories:
  - Java
tags:
  - Interface
date: 2015-11-01 20:03:17
---
来学习下接口Serializable，Cloneable。

## 一、Serializable

### 1. 特殊关键字

1. static ： static类型变量不能被序列化；
2. transient ： 修饰的变量不能被序列化；

### 2. 序列化、反序列化实例

用ObjectInputStream和ObjectOutputStream来操作相应的InputStream和OutputStream。

```
public class Test implements Serializable{

    public Test() {
        b ++;
    }

    public int a = 1;
    public static int b = 2;
    public transient int c = 3;

    public static void main(String[] t){

        try {
            ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("Test.txt"));
            os.writeObject(new Test());
            os.flush();
            os.close();

            new Test(); // b values 4;

            ObjectInputStream in = new ObjectInputStream(new FileInputStream("Test.txt"));
            Test test = (Test) in.readObject();
            in.close();
            System.out.println("Test:" + test.a); // output:1
            System.out.println("Test:" + test.b); // output:4,但是如果序列化和反序列化在不同的机器上，反序列化值可能为0
            System.out.println("Test:" + test.c); // output:0
        } catch (ClassNotFoundException e) {
                e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
}
```

## 二、 Cloneable

实现对象的copy，继承此接口后，要override对象父类的Object.clone()方法实现copy。

### 1. 浅克隆、深克隆

1. 浅克隆： 直接使用父类的clone方法，回在堆中copy一份对象实例，但是对象内部的成员的实例不会被copy；
2. 深克隆： override父类的clone方法，实现成员对象实例的copy；

### 2. 实例

```
public class Test implements Cloneable{

    public User user = new User();

    @Override
    protected Object clone() throws CloneNotSupportedException {

        // 浅克隆
        return super.clone();

        // 深克隆
		// Test test = (Test) super.clone();
		//  test.user = (User) test.user.clone();
		//  return test;
    }

    public static void main(String[] t){

        try {
            Test test1 = new Test();
            Test test2 = test1;
            Test test3 = (Test) test1.clone();
            
            Test test4 = new Test();
            System.out.println(test1.equals(test4)); // 默认的equals比较的是堆里的对象实例是不是同一个，String是override的equals比较的是char数组

            System.out.println(test1 == test2); // true
            System.out.println(test1.equals(test2)); // true

            System.out.println(test1.user == test2.user); // true
            System.out.println(test1.user.equals(test2.user)); // true

            System.out.println(test1 == test3); // false
            System.out.println(test1.equals(test3)); // false

            System.out.println(test1.user == test3.user); // 浅克隆：true  深克隆：false
            System.out.println(test1.user.equals(test3.user)); // 浅克隆：true  深克隆：false

        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
    }

    class User implements Cloneable{
        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }

}
```



