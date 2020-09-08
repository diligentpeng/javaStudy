# 内部类
java中四中修饰符：

|属性 | public >| protected >| (default) >| private|
|:---|:---|:---|:---|:---|
|同一个类|Y|Y|Y|Y|
|同一个包(包括子类和非子类)|Y|Y|Y|NO|
|不同包子类|Y|Y|NO|NO|
|不同包非子类|Y|NO|NO|NO|


 ## 一 内部类分类：
 
   * 1：成员内部类
   * 2：局部内部类（包含匿名内部类）

### 1.1 成员内部类（定义在另一个类里面）
  * 1：格式：
```
  修饰符 class 外部类名称{  
               修饰符 class 内部类名称{  
                    //成员内部类的内容  
               }  
               //外部类内容  
          }
  ```
  
  **注意：内部类用外部类，随意访问（private都能访问），但是外部类用内部类，需要通过内部类的对象来访问**  
  
  ```
    例子
    public class Outer {
        private int num =10;
        class inter{
            public void show(){
                System.out.println(num);
            }
        }
     }
    class demo2{
        public static void main(String[] args) {
            Outer.inter obj = new Outer().new inter();
            obj.show();
        }
    }
```

  * 2：内部类的使用 
>   a；间接方式，在外部类的方法当中（new内部类的对象），使用内部类，然后在main中只是调用外部类的方法  
>   b：直接方式，格式：  外部类名称.内部类名 对象名 = new 外部类名称（）.new 内部类名称（）;此时，该对象就可以访问内部类了  
>  c：在内部类中使用与外部类重名成员变量，使用自己的可以直接使用或者this.变量名,使用外部类的Outer.this.变量名，不重名则可以直接使用外部的成员变量。

###  1.2局部内部类（定义在方法内部，只有所属方法才能使用它，出了这个方法就不能使用了）
* 1：格式
```
   修饰符 class 外部类名称{
          修饰符 返回值类型 外部方法名（参数列表）{
                 修饰符 class 内部类名称{
                      //成员内部类的内容
                 }
                 内部类名称 obj = new 内部类名称（）；
                 //obj就可以使用内部类了
            }
            //外部类内容
    }
```
* 2:针对不同类注意事项

> public > protected > (default) > private  
> a：外部类：public ，（default不写修饰符）  
> b：成员内部类：public，protected，（default），private  
> c：局部内部类，什么都不能写，但不同于（default），因为除了该方法外，什么地方都不能使用局部内部类

 3：局部内部类的final问题
 
 **局部内部类，如果希望访问所在方法的局部变量，那么这个局部变量必须是【有效final的】**  
 从java 8+开始，只要局部变量事实不变，那么final关键字可以省略 
 
 ```
 例子
 public class Outer {
    private int num =10;
    public void method(){
        int num2 = 20;
        //num2 = 30; 加上这句，就会报错
        class inter{
            public void show(){
                System.out.println(num);
                System.out.println(num2);
            }
        }
    }
}
```

## **1.3 匿名内部类**

**如果接口的实现类（或者父类的子类），只需使用唯一的一次，那么这种情况下就可以省略该类的定义，从而使用【匿名内部类】**

### 1：定义格式
```
父类名称 对象名 = new 父类名称（）{  //第一种多态
        //覆盖重写所有抽象方法
}; //此时分号不可省略
new 父类名称（）{  //第二种：直接通过new的对象使用该子类对象的方法
        //覆盖重写所有抽象方法
}.方法();

接口名称 对象名 = new 接口名称（）{
        //覆盖重写所有抽象方法
}; //此时分号不可省略
对象名.方法;(重写所有抽象方法)
```
### 2：解析“new 接口名{...}”

> a：new 代表创建对象的动作  
> b：接口名称就是匿名内部类需要实现的接口  
> c:{...}才是匿名内部类的内容  

### 3：注意事项
> * a：匿名对象类在创建对象时，只能使用唯一一次。如果想多次创建对象，就需要使用接口实现类了  
> * b：注意匿名对象和匿名对象类的区别  

