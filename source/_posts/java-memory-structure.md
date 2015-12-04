title: Java对象的内存结构
date: 2012-03-12 15:01:43
categories: dev
tags: Java
---

转载 - Java对象的内存结构

## 起因

Java对象的内存结构非常重要，但是很多人不清楚，至少我之前也不清楚。特转载一篇文章，我觉得讲得很好。

http://www.codeinstructions.com/2008/12/java-objects-memory-structure.html
SATURDAY, DECEMBER 13, 2008
## Java Objects Memory Structure

Update (December 18th, 2008): I've posted here an experimental library that implements Sizeof for Java.
One thing about Java that has always bothered me, given my C/C++ roots, is the lack of a way to figure out how much memory is used by an object. C++ features the sizeof operator, that lets you query the size of primitive types and also the size of objects of a given class. This operator in C and C++ is useful for pointer arithmetic, copying memory around, and IO, for example.
Java doesn't have a corresponding operator. In reality, Java doesn't need one. Size of primitive types in Java is defined in the language specification, whereas in C and C++ it depends on the platform. Java has its own IO infrastructure built around serialization. And both pointer arithmetic and bulk memory copy don't apply because Java doesn't have pointers.
But every Java developer at some point wondered how much memory is used by a Java object. The answer, it turns out, is not so simple.
The first distinction to be made is between shallow size and deep size. The shallow size of an object is the space occupied by the object alone, not taking into account size of other objects that it references. The deep size, on the other hand, takes into account the shallow size of the object, plus the deep size of each object referenced by this object, recursively. Most of the times you will be interested on knowing the deep size of an object, but, in order to know that, you need to know how to calculate the shallow size first, which is what I'm going to talk about here.
One complication is that runtime in memory structure of Java objects is not enforced by the virtual machine specification, which means that virtual machine providers can implement them as they please. The consequence is that you can write a class, and instances of that class in one VM can occupy a different amount of memory than instances of that same class when run in another VM. Most of the world, including myself, uses the Sun HotSpot virtual machine though, which simplifies things a lot. The remainder of the discussion will focus on the 32 bit Sun JVM. I will lay down a few 'rules that will help explain how the JVM organizes the objects' layout in memory.

### Memory layout of classes that have no instance attributes

In the Sun JVM, every object (except arrays) has a 2 words header. The first word contains the object's identity hash code plus some flags like lock state and age, and the second word contains a reference to the object's class. Also, any object is aligned to an 8 bytes granularity. This is the first rule or objects memory layout:
####  Rule 1: every object is aligned to an 8 bytes granularity.

Now we know that if we call new Object(), we will be using 8 bytes of the heap for the two header words and nothing else, since theObject class doesn't have any fields.

### Memory layout of classes that extend Object

After the 8 bytes of header, the class attributes follow. Attributes are always aligned in memory to their size. For instance, ints are aligned to a 4 byte granularity, and longs are aligned to an 8 byte granularity. There is a performance reason to do it this way: usually the cost to read a 4 bytes word from memory into a 4 bytes register of the processor is much cheaper if the word is aligned to a 4 bytes granularity.
In order to save some memory, the Sun VM doesn't lay out object's attributes in the same order they are declared. Instead, the attributes are organized in memory in the following order:
```
doubles and longs
ints and floats
shorts and chars
booleans and bytes
references
```

This scheme allows for a good optimization of memory usage. For example, imagine you declared the following class:
```
class MyClass {
    byte a;
    int c;
    boolean d;
    long e;
    Object f;
}
```

If the JVM didn't reorder the attributes, the object memory layout would be like this:
```
[HEADER:  8 bytes]  8
[a:       1 byte ]  9
[padding: 3 bytes] 12
[c:       4 bytes] 16
[d:       1 byte ] 17
[padding: 7 bytes] 24
[e:       8 bytes] 32
[f:       4 bytes] 36
[padding: 4 bytes] 40
```

Notice that 14 bytes would have been wasted with padding and the object would use 40 bytes of memory. By reordering the objects using the rules above, the in memory structure of the object becomes:
```
[HEADER:  8 bytes]  8
[e:       8 bytes] 16
[c:       4 bytes] 20
[a:       1 byte ] 21
[d:       1 byte ] 22
[padding: 2 bytes] 24
[f:       4 bytes] 28
[padding: 4 bytes] 32
```

