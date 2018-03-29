---
layout: post
title: "Design Patterns"
description: "An exploration of design patterns"
categories: [code]
tags: [design patterns]
redirect_from:
  - /2018/03/29
---
Whether you're tackling a greenfield project or diving deep into legacy code, we as developers must carefully decide on how to best solve the problems we are faced with. There are infinite ways to solve a finite problem, but as crafters we should strive to create solutions that allow our code to be as resilient as possible. What does resilient in this context? Resilient code is code that adapts to changes easily. As requirements in software are always changing based on changing needs and wants, our code should be designed to grow and change with as easy as possible. Trying to identify the optimal design strategy for any given problem can seem daunting but thankfully to those who have come before us, we can refer to the Design Principles to guide us.

The Design Principles are a set of principles that outline different solutions for a variety of design issues that occur often in the development of software. These principles were first outlined in *Design Patterns: Elements of Reusable Object-Oriented Software* by Erich Gamma, Richard Helm, Ralph Johnson and John Vlissides also known as the Gang of Four. Since 1994 (when it was published) until now, these design principles have remained hugely important and relevant in the development of resilient software systems. The design patterns are not copy and paste pieces of code but rather a blueprint for how to tackle common issues in designing software. These design principles can be adapted to any language, framework or system. In the next few paragraphs I would like to explore 3 of these design patterns.

COMMAND
The Command pattern encapsulates a request as an object, basically making it an object-oriented callback. The command pattern is not concerned with state or variables. It is actually quite "functional" in nature. You'll often find the command pattern implemented in situations when you need to execute a series of methods that result in an object being created/deleted/updated. When you run the execute command, validations can get run, classifications can be derived and databases can get updated. The Command pattern is implemented with an abstract class where multiple objects can implement the "execute" method. The user running the command doesn't care about the innards of the execute command, they are more concerned with the end state. The Command pattern (like most good design pattens) obfuscate complexity and are "black boxy" in nature. They get run, do their thing and that's the end of it. Another advantage of the Command pattern is the way it supports and "undo" action. If the object knows enough to change the system with the execute command, it stands to reason it knows enough to be able to undo that operation and return the system to its original state.

STRATEGY
Imagine you are trying to design a program that is able to implement Bubble Sort on a variety of primitive data types (I know, I know, Bubble Sort is probably one of the least efficient sorting algorithms but bear with me here). Using the Strategy pattern, we can define an interface that implements all the basics of a Bubble Sort algorithm (swap(), outOfOrder(), length(), setArray()) and just have different handlers implement that interface but with their unique data type. You'll see this pattern a lot when you need to encapsulate interface details in a base class (see bubble sort interface) and bury the implementation details (i.e. each implementation of the algorithm with it's primitive data type) in the derived classes. To continue from the bubble sort example, another nice detail of the Strategy pattern is that no matter how many different data types you have now (or in the future) the base class will not need to change which makes extension easy/painless.

Here is an example of what the bubble sort pattern might look like with the strategy pattern:
```Java
public abstract class BubbleSorter {
    private int length;
    private int operations;
    private SortHandle sortHandle;

    public BubbleSorter(SortHandle handle) {
        this.sortHandle = handle;
    }

    public int sort(Object array) {
        sortHandle.setArray(array);
        length = sortHandle.length();
        operations = 0;

        if (length <= 1)
            return operations;

        for (int nextToLast = length -2; nextToLast >= 0; nextToLast--)
            for (int index = 0; index <= nextToLast; index++) {
                if (sortHandle.outOfOrder(index))
                    sortHandle.swap(index);
                operations++;
            }

            return operations;
    }
}
```

```Java
public interface SortHandle {
    void swap(int index);
    boolean outOfOrder(int index);
    int length();
    void setArray(Object array);
}
```

```Java
public class IntSortHandle implements SortHandle {
   private int[] array = null;

    @Override
    public void swap(int index) {
        int temp = array[index];
        array[index] = array[index];
        array[index + 1] = temp;
    }

    @Override
    public boolean outOfOrder(int index) {
        return (array[index] > array[index + 1]);
    }

    @Override
    public int length() {
        return array.length;
    }

    @Override
    public void setArray(Object array) {
        this.array = (int[])array;
    }
}
```

You could then write a ```StringSortHandler``` or ```DoubleSortHandler``` or whatever primitive data type you needed. You could probably extend this to include non-primitive data types.

NULL OBJECT
Most of us have had to write a conditional at some point that checks for a null value. Sometimes we check for the null value and do something if it's true. Sometimes we deal with the situation by throwing an error, sometimes we us a ```catch/try``` block. These are all different ways of dealing with a null value, but what if we wrapped our potentially null value in an object that represented the absence of that object or value? That is what the Null Object design pattern tries to accomplish. What if we were writing a server and one of our classes was in charge of parsing the raw request string and the request was empty or was improperly formatted? We would have to do some checking of values that would be potentially null. What if we instead created a ```InvalidRequest``` object that would respond normally to methods called on it like ```.getPath()``` or ```.getMethod()```? We are insuring that functions always return valid objects, even when they fail. Type checking is a general no-no in development and by using this pattern we can avoid that pitfall.  So next time you might end up with a null value, try wrapping it up in an object instead!
