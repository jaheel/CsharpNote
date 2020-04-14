## 第4章 资源管理和序列化

### 建议46：显式释放资源需继承接口IDisposable

* **托管资源**：由CLR管理分配和释放的资源，即从CLR里new出来的对象

* **非托管资源**：不受CLR管理的对象，如Windows内核对象，或者文件、数据库连接、套接字、COM对象等。

> 如果我们的类型使用到了非托管资源，或者需要显式地释放托管资源，就需要让类型继承接口IDisposable。

**如果类型需要显示释放资源，那么一定要继承IDispose接口**

```
using(SampleClass c1=new SampleClass())
{
   //省略
}
等价于：
SampleClass c1;
try
{
   c1=new SampleClass();
   //省略
}
finally
{
   c1.Dispose();
}

```

* 如果存在两个类型一致的对象：

```
using(SampleClass c1=new SampleClass(),c2=new SampleClass())
  {
   //省略
  }
```

* 如果两个类型不一致：

```
using(SampleClass c1=new SampleClass())
using(SampleAnothorClass c2=new SampleAnothorClass())
{
   //省略
}
```

### 建议47：即使提供了显式释放方法，也应该在终结器中提供隐式清理

```
 /// <summary>
        /// 实现IDisposable中的Dispose方法
        /// </summary>
        public void Dispose()
        {
            //必须为true
            Dispose(true);
            //通知垃圾回收机制不再调用终结器（析构器）
            GC.SuppressFinalize(this);
        }
```

### 建议48：Dispose方法应允许被多次调用

* 在Dispose模式中，应该始终为类型创建一个变量，用来表示对象是否已经Dispose过

### 建议49：在Dispose模式中应提取一个受保护的虚方法

如果不为类型提供受保护的虚方法，开发者设计子类的时候容易忽略父类的清理工作。

### 建议50：在Dispose模式中应区别对待托管资源和非托管资源

显式释放资源的无参Dispose方法中，调用参数true(**代码同时处理托管资源和非托管资源**)

```
public void Dispose()
{
   Dispose(true);
   //省略
}
```

供垃圾回收器调用的隐式清理资源的终结器中，调用参数false（**隐式清理时，只要处理非托管资源即可**）

```
~SampleClass()
{
   Dispose(false);
}
```

### 建议51：具有可释放字段的类型或拥有本机资源的类型应该是可释放的

### 建议52：及时释放资源

> 自动的垃圾回收器满足以下条件之一时才发生垃圾回收
>
> 1、系统具有低的物理内存
>
> 2、由托管堆上已分配的对象使用的内存超出了可接受的范围
>
> 3、调用GC.Collect方法。（垃圾回收器会调用它）



**改进前：**

```
private void buttonOpen_Click(object sender,EventArgs e)
{
    FileStream fileStream=new FileStream(@"c:\test.txt",FileMode.Open);
}
private void buttonGC_Click(object sender,EventArgs e)
{
     System.GC.Collect();
}
```

​       打开文件方法执行完毕后，局部变量fileStream在程序中没有任何地方引用，在下一次垃圾回收时被运行时标记为垃圾，但需满足自动垃圾回收条件以后才进行回收，就不知道fileStream什么时候才会被回收，在回收之前，它一直占用着内存，降低内存使用效率。

**改进后：**

```
private void buttonOpen_Click(object sender,EventArgs e)
{
    FileStream fileStream=null;
    try{
           fileStream=new FileStream(@"c:\test.txt",FileMode.Open);
    }
    finally
    {
        fileStream.Dispose();
    }
}


进一步简化：
using(FileStream=new FileStream(@"c:\test.txt",FileMode.Open))
{
 
}
//这段程序和上面使用try-finally块生成的IL代码完全一致
```

### 建议53：必要时应将不再使用的对象引用赋值为null

静态变量的回收需要先赋值为null，才能进行回收

**鉴于此，程序应尽量少用静态变量**

### 建议54：为无用字段标注不可序列化

> 序列化：把对象转变成流（相反的过程，称为反序列化）
>
> 以下场合需要用到这项技术，例如：
>
> 1、把对象保存到本地，在下次运行程序的时候，恢复这个对象
>
> 2、把对象传到网络中的另外一台终端上，然后在此终端还原这个对象
>
> 3、其他场合，如：把对象复制到系统的粘贴板中，然后Ctrl+V恢复这个对象

* 类型被添加Serializable属性后，默认所有的字段全部能够被序列化

### 建议55：利用定制特性减少可序列化的字段

特性(attribute)可以声明式地为代码中的目标元素添加注解。

空间调用：

```
using System.Runtime.Serialization.Formatters.Binary;
using System.Runtime.Serialization;
```

> OnDeserializedAttribute，当它应用于某方法时，会指定在对象反序列化后立即调用此方法。
>
> OnDeserializingAttribute，当它应用于某方法时，会指定在反序列化对象时调用此方法。
>
> OnSerializedAttribute，如果将对象图应用于某方法，则应指定在序列化该对象图后是否调用该方法。
>
> OnSerializingAttribute，当它应用于某个方法时，会指定在对象序列化前调用此方法
>
> NonSerialized，不可序列化

### 建议56：使用ISerializable接口更灵活地控制序列化过程

​       发现对象继承了ISerializable接口，那它就会忽略掉类型所有的序列化特性，转而调用类型的GetObjectData方法来构造一个SerializationInfo对象，方法内部负责向这个对象添加有需要序列化的字段。

```
[Serializable]
    public class Person : ISerializable
    {
        public string FirstName;
        public string LastName;
        public string ChineseName;

        public Person()
        {
        }

//序列化字段
        protected Person(SerializationInfo info, StreamingContext context)
        {
            FirstName = info.GetString("FirstName");
            LastName = info.GetString("LastName");
            ChineseName = string.Format("{0} {1}", LastName, FirstName);
        }


        void ISerializable.GetObjectData(SerializationInfo info, StreamingContext context)
        {
            info.AddValue("FirstName", FirstName);
            info.AddValue("LastName", LastName);
        }
    }
```

​      **ISerializable接口运用得当，在版本升级中，能处理类型因为字段变化而带来的问题**

### 建议57：实现ISerializable的子类型应负责父类的序列化

​       父类没有序列化，子类型实现了序列化，此时如果要调用父类型的元素，应在子类型的序列化方法中获取父类元素并将其序列化，方能获取正确的数据。