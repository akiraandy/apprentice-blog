---
layout: post
title: "Simple Design Principles"
description: "4 Simple rules"
categories: [code]
tags: [design, code]
redirect_from:
  - /2018/02/06
---

Hi, I'm Andy. I'm a new apprentice here at 8th Light. I'll be posting to this blog to share my thoughts, musings and frustrations related to software development along my journey.

I would like to start by exploring the 4 rules of simple design proposed by Corey Haines in his book, *4 Rules of Simple Design* (didn't see that one coming, didja?).

The four rules can be summarized as follows:
1. Tests Pass
2. Expresses Intent
3. No Duplication (DRY)
4. Small

# Test Names Should Influence Object's API

On my interview day here at 8th Light my mentor Taka told me that tests can stand in as a form of documentation for the objects they are testing. Up to that point I felt that tests were a necessary evil when trying to write sustainable code. I had never thought about how tests could be a multifaceted tool for both the project developer and any future developers on the project. Not only do tests make your objects more resilient in the face of changes, they can also tell you a lot about what the code is *supposed* to do. This may seem obvious but as developers when we are knee deep in the weeds, tests can save us from becoming too lost. One way to avoid getting lost is to have our code present clear *intent*.

Doesn't adding more code (which tests inevitably do) make our projects more complicated? Is it possible to use tests to make our code more intuitive? Let's dive into a yummy thought-morsel provided by Haines that might help us here.

Consider the following case where we have a tic tac toe board initialized with all nil values.

```ruby
it "should be empty on creation" do
  board = Board.new
  assert_equal 0, board.spaces.compact.count
end

it "should be able to add a marker to itself" do
  board = Board.new
  board.add_marker(5, "X")
  assert_equal 1, board.spaces.compact.count
end
```

These tests seem reasonable right? We want to check if the board is empty and we want to able to test that a board has one less space when we fill a spot on the board. However, on closer inspection, the test code makes mention no mention of an empty space or markers. Haines challenges us to spend time thinking about the names we give our tests. "We want them to describe both the behavior of the system and the way we expect to use the component under test. ...Let the code in the test be a mirror in the test description."

```ruby
it "should be empty on creation" do
  board = Board.new
  assert_true board.empty?
end

it "should be able to add a marker itself" do
  board = Board.new
  board.add_marker(5, "X")
  assert_true board.marker_at?(5)
end
```

Adding these methods to our object can also make our tests more robust and expressive.

```ruby
it "should not be empty after adding a marker" do
  board = Board.new
  board.add_marker(5, "X")
  assert_false board.empty?
end
```

By creating symmetry between tests and the models they are testing, both our models and our tests reap the benefits of clearer intent and expressiveness. Going forward I want to practice this technique and see if I can emulate this in my own work.

Fail. Ask questions. Learn. Repeat.
