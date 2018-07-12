---
layout: post
title:  "Java 枚举"
date:   2017-01-25 15:00:09 -0200
description: Java SE.
permalink: /java/
---


# 枚举类型是个啥

首先定义一个枚举[Season.class]
{% highlight java %}
    public enum Season {
        SPRING,
        SUMMER,
        AUTUNM,
        WINTER;
    }
{% endhighlight %}

使用反编译工具对上面的Season.class进行反编译，反编译内容如下
{% highlight java %}
    public final class Season extends Enum {
        public static Season[] values(){
            return(Season[])$VALUES.clone();
        }
        
        public static Season valueOf(String s) {
            return (Season)Enum.valueOf(Season,s);
        }
        
        //构造函数必须private
        private Season(String s,int i) {
            super(s,i);
        }
        
        public static final Season SPRING;
        public static final Season SUMMER;
        public static final Season AUTUMN;
        public static final Season WINTER;
        private static final Season ENUM$VALUES[];
        
        static {
            SPRING = new Season("SPRING",0);
            SUMMER = new Season("SUMMER",1);
            AUTUMN = new Season("AUTUMN",2);
            WINTER = new Season("WINTER",3);
            $VALUES[] =(new Season[]{SPRING,SUMMER,AUTUMN,WINTER})
        }
    }
{% endhighlight %}

  通过查看反编译后内容，一眼就看到了枚举的本质，它就是一个继承了[Enum](https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html)的类。
  所以虽然我们在定义枚举的时候，使用的是enum修饰符，而不是class，但是最终经过jvm，还是变成了一个类。
  

# 枚举中所谓的常量（Constants）是个啥
先看java官网原话

> 引用：<br/>The body of an enum declaration may contain enum constants. An enum constant defines an instance of the enum type.

这段话什么意思呢？逐句来讲解下<Br/>
第一句：看文章开始处，我们定义的枚举类型Season，它包含了 [SPRING, SUMMER,AUTUNM,WINTER](),这四个就是constants。<br/>
第二句：看我们反编译后的内容中static语句块和上面变量的定义。我们发现我们在枚举Season中定义的[SPRING, SUMMER,AUTUNM,WINTER]()最后都变成了Season的实例变量了。所以这句话的意思就是说，你在枚举中定义的每个常量，实际上最终会成为这个枚举类的一个实例。
看反编译文件发现，[SPRING, SUMMER,AUTUNM,WINTER]()各自都通过new Season实例化了。而枚举中定义的这个四个值，变成了构造方法中的第一个参数的值了。<br/>

# 枚举的用法

##   枚举中自定义方法

###     自定义构造方法
{% highlight java %}
   
    public enum Color {
    	RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLOW("黄色", 4);
    	
    	// 成员变量
    	private String name;
    	private int index;
    	
    	// 构造方法
    	private Color(String name, int index) {
    		this.name = name;
    		this.index = index;
    	}
    	
    	 //get，set方法省略	
    }

{% endhighlight %}

代码中发现，此时[RED,GREEN,BLANK,YELLOW]()都跟着一对大括号，而且里面还有参数。这与Season中的[SPRING,SUMMER,AUTUMN,WINTER]()的写法不同。
这是因为这里我们写了一个构造方法，如果有点懵，没关系，继续看下面反编译后的内容。
{% highlight java %}
   
    public final class Color extends Enum {
        
        private Color(String s, int i, String name, int index)
        {
            super(s, i);
            this.name = name;
            this.index = index;
        } 
        
        public static final Color RED;
        public static final Color GREEN;
        public static final Color BLANK;
        public static final Color YELLOW;
        private String name;
        private int index;
        private static final Color ENUM$VALUES[];
        
        static 
        {
            RED   = new Color("RED",    0, "红色", 1);
            GREEN = new Color("GREEN",  1, "绿色", 2);
            BLANK = new Color("BLANK",  2, "白色", 3);
            YELLO = new Color("YELLOW", 3, "黄色", 4);
            ENUM$VALUES = (new Color[] { RED, GREEN, BLANK, YELLOW });
        }
        
        //get，set方法省略
    }
{% endhighlight %}
观察static静态块中的内容，是不是看出点意思了？我们在定义Color时，[RED,GREEN,BLANK,YELLOW]()后面大括号中的参数会在实例化的时候使用。
为什么可以这么用呢，我们再来套用官方文档

