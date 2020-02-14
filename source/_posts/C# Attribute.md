---
title: 'C# Attribute'
date: 2020-02-09 19:24:51
tags:
- Attribute
---

记录[C#特性](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/attributes/)学习过程。

##### 使用举例

```C#
[Serializable]  //表示类可以序列化 
public class SampleClass
{        
}
```

```C#
 //https://docs.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.inattribute?view=netframework-4.8
static void MethodA([In][Out] ref double x) { }
static void MethodB([Out,In] double x) {}
static void MethodB([Out] double x) {}

//DllImport使用 https://www.cnblogs.com/xumingming/archive/2008/10/10/1308248.html
[System.Runtime.InteropServices.DllImport("user32.dll")]
extern static void SampleMethod();

[Conditional("DEBUG"), Conditional("TEST1")] //指示编译器，该特性可以指定多次
void TraceMethod()
{            
}
```

按照约定，特性名称以“Attribute”结尾。 不过，在代码中使用特性时，无需指定特性后缀。

##### 显示标识特性、指定特性目标

```C#
[type: Serializable]  //表示类可以序列化 
public class SampleClass
{        
}
[method: Conditional("DEBUG"), Conditional("TEST1")] //目标是方法。 指示编译器，该特性可以指定多次
void TraceMethod()
{            
}
// default: applies to method
[ValidatedContract]
int Method1() { return 0; }

// applies to method
[method: ValidatedContract]
int Method2() { return 0; }

// applies to return value
[return: ValidatedContract]
int Method3() { return 0; }

```

自定义特性

```C#
//限制特性目标类型 AttributeTargets
//AllowMultiple 是否允许多次使用该特性
[System.AttributeUsage(System.AttributeTargets.Class | System.AttributeTargets.Struct, AllowMultiple = true)]
public class AuthorAttribute : System.Attribute
{
    public string name;
    public double version;

    public AuthorAttribute(string name)
    {
        this.name = name;
        version = 1.0;
    }
}

[Author("P. Ackerman", version = 1.1)]
[Author("P. Betty", version = 1.1)]
class SampleClass
{
}
```

##### AttributeUsage

AttributeUsage是应用到自定义特性的特性。它有好几个参数，AttributeTargets：自定义特性可以有哪些特性目标，默认是全部；AllowMultiple：自定义的特性是否能在同一个特性目标上使用多次；Inherited:自定义特性是否可以有派生类继承或者重写成员继承。

```C#
namespace ConsoleApp2
{
    class Program
    {
        static void Main(string[] args)
        {
            BaseClass baseClass = new BaseClass();
            DerivedClass derivedClass = new DerivedClass();
            var baseAttributes = baseClass.GetType().GetCustomAttributes();
            var derivedAttributes = derivedClass.GetType().GetCustomAttributes();            
            Console.WriteLine("BaseClass Attributes:");
            foreach (var item in baseAttributes)
            {
                Console.WriteLine(item);
            }

            Console.WriteLine("DerivedClass Attributes:");
            foreach (var item in derivedAttributes)
            {
                Console.WriteLine(item);
            }           
        }
    }

    [AttributeUsage(AttributeTargets.Class, Inherited = false)]
    class FirstAttribute : Attribute { }

    [AttributeUsage(AttributeTargets.Class)] //Inherited 默认值为true
    class SecondAttribute : Attribute { }

    [AttributeUsage(AttributeTargets.Class, AllowMultiple = true)]
    class ThirdAttribute : Attribute { }

    [AttributeUsage(AttributeTargets.All)]
    class FourthAttribute : Attribute { }

    [First, Second]
    class BaseClass
    {
        [field:Fourth]
        [property:Fourth]   
        public string name { get; set; }       
    }

    [Third, Third]
    class DerivedClass : BaseClass {
     
    }
}
```

##### 通过反射访问特性

对类型使用自定义特性，如果任何时候都不访问该特性信息，那特性就是多余的。可以通过反射获取自定义特性信息，主要方法是`System.Attribute.GetCustomAttributes`。

```C#

namespace ConsoleApp2
{
    class Program
    {
        static void Main(string[] args)
        {
            TestAuthor.Test();
        }
    }

    public class TestAuthor
    {
        public static void Test()
        {
            PrintAuthorInfo(typeof(FirstClass));
            PrintAuthorInfo(typeof(SecondClass));
        }

        private static void PrintAuthorInfo(Type t)
        {
            System.Console.WriteLine("Author information for {0}", t);
            
            System.Attribute[] attrs = System.Attribute.GetCustomAttributes(t);  
            foreach (System.Attribute attr in attrs)
            {
                if (attr is AuthorAttribute author)
                {
                    System.Console.WriteLine("{0}, version {1:f}", author.GetName(), author.version);
                }
            }
        }
    }

    //限制特性目标类型 AttributeTargets
    //AllowMultiple 是否允许多次使用该特性
    [System.AttributeUsage(System.AttributeTargets.Class | System.AttributeTargets.Struct, AllowMultiple = true)]
    public class AuthorAttribute : System.Attribute
    {
        string name;
        public double version;

        public AuthorAttribute(string name)
        {
            this.name = name;
            version = 1.0;
        }

        public string GetName()
        {
            return name;
        }
    }

    [Author("P. Ackerman", version = 1.1)]
    [Author("P. Betty", version = 1.2)]
    class FirstClass
    {
    }

    [Author("P. Betty", version = 1.22)]
    class SecondClass
    {
    }

```

##### 通用属性

ObsoleteAttribute,标记不再使用的元素。使用该特性时，最好传入参数提示。该特性不能被继承。

```C#
[System.Obsolete("use class B")] //使用该方法，给予警告，但是不报错
class A
{
    public void Method() { }
}
class B
{
     //第二个参数error如果true，使用该方法则编译报错
    [System.Obsolete("use NewMethod", true)] 
    public void OldMethod() { }
    public void NewMethod() { }
}
```

ConditionalAttribute  使得方法是否执行依赖于预处理标识符。自定义的预处理标识符：项目->属性->生成->条件编译和符号。也可以使用#define预处理指令。

```C#
namespace ConsoleApp4
{
    class Program
    {
        static void Main(string[] args)
        {           
            DoIfAorB(); 
            DoIfA(); //条件编译符号 有A和B才执行
        }

        [Conditional("A"), Conditional("B")]
        static void DoIfAorB()
        {
            Console.WriteLine("DoIfAorB");
        }

        [Conditional("A")]
        static void DoIfA()
        {
            DoIfAandB();
        }

        [Conditional("B")]
        static void DoIfAandB()
        {
            Console.WriteLine("DoIfA AND B");
            // Code to execute when both A and B are defined...  
        }
    }   
}
```

ConditionalAttribute通常与DEBUG标识符一起使用。代替 #if DEBUG #endif ...

```C#
namespace WindowsFormsApp1
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
            CheckMethod();
        }

        [Conditional("DEBUG")]
        public void CheckMethod()
        {
            MessageBox.Show("debug");
        }
    }
}
```

调用方特性

CallerMemberName、CallerFilePath、CallerLineNumber ，都只能应用于参数，且参数必须有默认值。如果调用方没有传入值，则参数值是运行时得到的调用方信息。

```C#

namespace ConsoleApp4
{
    class Program
    {
        static void Main(string[] args)
        {
            DoProcessing();
        }

        public static void DoProcessing()
        {
            TraceMessage("Something happened.");
            TraceMessage("Something happened11.",sourceFilePath:"...path");
        }

        public static void TraceMessage(string message,
                [System.Runtime.CompilerServices.CallerMemberName] string memberName = "",
                [System.Runtime.CompilerServices.CallerFilePath] string sourceFilePath = "",
                [System.Runtime.CompilerServices.CallerLineNumber] int sourceLineNumber = 0)
        {
            System.Diagnostics.Trace.WriteLine("message: " + message);
            System.Diagnostics.Trace.WriteLine("member name: " + memberName);
            System.Diagnostics.Trace.WriteLine("source file path: " + sourceFilePath);
            System.Diagnostics.Trace.WriteLine("source line number: " + sourceLineNumber);
        }       
    }
}

输出结果:
message: Something happened.
member name: DoProcessing
source file path: C:\Users\bibi\Desktop\代码\异步\ConsoleApp4\Program.cs
source line number: 16
message: Something happened11.
member name: DoProcessing
source file path: ...path
source line number: 17

```

##### 程序集特性

 AssemblyInfo.cs 文件放着程序集全局特性。分三种：唯一标识特性（例如：AssemblyVersionAttribute）、公司或产品信息特性（例如AssemblyCompanyAttribute）、程序集功能特性(例如：AssemblyTitleAttribute)。	



