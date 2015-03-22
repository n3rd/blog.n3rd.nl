---
layout: post
title: Exploring the C# dynamic type
tags: [.NET, C#, DLR, dynamic]
description:
---

Last week I learned something new and very powerful about the dynamic keyword in C#. I already was aware of the possibility of creating your own dynamic object by deriving from the DynamicObject class. Here is a short example.

{% highlight c# %}
class MyDynamicObject : DynamicObject  
{  
 Dictionary<string, object> _Values = new Dictionary<string, object>();  

 public override bool TryGetMember(GetMemberBinder binder, out object result)  
 {  
  if (_Values.ContainsKey(binder.Name))  
   result = _Values[binder.Name];  
  else  
   result = null;  

  return true;  
 }  

 public override bool TrySetMember(SetMemberBinder binder, object value)  
 {  
  if (!_Values.ContainsKey(binder.Name))  
   _Values.Add(binder.Name, value);  
  else  
   _Values[binder.Name] = value;  

   return true;  
 }  
}
{% endhighlight %}

Now you can create instances of MyDynamicObject and just set any member variable you want and access those as well, however you should be careful or runtime exceptions will occur.

{% highlight c# %}
dynamic test = new MyDynamicObject();  
test.FirstName = "Boaz";  

Console.WriteLine(test.FirstName);  

// RuntimeException:  
Console.WriteLine(test.LastName);  
{% endhighlight %}

You don't even need to create your own MyDynamicObject, but you can use System.Dynamic.ExpandoObject. It even implements IDictionary<string, object> so you can foreach over its members. [More information about the ExpandoObject.](http://blogs.msdn.com/b/csharpfaq/archive/2009/10/01/dynamic-in-c-4-0-introducing-the-expandoobject.aspx)

The above example with ExpandoObject:

{% highlight c# %}
dynamic exp = new ExpandoObject();  
exp.FirstName = "Boaz";  

Console.WriteLine(exp.FirstName);  
// RuntimeException:  
Console.WriteLine(exp.LastName);  
{% endhighlight %}

This works because the compiler converts the dynamic type to classes in Microsoft.CSharp.RuntimeBinder and System.Runtime.CompilerServices. These classes will invoke the DLR at runtime.

But what I didn't know is that you can't only do this for classes that inherit from DynamicObject, but for any object. A very simple example:

{% highlight c# %}
class Program  
{  
 static void Main(string[] args)  
 {  
  Console.WriteLine(Add(1, 2));  
  Console.WriteLine(Add("Bo", "az"));  
  Console.ReadKey();  
 }  

 static dynamic Add(dynamic a, dynamic b)  
 {  
  return a + b;  
 }  
}  
{% endhighlight %}

This is also known as Duck Typing, what means you identify an object by its members rather then by its Type or Interfaces it implements. Just like you most likely will identify a bird that walks, swims and quacks like a duck as a duck.

When you use the dynamic type the compiler will not check your properties, methods or even operators, but will leave it for the DLR to process it. This can cause very nasty runtime exceptions, so be careful just like you would when you would typecast an object.

[View the full example on gist.](https://gist.github.com/n3rd/5451555)
