## 第1章 基本语言要素

### 建议1：正确操作字符串

```
String str1="str1"+9;
String str2="str2"+9.ToString();
```

第二行效率高于第一行，减少了装箱操作

（在自己编写的代码中，应当尽可能地避免编写不必要地装箱代码）

① 避免过多装箱

② 避免分配额外的内存空间

### 建议2：使用默认转型方法

1、使用类型的转换运算符

2、使用类型内置的Parse、TryParse，或者如ToString、ToDouble和ToDateTime等方法

3、使用帮助类提供的方法

4、使用CLR支持的转型

### 建议3：区别对待强制转型与as和is

如果类型之间都上溯到某个共同的基类，那么根据此基类进行的转型（即基类转型为子类本身）应该使用as。子类与子类之间的转型，则应该提供转换操作符，以便进行强制转型。

**as 操作符永远不会抛出异常，如果类型不匹配或转型源对象为null，那么转型之后的值也为null**

```
static void DoWithSomeType(object obj)
{
   SecondType secondType=obj as SecondType;
   if(secondType!=null)
   {
       //省略
   }
}

```

**as 不能操作基元类型**

涉及基元类型的算法，就需要is转型前的类型判断，以避免转型失败。

```
static void DoWithSomeType(object obj)
{
   if(obj is SecondType)
   {
    SecondType secondType=obj as SecondType;
    //省略
   }
}
```

**没有上一版效率高，进行了两次类型检测**

### 建议4：TryParse比Parse好

能用TryParse就用TryParse函数

### 建议5：使用int?来确保值类型也可以为null

```
Nullable<int> i=null;
也可为：
int? i=null;

例如：Nullable<Int32> 值范围：-2147483648-2147483647,再加一个null值

int? i=123;
int j= i ?? 0;//如果i的HasValue为true，则将i的value赋值给j；否则，就给j赋值为0
```

### 建议6：区别readonly和const的使用方法

本质区别：

const 是一个编译期常量，readonly 是一个运行时常量。

const 只能修饰基元类型、枚举类型或字符串类型，readonly没有限制。

### 建议7：将0值作为枚举的默认值

### 建议8：避免给枚举类型的元素提供显式的值

枚举元素允许设定重复的值

### 建议9：习惯重载运算符

```
Salary mikeIncome =new Salary(){RMB=22};
Salary roseIncome=new Salary(){RMB=33};
Salary familiIncome=mikeIncome+roseIncome;

class Salary
{
   public int RMB{get;set;}
   public static Salary operator +(Salary s1,Salary s2)
   {
   s2.RMB+=s1.RMB;
   return s2;
   }
}
```

### 建议10：创建对象时需要考虑是否实现比较器

### 建议11：区别对待==和Equals

值相等性、引用相等性，==和Equals皆可重载

我们要定义”值相等性“，应该仅仅去重载Equals方法，同时让"=="表示"引用相等性"

### 建议12：重写Equals时也要重写GetHashCode

### 建议13：为类型输出格式化字符串

### 建议14：正确实现浅拷贝和深拷贝

拷贝：为对象创建副本

浅拷贝：值类型字段完全拷贝，副本中修改不影响源对象；引用类型在副本中是引用类型的引用，对引用类型的修改会影响到源对象本身。

深拷贝：值类型、引用类型均会被重新创建并赋值，不会影响到源对象本身。

### 建议15：使用dynamic来简化反射实现

var类型，一旦被编译，编译期会自动匹配var变量的实际类型，并用实际类型来替换该变量的声明

dynamic类型，被编译后为object类型，（编译器对dynamic类型进行特殊处理，让它在编译期间不进行任何的类型检查，而是将类型检查放到了运行期）

**始终用dynamic优化反射实现**

