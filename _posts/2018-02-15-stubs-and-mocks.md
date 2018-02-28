---
layout: post
title: "Stubs and Mocks"
description: "Exploring stubs and mocks"
categories: [code]
tags: [stubs, mocks, TDD]
redirect_from:
  - /2018/02/15
---
When I attended Dev Bootcamp, from the very beginning we were told to test our code. Test driven development (TDD) is a philosophy and method for developing software that emphasizes the practice of writing tests before you write any source code. Red, green, refactor as the saying goes refers to writing the initial test (having it fail, thus going red), writing the source code to have that test pass (making it go green) and then taking the time to refactor your code afterwards. This method is more time and effort upfront but can pay off big time later on in development. Although the basic idea and procedure of TDD (a la red green refactor) are fairly easy to understand, for a long time I didn't understand what mocks and stubs were. You hear those terms thrown around a lot when talking about testing and up until now I barely had any idea what they meant. Today I'd like to share what I've learned about these important concepts.

## Stubs

In testing we want to isolate our models as best we can. When our objects are dependent on other objects we violate the principle of isolated unit tests and run the risk of creating more tests than we need to or complicate our test suite. One way we can reduce complexity and isolate our models is by "stubbing" methods. Basically we implement a version of a method that returns a canned answer. One way to think about a stub is a test "double". It stands in for something that would normally exist in your application but without all the complication of setting up the real context of your program. We do have to be mindful though. As Sandi Metz states:
  "[This] choice between injecting real or fake objects has far-reaching consequences. Injecting the same objects at test time as are used at runtime ensures that tests break correctly but may lead to long running tests. Alternatively, injecting doubles can speed tests but leave them vulnerable to constructing a fantasy world where test work but the application fails."
Basically, we need to be careful how we use doubles and make sure we aren't creating false positives (or negatives) by including these artificial objects in our tests.

## Mocks

Testing whether or not our objects are sending and receiving the correct messages is an important thing to test. Something that testing frameworks come equipped with are mocks. Mocks test behavior, as opposed to state. Instead of making assertions about what a message returns, mocks are concerned with defining an expectation that a message will be sent. Take this example:

```ruby
class TicTacToe
  attr_reader :human_player, :computer_player, :board

  def initialize(human_player, computer_player, board)
    @human_player = human_player
    @computer_player = computer_player
    @board = board
  end

  def human_player_take_turn
    human_player.choose_spot
  end
end

class TicTacToeTest < MiniTest::Unit::Testcase

  def setup
    @human_player = MiniTest::Mock.new
    @game = TicTacToe.new(@human_player, Computer.new, Board.new)
  end

  def test_human_choose_spot_is_called_on_take_turn
    @human_player.expect(:choose_spot)
    @game.human_player_take_turn
    @human.verify
  end
end

```
Here we are making sure that ```choose_spot``` is being called when ```human_player_take_turn``` is called. This is a very rough example but the point here is that we can see if our expectations about what objects send and receive what messages are correct. We don't care what ```human_player_take_turn``` returns us, we only care that TicTicToe should have the responsibility of sending that message. Mocks allow us to make sure that the objects in our tests fulfills its responsibilities without duplicating assertions that belong elsewhere. In this way we keep our code DRY and our tests become a better source of documentation for other developers.
