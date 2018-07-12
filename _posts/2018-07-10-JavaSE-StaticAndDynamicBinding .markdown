---
layout: post
title:  "Java SE 静态和动态绑定"
date:   2018-07-10 00:00:09 -0200
description: Java SE 静态和动态绑定
permalink: /java/
---


Java中有两种类型的绑定（binding）：
- 静态绑定（static binding）：在编译时进行
- 动态绑定（dynamic binding）：在执行时进行

那么什么是绑定呢？通过方法名对方法体的调用就是绑定。如果还有点晕，看下面的代码：
{% highlight java %}

    class Human{
        public int hight;
        public int weight;
        
        public void m (){
            System.out.println("Human");
        }
    }
    
    class Boy extends Human{
        public String boyName;
        
        public void m(){
            System.out.println("boy");
        }
        
        public static void main(String[] args) {
            Human h = new Boy();
            h.m(); //boy
        }
    }

{% endhighlight %}

代码中Human和Boy有个一个相同的方法名为m。但是相同的m方法各自有不同的方法体（花括号中的内容不同）。<br/>
当执行h.m()时，调用哪个方法体呢？这就是绑定。是不是秒懂吧：）



下面我们就开始讲解[静态绑定]()和[动态绑定]()

## 静态绑定
在编译时，就能确定方法名应该调用哪个具体的方法体。这就是静态绑定。<br/>
方法如果有private，static，final修饰符，那么就是静态绑定。<br/>
这些修饰符修饰的方法代码编译时期就会被绑定（如上面代码，编译时就可以确定调用m方法，最终是执行Human中的还是Boy中的）


这是为什么呢？这是因为被这些修饰符修饰的方法都不能被重写。所以在编译阶段就可以确定具体用的哪个方法体（子类和父类哪个类中方法）

下面在来看一个例子（代码如下）：
    假如有2各类Human和Boy，两个类都拥有相同的方法wolk()，并且这两个方法都是static，这就意味着这个方法不能被重写，因为尽管
     Human obj = new Boy();但是最后还是会调用Human的walk()方法。因为obj是Human类型（父类）。所以，当绑定（调用）一个static，final，private
     修饰的方法时，编译器在编译时就能决定调用的是哪个类中的方法。
     
     
{% highlight java %}

    class Human{
       public static void walk()
       {
           System.out.println("Human walks");
       }
    }
    
    class Boy extends Human{
       public static void walk(){
           System.out.println("Boy walks");
       }
       public static void main( String args[]) {
           /* Reference is of Human type and object is
            * Boy type
            */
           Human obj = new Boy();
           /* Reference is of HUman type and object is
            * of Human type.
            */
           Human obj2 = new Human();
           obj.walk();
           obj2.walk();
       }
    }  
{% endhighlight %}
    
Output：
    
{% highlight java %}
    Human walks
    Human walks
{% endhighlight %}  

## 动态绑定

当编译器无法再编译时确定调用/绑定哪个类的方法，那么就被称作动态绑定。
方法重写是动态绑定的一个很好的例子。
当子类重写父类的方法时，对象的类型决定了调用哪个类的方法（也就是说主要看=号右边new的是哪个类）。这个类型是在运行时才被决定的，因此也被称为动态绑定

动态绑定的例子：
同样还是Human和Boy两个类，不同的是，这两个类中不在使用static，private或者final修饰符修饰方法，
在重写的情况下，通过对象类型在运行时确定对重写方法的调用，从而发生后期绑定。代码如下：

{% highlight java %}

    class Human{
       //Overridden Method
       public void walk()
       {
           System.out.println("Human walks");
       }
    }
    class Demo extends Human{
       //Overriding Method
       public void walk(){
           System.out.println("Boy walks");
       }
       public static void main( String args[]) {
           /* Reference is of Human type and object is
            * Boy type
            */
           Human obj = new Demo();
           /* Reference is of HUman type and object is
            * of Human type.
            */
           Human obj2 = new Human();
           obj.walk();
           obj2.walk();
       }
    }
{% endhighlight %}

output:

{% highlight java %}
    Boy walks
    Human walks
{% endhighlight %}

从output中我们会发现，结果与上面静态绑定的结果不同。   在这个例子中最终在运行时确定将Boy类赋给obj，所有Boy的方法会被调用。

## 静态绑定和动态绑定的区别
1.静态绑定发生在编译时，动态绑定发生在运行时
2.私有、静态和最终方法的绑定总是在编译时发生，因为无法重写这些方法。当进行了方法重写时并且将子类赋值个了一个父类的引用，那么具体绑定哪个类是在运行时来确定的。
3.重载是静态绑定，重写是动态绑定。

## 总结
1.方法前面如果有static，private，final之一，那么便是静态绑定，此种情况下，等号左边的引用是什么类型，那么就会调用这个类的方法。

https://beginnersbook.com/2013/04/java-static-dynamic-binding/