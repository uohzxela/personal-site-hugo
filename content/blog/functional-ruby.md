+++
showpagemeta = true
categories = ["functional programming", "ruby"]
tags = []
comments = true
draft = false
date = "2014-05-09T11:44:16+08:00"
title = "Getting functional in Ruby"
showcomments = true
slug = ""

+++

Treading the waters of functional programming in Ruby makes me feel poignant that I didn’t take CS1101S. Functional programming is a totally different programming paradigm, one that is declarative instead of imperative. You seek to describe what you want done instead of specifying how you want something done. As a result, the code is more concise, leading to easier maintainability in large-scale systems. This is one of the big pluses of FP, along with the fact that it is more amenable to concurrency than its imperative cousin.

Ruby is a functional-hybrid, as such concepts of blocks and lambdas exist, along with the idiomatic concept: Proc.

### Differences between functional and imperative programming in Ruby

Suppose we want to make a simple calculator. To do that we have to define add and subtract methods.

Here’s how you do it the imperative way:

```ruby
def calculate_add(a, b)
  a + b
end

def calculate_subtract(a, b)
  a - b
end

puts calculate_add(2, 3)
puts calculate_subtract(4, 2)
```

Here’s how you do it the functional way:

```ruby
def calculate(a, b)
  yield(a, b)
end

puts calculate(2, 3) { |a, b| a + b }
puts calculate(4, 2) { |a, b| a - b }
```

See the difference? The code is shortened by almost half! It’s easier to read and understand what the method calculation is doing, as the logic is exposed in the form of blocks, instead of being abstracted away from the programmer in the imperative example. No doubt, code readability and brevity is increased through the functional approach.

### What is a block?

In Ruby, everything is an object. There are some exceptions to this rule.

Block is not an object. Its definition from Wikipedia is as follows:

	In computer programming, a block is a section of code which is grouped together.

In the functional example shown above, `{ |a, b| a + b }` is a block, a bunch of code enclosed by curly brackets. It is stand-alone, not associated with any objects. I prefer to think of it as an external code snippet which is exposed to the programmer, as compared to the internal code snippet `a + b` in the imperative example.

Blocks are not methods (or functions as you call it); they are just a bunch of code. They are not objects too. As faithful Rubyists, how do we transform blocks into objects the Ruby way?

### Proc(edure): the wrapper class for block

Similar to the wrapper class in Java which provides a way to use primitive types as objects, Ruby has an idiomatic solution to transform a block into an object through the Proc class. The block still retains its functionality, but it can be saved into a variable and can be `call`ed at demand.

Going back to the calculator example, we re-implement the add method using Proc:

```ruby
def calculate(a, b, add)
  add.call(a, b)
end

add = Proc.new { |a, b| a + b }

puts calculate(2, 3, add)
```

Notice that we no longer pass a block as a parameter, instead we created an new instance of Proc called `add` that contains the original block and passed it as a parameter instead.

As `add` is an object, we can call it on its own:

```ruby
puts add.call(2,3)
```

In addition, we are free to pass `add` to whatever methods we want and `call` it indiscriminately. As blocks are one-time solutions, by wrapping it in an object we make it reusable anywhere in the code. This makes the code DRY.

### Lambda: anonymous functions

Proc has a cousin: lambda, the cornerstone of functional programming. They are very similar.

Here’s how you use a lambda:

```ruby
def calculate(a, b, add)
  add.call(a, b)
end

add = lambda { |a, b| a + b }

puts calculate(2, 3, add)
```

What is a lambda exactly? They are anonymous functions.

At this point, the astute reader may be wondering what is the difference between lambda and Proc. Remember that Proc is a wrapper class for block, which is simply a code snippet. Therefore Proc is not a function, but lambda is. When we use Proc in a method, the code snippet that is wrapped up is now a part of the method’s code. You may think of the code snippet as being embedded inside the method, whereas lambda is a merely a function that is being called by the method.

Here’s an example to illustrate the difference between Proc and lambda:

```ruby
def method1(a)
  a.call
  return "method1 ended"
end

proc = Proc.new { return }
lambda = lambda { return }

puts method1(lambda) #STDOUT: "method1 ended"
puts method1(proc)  #STDERR: LocalJumpError: unexpected return
```

Since the code snippet in `proc` is embedded inside the method, its return statement is treated as part of `method1` therefore an “unexpected return” error is flagged when `method1` is executed.

For lambda, it is treated as a function instead of a code snippet, therefore `method1` merely calls it and exits normally.

### Summary

Block: a bunch of code, non-object

Proc: a block encapsulated in a wrapper object

Lambda: anonymous function, is an object