This time, only 6 bytes are used for padding and the object uses only 32 bytes of memory.
So here is rule 2 of object memory layout:

#### Rule 2: class attributes are ordered like this: first longs and doubles; then ints and floats; then chars and shorts; then bytes and booleans, and last the references. The attributes are aligned to their own granularity.

Now we know how to calculate the memory used by any instance of a class that extends Object directly. One practical example is the java.lang.Boolean class. Here is its memory layout:
```
[HEADER:  8 bytes]  8
[value:   1 byte ]  9
[padding: 7 bytes] 16
```

An instance of the Boolean class takes 16 bytes of memory! Surprised? (Notice the padding at the end to align the object size to an 8 bytes granularity.)
Memory layout of subclasses of other classes

The next three rules are followed by the JVM to organize the the fields of classes that have superclasses. Rule 3 of object memory layout is the following:
#### Rule 3: Fields that belong to different classes of the hierarchy are NEVER mixed up together. Fields of the superclass come first, obeying rule 2, followed by the fields of the subclass.

Here is an example:
```
class A {
   long a;
   int b;
   int c;
}

class B extends A {
   long d;
}
```

An instance of B looks like this in memory:
```
[HEADER:  8 bytes]  8
[a:       8 bytes] 16
[b:       4 bytes] 20
[c:       4 bytes] 24
[d:       8 bytes] 32
```

The next rule is used when the fields of the superclass don't fit in a 4 bytes granularity. Here is what it says:
#### Rule 4: Between the last field of the superclass and the first field of the subclass there must be padding to align to a 4 bytes boundary.

Here is an example:
```
class A {
   byte a;
}

class B {
   byte b;
}
```
```
[HEADER:  8 bytes]  8
[a:       1 byte ]  9
[padding: 3 bytes] 12
[b:       1 byte ] 13
[padding: 3 bytes] 16
```

Notice the 3 bytes padding after field a to align b to a 4 bytes granularity. That space is lost and cannot be used by fields of class B.
The final rule is applied to save some space when the first field of the subclass is a long or double and the parent class doesn't end in an 8 bytes boundary.
#### Rule 5: When the first field of a subclass is a double or long and the superclass doesn't align to an 8 bytes boundary, JVM will break rule 2 and try to put an int, then shorts, then bytes, and then references at the beginning of the space reserved to the subclass until it fills the gap.

Here is an example:
```
class A {
  byte a;
}

class B {
  long b;
  short c;  
  byte d;
}
```

Here is the memory layout:
```
[HEADER:  8 bytes]  8
[a:       1 byte ]  9
[padding: 3 bytes] 12
[c:       2 bytes] 14
[d:       1 byte ] 15
[padding: 1 byte ] 16
[b:       8 bytes] 24
```

At byte 12, which is where class A 'ends', the JVM broke rule 2 and stuck a short and a byte before a long, to save 3 out of 4 bytes that would otherwise have been wasted.

### Memory layout of arrays
Arrays have an extra header field that contain the value of the 'length' variable. The array elements follow, and the arrays, as any regular objects, are also aligned to an 8 bytes boundary.
Here is the layout of a byte array with 3 elements:
```
[HEADER:  12 bytes] 12
[[0]:      1 byte ] 13
[[1]:      1 byte ] 14
[[2]:      1 byte ] 15
[padding:  1 byte ] 16
```

And here is the layout of a long array with 3 elements:
```
[HEADER:  12 bytes] 12
[padding:  4 bytes] 16
[[0]:      8 bytes] 24
[[1]:      8 bytes] 32
[[2]:      8 bytes] 40
```

### Memory layout of inner classes

Non-static inner classes have an extra 'hidden' field that holds a reference to the outer class. This field is a regular reference and it follows the rule of the in memory layout of references. Inner classes, for this reason, have an extra 4 bytes cost.

### Final thoughts

We have learned how to calculate the shallow size of any Java object in the 32 bit Sun JVM. Knowing how memory is structured can help you understand how much memory is used by instances of your classes.
In the next post I will will show code that puts it all together and uses reflection to calculate the deep size of an object. Subscribe to my Feed or keep watching this blog for updates!
POSTED BY DOMINGOS NETO AT 11:05 PM TAGS: JAVA

------END-------

伍涛@成都 重整
2013-07-28