> 引用：The body of an enum declaration may contain enum constants. An enum constant defines an instance of the enum type.

它的意思是：定义枚举的body中可以包含一些constants（常量）。一个contant定义了一个枚举类型的实例。<br/>
所以，是实例，就会有构造方法，因为我们在Color中自定义了构造方法，所以[RED,GREEN,BLANK,YELLOW]()就必须按照构造方法的要求来写。构造方法有参数，就需要带着大括号写参数，没有参数就不过那个写大括号。记住就行。

>切记：不要把枚举中的[RED,GREEN,BLANK,YELLOW]()当成是字符常量。它不是字符串，它的最终心态是一个类的对象。把它当成对象一切就看通了


###     自定义普通方法
{% highlight java %}
    
    public enum Color {
       	
       	RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLOW("黄色", 4);
       	
       	// 成员变量
       	private String name;
       	private int index;
       	
       	// 构造方法
       	private Color(String name, int index) {
       		this.name = name;
       		this.index = index;
       	}	
       	
       	// 普通方法
        public String getName(int index) {
            for (Color c : Color.values()) {
                if (c.getIndex() == index) {
                    return c.name;
                }
            }
            return null;
        }
        
        //get，set方法省略

    }
   
{% endhighlight %}

明白了上面说的构造方法，应该也就很容易明白普通方法的定义了。其实就是对一个类定义了一个方法，然后通过实例调用即可。只不过我们在写代码时，是写在枚举中，但最后还是会被jvm转成一个类。将枚举中的方法，变成了类的方法。
下面通过代码调用一下，看看结果
{% highlight java %}

    public static void main(String[] args){
		
		Color color = Color.RED;
		
		System.out.println(color.ordinal());// 0
		System.out.println(color.index);// 1
		System.out.println(color.name); // 红色。因为使用了自定义的构造方法，使用RED("红色",1)定义了RED。
	}
{% endhighlight %}

### 定义抽象方法

{% highlight java %}

    public enum Color {
    
        RED("红色", 1){
            @Override
            public void print() {
                System.out.println("我是红色");
            }
        }, 
        GREEN("绿色", 2){
            @Override
            public void print() {
                System.out.println("我是绿色");
            }
        }, 
        BLANK("白色", 3){
            @Override
            public void print() {
                System.out.println("我是白色");
            }
        }, 
        YELLOW("黄色", 4){
            @Override
            public void print() {
                System.out.println("我是黄色");
            }
        };
    
        // 成员变量
        private String name;
        private int index;
    
        // 构造方法
        private Color(String name, int index) {
            this.name = name;
            this.index = index;
        }
    
        // 普通方法
        public String getName(int index) {
            for (Color c : Color.values()) {
                if (c.getIndex() == index) {
                    return c.name;
                }
            }
            return null;
        }
    
        // get set 方法省略
        
        
        //抽象方法
        public abstract void print();
    
    }
{% endhighlight %}

代码中发现，当在枚举中声明了一个抽象方法后，那么在写[RED,GREEN,BLANK,YELLOW]()这几个常量的时候，就必须重写这个抽象方法。这就等同于在实例化对象时直接重写抽象方法。不懂得可以查看一下相关文档。
反正这里记住这个写法就行。

