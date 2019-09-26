---
title: java枚举
date: 2019-09-10 20:58:22
category:
- javaSE
tags:
- java
---
### JDK1.5之前的枚举
在JDK1.5之前是没有enum这个关键字的，那么那个时代是怎么实现枚举的呢？主要是通过私有化构造器，然后在类里面创建静态final的对象，在类的外面通过 ** 类名.对象名 ** 来使用枚举的，
<!-- more -->
如下：
```java
class Season{
    String name;
    String description;

    private Season(String name, String description) {
        this.name = name;
        this.description = description;
    }
    public static final Season SPRING = new Season("Spring", "春天");
    public static final Season SUMMER = new Season("Summer", "夏天");
    public static final Season AUTUMN = new Season("Autumn", "秋天");
    public static final Season WINTER = new Season("Winter", "冬天");

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    @Override
    public String toString() {
        return "Season{" +
                "name='" + name + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}

// 测试方法
@Test
public void test1(){
    Season Spring = Season.SPRING;
    Season Summer = Season.SUMMER;
    Season Autumn = Season.AUTUMN;
    Season Winter = Season.WINTER;
    System.out.println(Spring);
    System.out.println(Spring.getName()+" "+Spring.getDescription());
    /**
     * 打印结果：
     * Season{name='Spring', description='春天'}
     * Spring 春天
     */
}
```
### JDK1.5之后的枚举
在JDK1.5之后，出现了enum关键字，它的作用就是简化前面使用枚举的繁琐，解放程序员的双手的：
```java
enum Season1{
    SPRING("Spring","春天"),
    SUMMER("Summer", "夏天"),
    AUTUMN("Autumn", "秋天"),
    WINTER("Winter", "冬天");

    String name;
    String description;

    Season1(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    @Override
    public String toString() {
        return "Season{" +
                "name='" + name + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}

@Test
public void test2(){
    Season1 Spring = Season1.SPRING;
    Season1 Summer = Season1.SUMMER;
    System.out.println(Spring);
    Season1[] season1s = Season1.values();
    for(Season1 season1: season1s){
        System.out.println(season1 + " ----------"+season1.getName()+"--"+season1.getDescription());
    }
    /**
     * 打印结果：
     * Season{name='Spring', description='春天'}
     * Season{name='Spring', description='春天'} ----------Spring--春天
     * Season{name='Summer', description='夏天'} ----------Summer--夏天
     * Season{name='Autumn', description='秋天'} ----------Autumn--秋天
     * Season{name='Winter', description='冬天'} ----------Winter--冬天
     */
}
```
使用enum之后只是有几个点要注意一下的：
1. 对象名紧跟在类名后面
2. 对象名后面的参数其实就是传向构造器中的参数

在我看来，其实enum就是一个类，只是这个类定义的时候与其他类不同而已，它的使用方法也略微有点不同，它有一些自己特有的方法，比如values(),valueOf(String str)等。
同样，作为一个类，它也能实现接口，比如：
```java
interface EnumInterface{
    void show();
}
enum Season1 implements EnumInterface{
    SPRING("Spring","春天"){
        @Override
        public void show(){
            System.out.println("Spring show()");
        }
    },
    SUMMER("Summer", "夏天"){
        @Override
        public void show(){
            System.out.println("Summer show()");
        }
    },
    AUTUMN("Autumn", "秋天"){
        @Override
        public void show(){
            System.out.println("Autumn show()");
        }
    },
    WINTER("Winter", "冬天"){
        @Override
        public void show(){
            System.out.println("Winter show()");
        }
    };

    String name;
    String description;

    Season1(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    @Override
    public String toString() {
        return "Season{" +
                "name='" + name + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}
@Test
public void test2(){
    Season1 Spring = Season1.SPRING;
    Season1 Summer = Season1.SUMMER;
    System.out.println(Spring);
    Season1[] season1s = Season1.values();
    for(Season1 season1: season1s){
        System.out.println(season1 + " ----------"+season1.getName()+"--"+season1.getDescription());
        season1.show();
    }
    /**
	 * 打印结果：
     * Season{name='Spring', description='春天'}
     * Season{name='Spring', description='春天'} ----------Spring--春天
     * Spring show()
     * Season{name='Summer', description='夏天'} ----------Summer--夏天
     * Summer show()
     * Season{name='Autumn', description='秋天'} ----------Autumn--秋天
     * Autumn show()
     * Season{name='Winter', description='冬天'} ----------Winter--冬天
     * Winter show()
     */
}
```
此时，在每个对象中重写show()方法，可以使得每个对象都具有不同的行为，倘若只是在类中重写show()方法，则所有的对象都只是具有一个相同的行为而已。

### 总结
枚举并非什么神奇的东西，它只不过是帮助我们简化了一些操作而已，比如我们之前写一些枚举状态之类的东西，可能是这样：
```java
class Status{
    public static final String NEW = "NEW";
    public static final String FINISHED = "FINISHED";
}
```
用了枚举之后就是这样的，在枚举类的外部还可以使用values()来获取一组Status的对象的数组，极大的方便了我们对状态的遍历。
```java
enum Status1{
    NEW("NEW"),
    FINISHED("FINISHED");

    private String str;
    private Status1(String str){
        this.str = str;
    }
}
```