# Lamda表达式
![Lamda表达式](https://github.com/diligentpeng/javaStudy/blob/master/images/lamda.PNG)

## 一：Lambda表达的标准格式：
```
    由三部分组成：a：一些参数     b：一个箭头     c:一段代码
    格式：
        （参数列表）-> {一些重写方法的代码}；
    解释说明格式：
        a：（）：接口中的抽象方法的参数列表，没有参数，就为空，有参数就写出参数，多个参数使用逗号隔开
        b： -> ：传递的意思，把参数传递给方法体
        c：{ } ：重写接口抽象方法的方法体
```

例子1：
```
    //定义一个接口
    interface Cook{
        //接口中有一个抽象方法
        void makeFood();
    }
    public class LambdaTest {
        public static void main(String[] args) {
            //调用invokCook方法，参数是Cook接口，传递Cook接口的匿名内部类对象
            invokeCook(new Cook() {
                @Override
                public void makeFood() {
                    System.out.println("吃饭");
                }
            });
            //使用Lambda表达式，简化匿名内部类的书写
            invokeCook(()->{
                System.out.println("lambda吃饭");
            });
        }
        //定义一个方法，参数传递Cook接口，方法内部调用Cook接口中的方法makeFood
        public static void invokeCook (Cook cook){
            cook.makeFood();
        }
    }
```

例子2：
```
    import java.util.Arrays;
    import java.util.Comparator;

    class Person0 {
        private String name;
        private int age;

        //有参和无参构造方法
        public Person0() {
        }

        public Person0(String name, int age) {
            this.name = name;
            this.age = age;
        }

        //get方法


        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        @Override
        public String toString() {
            return "Person0{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
    public class LambdaTest {
        public static void main(String[] args) {
            //创建数组存储多个Person对象
            Person0[] arr = {new Person0("小明", 20),
                    new Person0("小红", 19),
                    new Person0("小亮", 21)
            };
            //对数组中的Person对象使用Arrays的sort方法通过年龄进行升序（前-后）
            Arrays.sort(arr, new Comparator<Person0>() {
                @Override
                public int compare(Person0 o1, Person0 o2) {
                    return o1.getAge() - o2.getAge();
                }
            });
            //遍历数组
            for (Person0 p : arr) {
                System.out.println(p);
            }//Person0{name='小红', age=19} Person0{name='小明', age=20} Person0{name='小亮', age=21}

            //使用Lambda简化对象
            Arrays.sort(arr,(Person0 o1,Person0 o2)->{
                return o2.getAge()-o1.getAge();
            });
            //遍历数组
            for (Person0 p : arr) {
                System.out.println(p);
            }//Person0{name='小亮', age=21}  Person0{name='小明', age=20} Person0{name='小红', age=19}
        }
    }
```

##  二：Lambda省略格式（可推导即可省略）
```
 凡是根据上下文推导出来的内容，都可以省略书写
 可以省略的内容：
    1：（参数列表）：括号中的参数列表的数据类型，可以省略不写
    2：（参数列表）：括号中的参数如果只有一个，那么类型和（）都可以省略
    3： { 代码体 } ：如果{ }中的代码只有一行，无论是否有返回值，都可以省略（{},return,分号）
                     注意：如果要省略{ }，return，分号，它们就必须一起被省略，没有返回值就把
                           { }和分号一起省略
``

## 三：Lambda的使用前提：

* 1. 使用Lambda必须具有接口，且要求接口中有且仅有一个抽象方法。  
    无论是JDK内置的 Runnable 、 Comparator 接口还是自定义的接口，只有当接口中的抽象方法存在且唯一 时，才可以使用Lambda。  
* 2. 使用Lambda必须具有上下文推断。 也就是方法的参数或局部变量类型必须为Lambda对应的接口类型，才能使用Lambda作为该接口的实例。  
* 3：备注：有且仅有一个抽象方法的接口，称为“函数式接口”。  

