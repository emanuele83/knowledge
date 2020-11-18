# [ThreadStatic] Attribute
Indicates that the value of a static field is unique for each thread.
```c#
public class Example
{
   [ThreadStatic] static double previous = 0.0;
   [ThreadStatic] static double sum = 0.0;
   [ThreadStatic] static int calls = 0;
   [ThreadStatic] static bool abnormal;
}

```
A static field marked with ThreadStaticAttribute is not shared between threads. Each executing thread has a separate instance of the field, and independently sets and gets values for that field. If the field is accessed on a different thread, it will contain a different value.

Note that in addition to applying the ThreadStaticAttribute attribute to a field, you must also define it as a static field (in C#) or a Shared field (in Visual Basic).

Resource: [https://docs.microsoft.com/en-us/dotnet/api/system.threadstaticattribute?view=net-5.0]

# Custom ICollection Object
```c#
public class CustomCollection : ICollection<T>
{
   private readonly ICollection<t> _items;

   public CustomCollection()
   {
      _items = new Collection<T>();
   }
}
```
N.B. Use Intellisense to create interface missing methods **using _items for implementation**