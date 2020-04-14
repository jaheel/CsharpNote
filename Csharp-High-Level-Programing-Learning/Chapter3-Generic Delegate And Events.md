## 第3章 泛型、委托和事件

### 建议32：总是优先考虑泛型

设计自己类型时，尽量让自己的类型成为泛型类

### 建议33：避免在泛型类型中声明静态成员

为T指定不同数据类型，MyList\<T>相应地也变成了不同地数据类型，在它们之间是不共享静态成员的。

* 非泛型类型中的泛型方法并不会在运行时的本地代码中生成不同的类型

### 建议34：为泛型参数设定约束

### 建议35：使用default为泛型类型变量指定初始值

* 值类型变量的默认初始值为0，引用类型变量的默认初始值为null

  ```
  public T Func<T>()
  {
      T t=default(T);
      return t;
  }
  ```

* 这样default以后，在运行时碰到的T是一个整型，那么运行时会为其赋值为0；碰到的是引用类型，则会为其赋值null。

### 建议36：使用FCL中的委托声明

FCL中存在三类委托声明：Action、Func、Predicate

Action表示接收0个或多个输入参数，执行一段代码，没有任何返回值

Func表示接受0个或多个输入参数，执行一段代码，带返回值

​       （Func<T>的用法：多个参数，前面的为委托方法的参数，最后一个参数，为委托方法的返回类型。）

Predicate表示定义一组条件并判断参数是否符合条件

```
class Program
    {
        delegate int AddHandler(int i, int j);
        delegate void PrintHandler(string msg);

        static void Main(string[] args)
        {
            //AddHandler add = Add;
            //PrintHandler print = Print;
            //print(add(1, 2).ToString());

            Func<int, int, int> add = Add;
            Action<string> print = Print;
            print(add(1, 2).ToString());

        }

        static int Add(int i, int j)
        {
            return i + j;
        }

        static void Print(string msg)
        {
            Console.WriteLine(msg);
        }

    }
```



### 建议37：使用Lambda表达式代替方法和匿名方法

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Tip37
{
    class Program
    {
        static void Main(string[] args)
        {
            //第一种写法
            //Func<int, int, int> add = new Func<int, int, int>(Add);
            //Action<string> print = new Action<string>(Print);
            //print(add(1, 2).ToString());

            //Func<int, int, int> add = new Func<int, int, int>(delegate(int i, int j)
            //{
            //    return i + j;
            //});
            
            //第二种写法：匿名委托
            //Action<string> print = new Action<string>(delegate(string msg)
            //{
            //    Console.WriteLine(msg);
            //});
            //print(add(1, 2).ToString());   

            //Func<int, int, int> add = delegate(int i, int j)
            //{
            //    return i + j;
            //};
            //Action<string> print = delegate(string msg)
            //{
            //    Console.WriteLine(msg);
            //};
            //print(add(1, 2).ToString());
            
            //第三种写法：Lambda表达式 
            Func<int, int, int> add = (i, j) =>
            {
                return i + j;
            };
            Action<string> print = (msg) =>
            {
                Console.WriteLine(msg);
            };
            print(add(1, 2).ToString());

            //第一种写法
            //return this.Find(new Predicate<Student>(delegate(Student target)
            //{
            //    if (target.Name == name)
            //    {
            //        return true;
            //    }
            //    else
            //    {
            //        return false;
            //    }
            //}));
            //第二种写法
            //return this.Find(new Predicate<Student>((target) =>
            //    {
            //        if (target.Name == name)
            //        {
            //            return true;
            //        }
            //        else
            //        {
            //            return false;
            //        }
            //    }));
            //第三种写法
            //return this.Find((target) =>
            //    {
            //        if (target.Name == name)
            //        {
            //            return true;
            //        }
            //        else
            //        {
            //            return false;
            //        }
            //    });
            //第四种写法
            //return this.Find( target => target.Name == name );

        }

        static int Add(int i, int j)
        {
            return i + j;
        }

        static void Print(string msg)
        {
            Console.WriteLine(msg);
        }

    }
}

```



**Lambda表达式操作符“=>”的左侧是方法的参数，右侧是方法体，本质是匿名方法。**



### 建议38：小心闭包中的陷阱

注意局部变量的添加

### 建议39：了解委托的本质

* 委托是方法指针

* 委托是一个类，当对其进行实例化的时候，要将引用方法作为它的构造方法的参数

###  建议40：使用event关键字为委托施加保护

### 建议41：实现标准的事件模型

### 建议42：使用泛型参数兼容泛型接口的不可变性

### 建议43：让接口中的泛型参数支持协变

```
interface ISalary<out T>
{
 void Pay();
}
```

### 建议44：理解委托中的协变

### 建议45：为泛型类型参数指定逆变

逆变是指方法的参数可以是委托或泛型接口的参数类型的基类

支持逆变的常用委托：

```
Func<in T,out TResult>
Predicate<in T>
```

常用泛型接口：

```
IComparer<in T>
```



  