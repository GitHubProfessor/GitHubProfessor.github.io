---
layout: post
title:  "Java SE 静态和动态绑定"
date:   2018-07-10 00:00:09 -0200
description: Java SE 静态和动态绑定
permalink: /java/
---


Java中有两种类型的绑定（binding）：<br/>
- 静态绑定（static binding）：在编译时进行<br/>
- 动态绑定（dynamic binding）：在执行时进行

那么什么是绑定呢？就是一个方法的调用与方法所在的类(方法主体)关联起来。<br/>
说的再直白一些，就是子类集成父类，并重写了其中的方法，在实例化对象后，调用对象方法的时候，调用的是父类的还是子类的方法。
这个确定的过程就是**绑定**。那么是在什么时候确定下的呢？一种是在编译时就可以确定，一种是在运行时才能确定的。
这也就是所谓的**静态绑定**和**动态绑定**。如果还有点晕，看下面的代码对应理解上面的话：

{% highlight java %}

    class Human{
        public int hight;
        public int weight;
        
        public void speak (){
            System.out.println("Human");
        }
    }
    
    class Boy extends Human{
        public String boyName;
        
        public void speak(){
            System.out.println("boy");
        }
        
        public static void main(String[] args) {
            Human h = new Boy();
            h.speak(); //boy
        }
    }

{% endhighlight %}


代码中子类Boy继承了父类Human，并重写了speak方法，那么这时候当实例化时（Human h = new Boy();）
就需要jvm进行绑定（时动态绑定还是静态绑定看下面内容）。来决定当执行h.speak()方法时，是调用Human的还是Boy的。<br/>
这就是绑定。是不是秒懂吧：）

知道了什么是绑定，那么何为静态绑定，何为动态绑定呢？
> 注意：绑定只是针对方法的。属性不存在绑定的概念（可以在父类和子类写一个相同的属性进行试验。）下文会给出例子。

下面我们就开始讲解[静态绑定]()和[动态绑定]()

## 静态绑定

   在编译时，就能确定对象(h)调用的方法(h.speak()),应该调用父类(Human)的方法()speak())还是子类(Boy)的方法(speak())。这就是**静态绑定**。<br/>
   <br/>
   那么问题来了，什么条件下在编译时就能确定呢？<br/>
   答：如果方法名前有修饰符**private**，**static**，**final**，那么在编译时就能确定调用的是父类(Human)还是子类(Boy)的方法speak()。<br/>
       这是为什么呢？这是因为被这些修饰符修饰的方法都不能被重写。所以在编译阶段就可以确定具体用的哪个方法体（子类和父类哪个类中方法）

   下面在来看一个例子：
       假如有2各类Human和Boy，两个类都拥有相同的方法walk()，这两个方法都用static修饰，这就意味着这个方法不能被重写，所以就会在编译时就会确定
       Human obj = new Boy()后，执行obj.walk()方法时，调用的是Human的walk()方法。因为obj是Human类型（父类）。
       
   代码如下：
     
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

当编译器无法在编译时确定调用/绑定哪个类的方法，那么就被称作动态绑定。
方法重写是动态绑定的一个很好的例子。当子类重写父类的方法时， 只有在运行时，才能确定对象(h)调用的方法(h.speak()),应该调用父类(Human)的方法()speak())还是子类(Boy)的方法(speak())。

对象的类型决定了调用哪个类的方法（也就是说主要看=号右边new的是哪个类）。这个类型是在运行时才被决定的，所以方法重载属于动态绑定

再举一个动态绑定的例子：
同样还是Human和Boy两个类，不同的是，这两个类中不在使用static，private或者final修饰符修饰方法，
在方法重写的情况下，通过对象类型在运行时确定对重写方法的调用，从而发生后期绑定。代码如下：

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
1.方法前面如果有static，private，final之一，那么便是静态绑定，此种情况下，等号左边的引用是什么类型，那么就会调用这个类的方法。<br/>
2.方法重写是动态绑定的一个体现<br/>
3.属性不会被绑定。例子如下：<br/>

{% highlight java %}

    class Human {
        public int x = 0;
        public static void walk() {
            System.out.println("Human walks");
        }
    
        public void talk() {
            System.out.println("Human talk");
        }
    }
    
    class Boy extends Human {
        public int x =1;
        
        public static void walk() {
            System.out.println("Boy walks");
        }
        
        public void talk() {
            System.out.println("Human talk");
        }
    
        public static void main(String args[]) {
           
            Human obj = new Boy();
            Human obj2 = new Human();
            
            System.out.println(obj.x); // 0
            System.out.println(obj2.x);// 0
        }
    }
{% endhighlight %}
 
https://beginnersbook.com/2013/04/java-static-dynamic-binding/
