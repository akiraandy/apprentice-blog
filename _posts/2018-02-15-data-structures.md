---
layout: post
title: "Data Structures"
description: "The wide world of data structures"
categories: [code]
tags: [data structures]
redirect_from:
  - /2018/02/15
---
This week I have to give a presentation on data structures and up until doing the research I can honestly say I did not know a whole lot about how data structures exist in memory or how they operate under the hood. Data structures is just a fancy way of organizing information in a computer system. You probably know plenty of data structures, arrays, heaps, trees, graphs, hashes, these are all great examples. I'd like to dive into some of the interesting things I learned this week about data structures and hopefully you'll find them interesting too!

Let's start with arrays. Arrays is an ordered set of elements stored in memory. For the sake of simplicity I'm going to talk about arrays in C and C++. When an array is created, the computer will allocate a limited set of memory to the array based on how many elements the array has and what data type it contains. So for example, in most systems Integers are 4 bytes. In the case where we create an array of 10 integer elements the computer must allocate 4 * 10 bytes of memory, totaling 40bytes. How do we gain access to an array? Well most of you know that we usually do it like this:
```c++
int example_array[10];
example_array[0] = 1;
return example_array[0];
// => 1
```
But how does the computer know how to access any elements? How fast can find them? Well it turns out it's pretty fast and easy. So when an array is created, the memory is created in a contiguous block of memory meaning that each element in the array sits right adjacent to the next element in the array. Whenever we reference our array we are actually referencing the very first block of memory or the array's "base element". To find out where any subsequent elements are, it uses pointer arithmetic to find out what value exists at the target index. So let's say our array is created at memory block ```100```. And let's say we are looking for the element at 2, so this: ```example_array[2]```. The computer multiplies desired index by the byte amount of each element in the array, in this case, 4 bytes (because an Integer is 4 bytes in our system). 4 * 2 would give us 8 bytes, so then we look to our ```100``` memory block and we access the memory block 8 bytes away from that, block ```108```. And that's how C and C++ access arrays! It's pretty cool actually. The math for finding a multidimensional array is also pretty simple. So let's say we have a two dimensional array, we could find our index of ```Array[m][n]``` by doing the following calculation: (m * C + n)B and then adding the resulting bytes to whatever memory block our base element is pointing to. C here refers to the total columns and B is the byte amount of elements in the array.

Another cool thing I learned this week was how hash tables (a.k.a. HashMaps) work. For those who aren't familiar with hash tables, they are a data structure that stores a key/value pairs where the keys are always unique. Think about a person's social security number followed by their name. Under the hood they are arrays but have a special sauce that makes them very unique. Hash tables employ a hash function that when given a key, spits out an integer that represents an element in the underlying array. A good hash function should be able to create a unique integer each time it is given a key but there are occasions where a "collision" occurs. A "collision" in terms of a hash map just means that the hash function has given you the same integer for two different keys and since keys must be unique in a hash table structure this presents a problem. There are multiple ways of solving a collision but one of the most popular methods is called the separate chaining. In this strategy, when we encounter a collision, we add a linked list node to that element in the underlying array which will carry the hash and the value for the key. There are many other strategies for resolving collisions and they are ingenious. I would encourage you to research them and check them out!
