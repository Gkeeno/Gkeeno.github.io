# BLOG TODO:
- [x] 计算精度丢失
- [ ] C#中的闭包
- [ ] C#中的析构函数 + GC回收(超过MaxGeneration回收的机制)
```csharp
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("gc:"+ GC.MaxGeneration);
        for (int i = 0; i < 1e4; i++)   // 这里创造过多的对象会导致回收
        {
            var a = new DisposableClass("class:"+i);
            if(int.TryParse(Math.Log10(i).ToString(), out int b)) 
                Console.WriteLine(a.Name);
        }
    }
    
}

class DisposableClass : IDisposable
{
    public string Name { get; private set; } = "DisposableClass";
    public DisposableClass(string name = "DisposableClass")
    {
        Name = name;
    }
    ~DisposableClass()
    {
        Console.WriteLine(Name + "has been Free.");
    }
    public void Dispose()
    {
        Console.WriteLine(Name + " has been Disposed.");
    } 
}
```