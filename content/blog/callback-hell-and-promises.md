+++
date = "2014-06-14T09:59:28+08:00"
slug = ""
comments = true
draft = false
showcomments = true
title = "To hell and back with promises"
categories = ["callbacks", "promises"]
tags = []
showpagemeta = true

+++

So this week I’m self-studying AngularJS as I will be working on the front-end for a project I’m assigned to. Not much building nowadays, as I am still learning the theory of how AngularJS apps work. But I did create a sample app by following [thinkster.io’s tutorial](http://www.thinkster.io/angularjs/r1gRPYp4kM/angularjs-tutorial-learn-to-build-modern-webapps). It’s not that well explained and it assumes the reader to have a basic familiarity with AngularJS concepts.

Prior to attempting the tutorial, I read the *O’Reilly AngularJS* book, authored by two engineers who worked on AngularJS at Google. It is well explained but the amount of errata in code snippets is really bad to the point of hindering my learning.

Thankfully, there is a canonical book on AngularJS, aptly titled [ng-book](https://www.ng-book.com/). I won’t gush about how excellent this book is; the author already did that for you on the homepage. Even though the author tries too hard to the extent of sounding like a snake oil salesman, he really walks the talk.

To truly understand AngularJS, there’s a slew of Javascript concepts to master. In this post I’ll be dissecting callbacks and promises.

<hr>

### What is a callback?

Simply put, it is a function that is passed to another function to be invoked at a later time.

Imagine we have a really slow function that blocks the code execution.

```javascript
response = sendLetterToMum();
// code onwards not executed while the above function is running
doSomething(response);
carryOnWithLife();
// rest of the code
```

This is the synchronous way of programming, you have a slow function (`sendLetterToMum()`) hogging up the execution time while the rest of the code is not executed. As an analogy, imagine yourself sending a letter to your mother at the post office, and staying there doing nothing until the reply has arrived.

This is why we have asynchronous programming, what any sane person would do in this scenario is to carry on with their daily life whilst waiting for the reply.

In Javascript, we implement this behavior using callbacks.

```javascript
sendLetterToMum(doSomething(response));
// code onwards executed while waiting for the above function to finish
carryOnWithLife();
// rest of the code
```

In this case, `doSomething(response)` is the callback. It is invoked at a later time when `sendLetterToMum()` returns a response.

Callbacks are very important in asynchronous programming, where it is used to avoid blocking during requests made to the server.

But it has its drawbacks.

### Callback hell

Code snippet demonstrating callback hell:

```javascript
User.get(fromId, {
  success: function(err, user) {
      if (err) return {error: err};
      user.friends.find(toId, function(err, friend) {
          if (err) return {error: err};
          user.sendMessage(friend, message, callback);
      });
  },
  failure: function(err) {
      return {error: err}
  }
});
```

As you can see, code readability for callbacks can easily spiral out of control as they become deeply nested. This is callback hell, and for each level, you have to explicitly handle errors.

Enter promises.

### Promise

Code snippet demonstrating the elegance of promises:

```javascript
User.get(fromId)
.then(function(user) {
  return user.friends.find(toId);
}, function(err) {
  // error handling
})
.then(function(friend) {
  return user.sendMessage(friend, message);
}, function(err) {
  // error handling
})
.then(function(success) {
  // success handling
}, function(err) {
  // error handling
});
```

Notice that the levels of indentations are much lesser now and the code is structured in a predictable way.

Promises are objects that represent value or thrown exception that is eventually returned. It acts as a proxy for the actual returned response, hence providing an abstraction that allows for a structured way of resolving returned values. As a result of a neater and more readable syntax, promises alleviate the pain of writing and reading asynchronous functions.

There are more to promises though, but I haven’t finished the chapter on promises in ng-book yet. All the more reason to read the book!