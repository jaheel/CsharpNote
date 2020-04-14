## Chapter 9 : Security Design

### 建议113 ：声明变量前考虑最大值

为运算加上**checked关键字**，可以在运算溢出的时候抛出一个异常，而不是让程序继续下去。

```c#
static void Main(string[] args)
        {
            ushort salary = 65534;
            checked
            {
                salary = (ushort)(salary + 1);
                Console.WriteLine(string.Format("第一次加薪，工资总数：{0}", salary));
                salary = (ushort)(salary + 1);
                Console.WriteLine(string.Format("第二次加薪，工资总数：{0}", salary));
            }
        }
```

此时会抛出System.OverflowException:算术运算导致溢出



### 建议114：MD5不再安全

### 建议115：通过HASH来验证文件是否被篡改

对文件求MD5值，可以用来验证文件是否被篡改

### 建议116：避免非对称算法加密文件

非对称加密算法：复杂，加/解密码速度慢，适用于数据量小的场合。

对称加密算法：效率高，系统开销小，适用于大数据量的加/解密。

### 建议117：使用SSL确保通信中的数据安全

### 建议118：使用SecureString保存密钥等机密字符串

每次使用System.String类中的方法之一时，或者使用此类型进行运算时（如赋值、拼接等），都要在内存中创建一个新的字符串对象，也就是为该对象分配新的空间。**带来两个问题：**

* 原来的字符串是不是还在内存当中？
* 如果在内存当中，那么机密数据（如密码）该如何保存才足够安全？

**System.Security.SecureString**，SecureString表示一个应保密的文本，它在初始化时就已经被加密了。



非托管代码可直接在内存中注销，托管代码使用无意义的文本（如：“xxxxx”）进行替换。

### 建议119：不要使用自己的加密算法

### 建议120：为程序集指定强名称

### 建议121：为应用程序设定运行权限

