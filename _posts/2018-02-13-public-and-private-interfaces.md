---
layout: post
title: "Public and private interfaces"
description: "What's the difference?"
categories: [code]
tags: [interfaces]
redirect_from:
  - /2018/02/23
---
When you rummage through open-source projects on Github written in Ruby, you'll often come across the words ```public```, ```private``` and ```protected```. Up until now I had a vague idea of what they meant but thanks to Sandi Metz I have a better understand about the what and why of these keywords.

To understand these keywords, first we have to understand what Interfaces are. Interfaces in the context of object oriented design (OOD) are what messages that objects in the application send either within themselves or to other objects. This is where the idea of "public" versus "private" interfaces comes in. One way to think about it as a fast-food joint like McDonald's. Imagine you roll up to the drive-thru and stop at the menu to order. If McDonald's was a program, this menu would be one of its public interfaces because people and things outside of McDonald's can use this to interact and send messages to the restaurant. What would constitute the private interface of Micky D's? This would comprise of things like the kitchen, the tools and people that make the food. In life, it's illegal to go behind the counter of a McDonald's and start cooking the food yourself and for good reason. In programs we treat the private interfaces with the same respect in order to simplify things and maintain order.

In OOD, we are concerned with the conversation our objects are having. By controlling how our objects are able to speak to each other, we can more easily direct the flow of information throughout our applications. What exactly defines a public versus private interface?

#### Public Interfaces:
 - Reveal their primary responsibility
 - Are expected to be invoked by others
 - Will not change on a whim
 - Are safe for others to depend on
 - Are thoroughly documented in tests

#### Private Interfaces:
 - Handle implementation details
 - Are not expected to be sent by other objects
 - Can change for any reason
 - Are unsafe for others to depend on
 - May not even be referenced in tests

So when we use ```private```, ```protected``` and ```public``` we are explicitly defining our interfaces. These are not hard and fast rules as you can get access to ```private``` methods in Ruby if you really want to. These are more like soft barriers to guide other developers and show them where the walls of your restaurant are ;)
```private``` methods in Ruby are considered the most unstable and thus this method provides the most restricted visibility.
```ruby
class Foo
    private

    def bar
        "Baz"
    end
end
```
Calling ```Foo.new.bar``` will result in the following error:
```NoMethodError (private method `bar' called for #<Foo:...>)```

Protected still expresses instability but with slightly looser restrictions on visibility.
```ruby
class Foo
    protected

    def bar
        "Baz"
    end
end
```
Calling ```Foo.new.bar``` will throw the same error:
```protected method `bar' called for #<Foo:...> (NoMethodError)```
However, if we call ```bar``` on an instance of ```self``` i.e. the class and/or subclass that owns this method we can call it without a problem.
```ruby
class Foo
    def call_bar
        self.bar
    end
    protected

    def bar
        "Baz"
    end
end
```

Finally, ```public``` are what all methods in a class default to, so they are able to be called on any instance of the class.

The point of all these keywords is not to complicate things but to reduce the complexity by controlling who gets access to what. By designing our applications to take advantage of public versus private interfaces, we can achieve better OOD.
