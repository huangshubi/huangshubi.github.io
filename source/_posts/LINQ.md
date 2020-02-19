---

title: LINQ
date: 2020-02-18 12:07:04
tags:
---

记录[LINQ](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/)学习过程。

##### 概要

LINQ是一种“语言集成”的查询表达式，使用LINQ可以智能提示和进行类型检查。C#里可以编写的LINQ查询有SQL数据库、XML文档、ADO.NET数据集、支持IEnumerable和IEnumerable<T>的对象。使用LINQ，可以简单对数据源进行分组、排序、筛选。有一些第三方库也为Web服务和其他数据库提供了LINQ支持。

数组隐式支持了 IEnumerable<T> 接口的。

```C#
//尚未执行查询，等待遍历再查。 可以在查询表达式后面添加ToList或ToArray方法，使之立刻查询。
IEnumerable<int> scoreQuery =           
    from score in new int[] { 97, 92, 81, 60 }
	where score > 90
    select score;
```

在编译时，查询表达式根据 “C# 规范规则”转换成“标准查询运算符方法调用”。

在编写 LINQ 查询时最好使用查询语法，必要时可以使用方法调用。

##### 查询简介

要想查询，数据源必须在内存中。

```C#
//LINQ to XML 将 XML 文档加载到可查询的 XElement 类型中：
XElement contacts = XElement.Load(@"c:\myContactList.xml"); 
//LINQ to SQL ,可以手动设计对象关系映射或借助LINQ TO SQL工具
```

##### LINQ和泛型

LINQ引入泛型机制。泛型属于强类型，不必执行运行时类型转换，与Object相比，优势更多。	

编译器可以处理泛型类型声明，因此可以通过var关键字避免使用泛型语法。在LINQ里，是否显示指定类型并不重要时，例如LINQ分组查询之后的指定嵌套泛型类型，建议使用var。毕竟晦涩难懂的代码不太招人喜欢喔！

##### LINQ基本查询

筛选，排序

```C#
namespace ConsoleApp4
{
    class Program
    {
        static void Main(string[] args)
        {
            IEnumerable<Customer> customers = new List<Customer>() {
            new Customer(){Name="David M. Buss" , City="美国奧斯汀"},
            new Customer(){Name="David M. Buss 1" , City="美国奧斯汀"},
            new Customer(){Name="David M. Buss 2" , City="美国奧斯汀"},
            new Customer(){Name="阿尔弗雷德·阿德勒" , City="英国阿伯丁"},};

            var queryUSCustomers = 
                from customer in customers
                where customer.City == "美国奧斯汀"
                orderby customer.Name ascending  
                select customer;

            foreach(var customer in queryUSCustomers)
            {
                Console.WriteLine(customer.Name);
            }
        }
    }
    class Customer
    {
        public string Name { get; set; }
        public string City { get; set; }
    }
}

David M. Buss
David M. Buss 1
David M. Buss 2
```

分组

```C#
namespace ConsoleApp4
{
    class Program
    {
        static void Main(string[] args)
        {
            IEnumerable<Customer> customers = new List<Customer>() {
            new Customer(){Name="David M. Buss" , City="美国奧斯汀"},
            new Customer(){Name="David M. Buss 1" , City="美国奧斯汀"},
            new Customer(){Name="David M. Buss 2" , City="美国奧斯汀"},
            new Customer(){Name="阿尔弗雷德·阿德勒" , City="英国阿伯丁"},};

            // queryCustomersByCity is an IEnumerable<IGrouping<string, Customer>>
            var queryCustomersByCity =
                from cust in customers
                group cust by cust.City; //key 就是City

            // customerGroup is an IGrouping<string, Customer>
            //分组类型实现了IEnumerable接口
            foreach (var customerGroup in queryCustomersByCity)
            {
                Console.WriteLine(customerGroup.Key);
                foreach (Customer customer in customerGroup)
                {
                    Console.WriteLine($"    {customer.Name}");
                }
            }
        }
    }
    class Customer
    {
        public string Name { get; set; }
        public string City { get; set; }
    }
}
```

关联

```C#
namespace ConsoleApp4
{
    class Program
    {
        static void Main(string[] args)
        {
            IEnumerable<Customer> customers = new List<Customer>() {
            new Customer(){Name="David M. Buss" , City="美国奧斯汀"},
            new Customer(){Name="David M. Buss 1" , City="美国奧斯汀"},           
            new Customer(){Name="阿尔弗雷德·阿德勒" , City="英国阿伯丁"},};

            IEnumerable<City> cities = new List<City>() {
                new City(){Country="美国",CityName="美国奧斯汀"},
                new City(){Country="英国",CityName="英国阿伯丁"},
            };

            var innerJoinQuery =
                from cust in customers
                join city in cities on cust.City equals city.CityName
                select new { Country = city.Country, Customer = cust.Name };
            //匿名类型
          
            foreach (var item in innerJoinQuery)
            {
                Console.WriteLine($"{item.Country},{item.Customer}");               
            }
        }
    }
    class Customer
    {
        public string Name { get; set; }
        public string City { get; set; }
    }
    class City
    {
        public string Country { get; set; }
        public string CityName { get; set; }
    }
}
```

##### 使用LINQ进行数据转换

可以对原序列修改，可以在返回值创建新类型。

```C#
//Concat合并序列
var peopleInSeattle = (from student in students
                         where student.City == "Seattle"
                         select student.Last)
                  .Concat(from teacher in teachers  
                          where teacher.City == "Seattle"
                          select teacher.Last);
```

````C#
//内存中数据结构转换为xml
var studentsToXML = new XElement("Root",
                from student in students
                let scores = string.Join(",", student.Scores) //存储子表达式的结果
                select new XElement("student",
                           new XElement("First", student.First),
                           new XElement("Last", student.Last),
                             new XElement("Scores", GetScore(scores)) //调用C#方法
                        ) // end "student"
                    ); // end "Root"
````



