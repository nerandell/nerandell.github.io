---
layout: post
title: Ordered class members in python
date: 2015-10-16 12:32:18
summary: List all the members of a class in the order in which they were declared
comments: true
categories: python
thumbnail: python
tags:
 - python
 - collections
 - ordereddict
---

Recently I came across this problem where I had to list all the members of a class in the order in which they were declared. Normally, to list all the members of the class, one would use `dir` to get all the attributes of the object. Following example demonstrates the working of `dir` command:

{% highlight python %}

class TestClass:

	B = 2
	A = 1

	def set_method(self):
		print('method1')

	def get_method(self):
		print('method2')

if __name__ == '__main__':
	print(dir(TestClass))
	print(TestClass.__dict__)

# output

['A', 'B', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'get_method', 'set_method']
{'__dict__': <attribute '__dict__' of 'TestClass' objects>, '__weakref__': <attribute '__weakref__' of 'TestClass' objects>, 'set_method': <function TestClass.set_method at 0x102133bf8>, 'get_method': <function TestClass.get_method at 0x102133c80>, 'B': 2, 'A': 1, '__doc__': None, '__module__': '__main__'}

{% endhighlight %}

We can see that the class members are not listed in the order in which they are declared. I started googling for the answers and found a solution [here](http://stackoverflow.com/a/27113652/1465701). Sample output with this solution is:

{% highlight python %}
import collections

class OrderedClassMembers(type):
    @classmethod
    def __prepare__(self, name, bases):
        return collections.OrderedDict()

    def __new__(self, name, bases, classdict):
        classdict['__ordered__'] = [key for key in classdict.keys()
                if key not in ('__module__', '__qualname__')]
        return type.__new__(self, name, bases, classdict)


class TestClass(metaclass=OrderedClassMembers):

    B = 2
    A = 1

    def set_method(self):
        print('method1')

    def get_method(self):
        print('method2')

if __name__ == '__main__':
    print(TestClass.__ordered__)

# output
['B', 'A', 'set_method', 'get_method']

{% endhighlight %}
To understand how this works, we first have to understand what a metaclass is. Metaclasses are basically classes that are used to create classes. In python, class is an object and it is basically an instance of a metaclass called `type`. Let's look at some code:

{% highlight python %}
In [1]: class A:
   ...:     pass
   ...: 

In [2]: type(A)
Out[2]: type

In [3]: A.__class__
Out[3]: type

In [4]: isinstance(A, type)
Out[4]: True

{% endhighlight %}

So, what we did while using `OrderedClassMembers` was that we defined our own metaclass. We use the magic method called `__prepare__` to achieve our goal. When we declare a class in python, a namespace for that particular class in prepared using a `dict` object. However `dict` fails to preserve order of the keys for the namespace. However, using  `__prepare__` method which was introduced in Python 3 (check [PEP3115](https://www.python.org/dev/peps/pep-3115/)) we can define the structure that is used to create this namespace. Now when `__new__` runs, we get an `OrderedDict` instead of a normal `dict`. In this dictionary, we add another key called `__ordered__` which can than be used to get the desired result.