### 在接口中定义枚举类型
{% highlight java %}

    interface Food {
        enum Coffee implements Food {
            BLACK_COFFEE, DECAF_COFFEE, LATTE, CAPPUCCINO
        }
    
        enum Dessert implements Food {
            FRUIT, CAKE, GELATO
        }
        
        public static void main(String[] args) {
            Food.Coffee coffe = Food.Coffee.BLACK_COFFEE;
            coffe.name();
        }
    }
{% endhighlight %}

# 官网部分翻译


>原文： It is a compile-time error if an enum declaration has the modifier abstract or final.

定义一个枚举类型时，如果使用了absolute或者final修饰符，那么编译时会报错。

>原文： An enum declaration is implicitly final unless it contains at least one enum constant that has a class body

如果枚举中的常量含有body体时（如：	RED("红色", 1), GREEN("绿色", 2)），那么jvm在将枚举类型转化为类时不会添加final修饰符。
如果枚举中的常量不包含body体，那么jvm将枚举类型装换为类时，会添加final修饰符。具体看下面两组代码：

常量中不含有body的枚举定义代码
{% highlight java %}

    public enum SeasonWithoutBody {
        SPRING,SUMMER, AUTUMN, WINTER;
    }
{% endhighlight %}


反编译后内容

{% highlight java %}

   public final class SeasonWithoutBody extends Enum {
       
       public static final SeasonWithoutBody SPRING;
       public static final SeasonWithoutBody SUMMER;      
       public static final SeasonWithoutBody AUTUMN;      
       public static final SeasonWithoutBody WINTER;      
       private static final SeasonWithoutBody ENUM$VALUES[];
   
       static 
       {
           SPRING = new SeasonWithoutBody("SPRING", 0);
           SUMMER = new SeasonWithoutBody("SUMMER", 1);
           AUTUMN = new SeasonWithoutBody("AUTUMN", 2);
           WINTER = new SeasonWithoutBody("WINTER", 3);
           ENUM$VALUES = (new SeasonWithoutBody[] {
               SPRING, SUMMER, AUTUMN, WINTER
           });
       }
       
   }
   
{% endhighlight %}

注意看，反编译后的文件中public修饰符后面有一个final修饰符


再来看常量中含有body的枚举定义代码
{% highlight java %}
  public enum SeasonWithBody {
  	SPRING("春天", 1){
  		public void print() {
  			System.out.println("春天");
  		}
  	}, 
  	SUMMER("夏天", 2){
  		public void print() {
  			System.out.println("夏天");
  		}
  	}, 
  	AUTUMN("秋天", 3){
  		public void print() {
  			System.out.println("秋天");
  		}
  	},
  	WINTER("冬天", 4){
  		public void print() {
  			System.out.println("冬天");
  		}
  	};
  	
  	private String name1;
  	private int index1;
  
  	private SeasonWithBody(String name1, int index1) {
  		this.name1 = name1;
  		this.index1 = index1;
  	}
  }
{% endhighlight %}

反编译后的内容
{% highlight java %}
    public class SeasonWithBody extends Enum {
    
        private SeasonWithBody(String s, int i, String name1, int index1)
        {
            super(s, i);
            this.name1 = name1;
            this.index1 = index1;
        }
    
      
        public static final SeasonWithBody SPRING;
        public static final SeasonWithBody SUMMER;
        public static final SeasonWithBody AUTUMN;
        public static final SeasonWithBody WINTER;
        private String name1;
        private int index1;
        private static final SeasonWithBody ENUM$VALUES[];
    
       //其他部分省略，此处只关注类定义上的是否有final修饰符
    }

{% endhighlight %}


# 总结

本文中我们通过原码和反编译后的文件对照进行了说明，从中不难发现，枚举类型其实就是一个类。所谓定义一个枚举类型，就是告诉jvm让把它生成一个继承了Enum类的子类
是类就会有自己的api，这个可以到java官网API中找到Enum类，进行查看。

枚举中也可以自定义方法（构造方法，抽象方法，普通方法）

定义枚举类型时不要使用abstract 或者 final修饰符

