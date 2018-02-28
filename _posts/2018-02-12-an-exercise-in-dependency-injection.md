---
layout: post
title: "An exercise in dependency injection"
description: "SOLID stuff!"
categories: [code]
tags: [SOLID, dependency injection]
redirect_from:
  - /2018/02/12
---
For my apprenticeship I'm making a web-app version of Tic Tac Toe. There are two game modes, human versus human and human versus computer. As I've been adding features and reading more about software engineering patterns and design, the code-spirits beckon me to refactor. In today's post, I'd like to explore how we might refactor a method in my Human player class to more decoupled from the game controller class that it currently relies on.

Here is the Human class as it stands:
```ruby
class Human < Player

  def initialize(marker, first=false)
    super
  end

  def take_turn(game, spot)
     game.board.fill_spot(spot, marker)
  end
end
```
Do you smell that? Ah, yes, a code-smell. Not only does it rely on another object to do it's job, but it goes two levels deep. One to the game object, second to the board object and then calls ```fill_spot``` on that board. Why should a player object know about a game or board? How can we alter this method to: 1. isolate it from other dependencies 2. have it make sense in the context of the program/game?

In her book *Practical Object Oriented-Design in Ruby* Sandi Metz talks about how to recognize when an object has a dependency:
* The name of another class appears
* The name of a message that the object intends to send to someone other than ```self```
* What the arguments are that a message requires
* The object somehow knows the order of those arguments

Our method (and our class) raise red flags on all those counts. We call not one but **two** classes, ```take_turn``` sends this message to someone other than itself, it knows that ```fill_spot``` requires two arguments and it knows the order in which those arguments should appear.

Currently, the ```Human``` class is *coupled* to ```Game``` and in turn to ```Board```. What can we do to decouple them? I think to answer this question we need to name and define what a turn is for a human player. Let's think about what a human player does when they take a turn in Tic Tac Toe. They see if any spaces are available on the board and then write in their respective marker into a spot on the board. Not accounting for error, this is how it would work. The two most important components of this action that need to be **communicated** is what the spot is and what the player's marker is. If a human and a game object were to actually have a conversation about a turn how would that conversation happen?

Game- "Hey, your turn! Can you tell me what your turn is?"
Human- "Hi, my turn is 'X', 6."
Game- "Hey board, here is the turn."
--Board jumps in--
Board- "Let me just fill myself in here...okay, done!"
Game- "Next turn!"

So the most important information from the human here is their spot and their marker. How might we express this information in code? We could deliver the information in an array like so:
```
["X", 6]
```
or as a hash
```ruby
{marker: "X", spot: 6}
```

One of Sandi Metz recommendations for good OOD is that you hide data structures. For one, they can sometimes be difficult to understand i.e. ```["X", 6]``` and if your objects rely on data coming in to be structured in a certain way, this can wreak havok later on if for some reason that data structure changes. How can avoid relying on brittle data structures and make our code more readable and intention revealing?

Since both computer players and human players take turns, wouldn't it make sense if after taking a turn they returned a ```Turn```? In cases where multiple objects share some data structure, it is sometimes helpful to create a struct to act as an object that hides the complexity of that data structure.

Let's define our new ```Turn``` struct:
```ruby
Turn = Struct.new(:marker, :spot)
```
Now we have an object that resembles what the end results of taking a turn are. Since both a computer player and a human player both take turns, to add it to our Human and Computer player class, let's add the struct to their parent class, ```Player```.

```ruby
class Player

  Turn = Struct.new(:marker, :spot)

  attr_reader :marker
  attr_accessor :opponent

  def initialize(marker)
    @marker = marker
    @opponent = nil
  end
end
```

Now let's look at our human class again:
```ruby
class Human < Player

  def initialize(marker, first=false)
    super
  end

  def take_turn(spot)
     Turn.new(marker, spot)
  end
end
```

A human now no longer needs to know about a game or a board. All the human class knows is how to make a turn. We have simplified the idea of a turn and if we need to make adjustments in the future, we can look to the ```Turn``` struct and make adjustments there as necessary. What is most important to take away from this is that we are trying to send messages and communicate across our objects. What we *don't* want to do is consume an object, conduct surgery on it and then ship it back out again. It doesn't make for great relationships, nor does it make for clean, decoupled code.
