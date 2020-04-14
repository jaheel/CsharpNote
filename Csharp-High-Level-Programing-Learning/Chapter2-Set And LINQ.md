## 第2章 集合和LINQ

### 建议16：元素数量可变的情况下不应使用数组

```
int [] iArr={0,1,2,3,4,5,6};
ArrayList arrayListInt=ArrayList.Adapter(iArr);//将数组转变为ArrayList
arrayListInt.Add(7);
List<int> listInt=iArr.ToList<int>();//将数组转变为List<T>
listInt.Add(7);
```



### 建议17：多数情况下使用foreach进行循环遍历

### 建议18：foreach不能代替for

foreach(迭代器)（不支持集合增删操作）

for(索引器)

### 建议19：使用更有效的对象和集合初始化

对象初始化：

```
class Program
{
  static void Main(string[] args)
  {
     Person person=new Person(){Name="Mike",Age=20};
  }
}

class Person
{
   public string Name{get;set;}
   public int Age{get;set;}
}
```

集合初始化：

```
List<Person> personList=new List<Person>()
{
   new Person() {Name="Rose",Age=19},
   mike,null
};
```

### 建议20：使用泛型集合代替非泛型集合

例如：

```
List<int> intList=new List<int>();
intList.Add(1);
intList.Add(2);
foreach(var item int intList)
{
   Console.WriteLine(item.ToString());
}
```

### 建议21：选择正确的集合

如果集合的数目固定且不涉及转型，使用数组效率高，否则就使用List<T>。

### 建议22：确保集合的线程安全

**多个线程上添加或删除元素时，线程之间必须保持同步**

通过加锁实现线程的互斥运行

### 建议23：避免将List\<T>作为自定义集合类的基类

​        需扩展的泛型接口通常是IEnumerable<T>和ICollection<T>，前者规范了集合类的迭代功能，后者规范了一个集合通常会有的操作。

### 建议24： 迭代器应该是只读的

### 建议25：谨慎集合属性的可写操作

### 建议26：使用匿名类型存储LINQ查询结果

匿名类型特性：

> 既支持简单类型也支持复杂类型
>
> > 简单类型必须是一个非空初始值，复杂类型则是一个以new开头的初始化项

> 只读属性，没有属性设置器，一旦被初始化不可更改

> 如果两个匿名类型的属性值相同，那么就认为两个匿名类型相等

> 匿名类型可以在循环中用作初始化器

> 匿名类型支持智能感知

### 建议27：在查询中使用Lambda表达式

### 建议28：理解延迟求值和主动求值之间的区别

延迟求值（lazy evaluation)、主动求值(eager evaluation)

```
List<int> list = new List<int>() { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
            var temp1 = from c in list where c > 5 select c;
            var temp2 = (from c in list where c > 5 select c).ToList<int>();
            list[0] = 11;
            Console.Write("temp1: ");
            foreach (var item in temp1)
            {
                Console.Write(item.ToString() + " ");
            }
            Console.Write("\ntemp2: ");
            foreach (var item in temp2)
            {
                Console.Write(item.ToString() + " ");
            }
            
            结果：
            temp1: 11 6 7 8 9
            temp2: 6 7 8 9
```

*查询调用ToList、ToArray等方法，将会使其立即执行*

### 建议29：区别LINQ查询中的IEnumerable\<T>和IQueryable\<T>

​       本地数据源用IEnumerable\<T>，远程数据源用IQueryable\<T>

​        IEnumerable\<T>查询的逻辑可以直接用我们所定义的方法，而IQueryable\<T>则不能使用自定义的方法，它必须先生成表达式树，查询由LINQ to SQL引擎处理。

### 建议30：使用LINQ取代集合中的比较器和迭代器

```
List<Salary> companySalary = new List<Salary>()
                {
                    new Salary() { Name = "Mike", BaseSalary = 3000, Bonus = 1000 },
                    new Salary() { Name = "Rose", BaseSalary = 2000, Bonus = 4000 },
                    new Salary() { Name = "Jeffry", BaseSalary = 1000, Bonus = 6000 },
                    new Salary() { Name = "Steve", BaseSalary = 4000, Bonus = 3000 }
                };
            Console.WriteLine("默认排序：");
            foreach (Salary item in companySalary)
            {
                Console.WriteLine(string.Format("Name:{0} \tBaseSalary:{1} \tBonus:{2}", item.Name, item.BaseSalary, item.Bonus));
            }
            Console.WriteLine("BaseSalary排序：");
            var orderByBaseSalary = from s in companySalary orderby s.BaseSalary select s;
            foreach (Salary item in orderByBaseSalary)
            {
                Console.WriteLine(string.Format("Name:{0} \tBaseSalary:{1} \tBonus:{2}", item.Name, item.BaseSalary, item.Bonus));
            }
            Console.WriteLine("Bonus排序：");
            var orderByBonus = from s in companySalary orderby s.Bonus select s;
            foreach (Salary item in orderByBonus)
            {
                Console.WriteLine(string.Format("Name:{0} \tBaseSalary:{1} \tBonus:{2}", item.Name, item.BaseSalary, item.Bonus));
            }
```

*LINQ功能的实现本身就是借助于FCL泛型集合的比较器、迭代器、索引器的，LINQ相当于封装了这些功能，让我们使用起来更加方便*

### 建议31：在LINQ查询中避免不必要的迭代

```
 
 List<Person> list = new List<Person>()
            {
                new Person(){ Name = "Mike", Age = 20 },
                new Person(){ Name = "Mike", Age = 30 },
                new Person(){ Name = "Rose", Age = 25 },
                new Person(){ Name = "Steve", Age = 30 },
                new Person(){ Name = "Jessica", Age = 20 }
            };
 
 
 MyList list = new MyList();
            var temp = (from c in list where c.Age == 20 select c).ToList();
            Console.WriteLine(list.IteratedNum.ToString());
            list.IteratedNum = 0;
            var temp2 = (from c in list where c.Age >= 20 select c).First();
            Console.WriteLine(list.IteratedNum.ToString());
       
       代码输出为：
       5
       1
```

第二次查询仅仅迭代1次是因为20正好放在List的首位。First方法实际完成的工作是：搜索到满足条件的第一个元素，就从集合中返回。如果没有符合条件的元素，它也会遍历整个集合。

```
MyList list = new MyList();
            var temp = (from c in list select c).Take(2).ToList();
            Console.WriteLine(list.IteratedNum.ToString());
            list.IteratedNum = 0;
            var temp2 = (from c in list where c.Name == "Mike" select c).ToList();
            Console.WriteLine(list.IteratedNum.ToString());
            
          代码输出为：
          2
          5
```

Take方法接收一个整型参数，然后为我们返回该参数指定的元素个数。与First一样，满足条件后，会从当前的迭代过程中直接返回。

**使用First和Take方法，能给应用带来高效性，避免无效迭代**