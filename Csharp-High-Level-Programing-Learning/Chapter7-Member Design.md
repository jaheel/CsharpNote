## 第七章 成员设计

### 建议90：不要为抽象类提供公开的构造方法

即使没有为抽象类指定构造方法，编译器也会为我们生成一个默认的protected构造方法。抽象类的构造方法不应该是public或internal的。

**抽象类设计的本意只能让子类继承**



### 建议91：可见字段应该重构为属性

字段和属性有本质的区别。区别是：**属性是方法**。

例如：

```c#
class Person
{
   public string Name{get;set;}
}
```

经过编译器编译后，针对属性Name，实际会生成一个静态字段和两个方法。如下所示：

```c#
[CompilerGenerated]
private string <Name>k_BackingField;

[CompilerGenerated]
public void set_Name(string value)
{
   this.<Name>k_BackingField=value;
}

[CompilerGenerated]
public string get_Name()
{
    return this.<Name>k_BackingField;
}
```

属性相比字段的优势：

* 可以为属性添加代码。属性是方法，可以在方法内对设置或获取属性的过程进行更多精细化的控制。如：可以为属性添加NameChanged事件支持等。
* 可以让属性支持线程安全。属性变线程安全，自身可实现；字段支持线程安全，只能靠调用者实现。
* 属性得到了VS编辑器的支持，还得到了实现自动属性的这种功能。
* 公开的字段也应该使用属性。



### 建议92：谨慎将数组或集合作为属性

如果只读属性应用于数组和集合，元素的内容和数量却仍旧可以随意更改。



### 建议93：构造方法应初始化主要属性和字段

类型的属性应该在构造方法调用完毕之前完成初始化工作。



### 建议94：区别对待override和new

+ 如果子类中的方法前面带有new关键字，则该方法被定义为独立于基类的方法。
+ 如果子类中的方法前面带有override关键字，则子类的对象将调用该方法，而不是调用基类方法。

### 建议95：避免在构造方法中调用虚成员

### 建议96：成员应优先考虑公开基类型或接口

### 建议97：优先考虑将基类型或接口作为参数传递

### 建议98：用params减少重复参数

如果方法参数数目不定，如下所示：

```C#
void Method1(string str,object a)
{

}

void Method2(string str,object a,object b)
{

}

void Method3(string str,object a,object b,object c)
{

}

```

可以合并成一个方法：

```c#
void Method(string str,params object[] args)
{

}
```

### 建议99：重写时不应使用子类参数

### 建议100：静态方法和实例方法没有区别

如果一个方法只跟类型本身有关系，那么它就应该被设计成静态方法，如果跟类型的实例对象有关系，那它就应该被设计成实例方法。

### 建议101：使用扩展方法，向现有类型“添加”方法

非扩展前，创建对象方法调用：

```c#
class Program
    {
        static void Main(string[] args)
        {
            Student student = new Student();
            Console.WriteLine(StudentConverter.GetSexString(student));
        }
    }

    public static class StudentConverter
    {
        public static string GetSexString(Student student)
        {
            return student.GetSex() == true ? "男" : "女";
        }
    }

    public class Student
    {
        public bool GetSex()
        {
            return false;
        }
    }
```

用更优美的形式调用Student类型的实例方法（即扩展方法）：

```c#
public class Student
    {
        public bool GetSex()
        {
            return false;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Student student = new Student();
            Console.WriteLine(student.GetSexString());
        }
    }

    public static class StudentExtension
    {
        public static string GetSexString(this Student student)
        {
            return student.GetSex() == true ? "男D" : "女?";
        }
    }
```

扩展方法有点：

* 可以扩展密封类型
* 可以扩展第三方程序集中的类型
* 扩展方法可以避免不必要的深度继承体系
* 扩展方法必须在静态类中，且该类不能是一个嵌套类
* 扩展方法必须是静态的
* 扩展方法的第一个参数必须是要扩展的类型，而且**必须加上this关键字**
* 不支持扩展属性、事件

