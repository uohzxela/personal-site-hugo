+++
title = "Start of a RESTful journey on Rails"
showpagemeta = true
categories = ["rest", "rails"]
tags = []
slug = ""
comments = true
draft = false
showcomments = true
date = "2014-05-23T11:16:31+08:00"

+++

After learning Ruby on Rubymonk (boy, I was glad I learnt from that site, it exposed me to advanced Ruby concepts which eased my Rails learning), I started to tackle Michael Hartl’s famous RoR [tutorial](https://www.railstutorial.org/).

Hartl does a good job of exposing the reader to quintessential software engineering concepts such as MVC, data modelling, refactoring, TDD and so on (my list does not do him justice, his coverage of concepts is insane). Rather than teaching the basics of Rails by showing the readers how to build a toy app, Hartl exposes you to the tried-and-tested tools of the trade by building an industrial-grade Twitter clone from the ground up, using Rails as a framework. In a sense, the tutorial is not really about Rails per se; it is about web development in the real world through the lens of an experienced software engineer.

I’m glad I started server-side programming with Rails. It is not just a framework but a way of thinking about web applications; it brings organization to the messy nature of web development with its emphasis on convention over configuration.

One interesting architecture pattern that Rails adopt is the RESTful network architecture. As an aside, MVC (another architecture pattern of Rails) is pretty intuitive, in a sense that each component in the system has a well-defined job (controlling, viewing, modelling) to execute. In contrast, REST is an unfamiliar concept with a lazy-sounding name.

I need to understand it.

### What is REST?

A quick google search would give you the name: “representational state transfer”, but that is not very helpful. Reading explanations of REST on Stackoverflow did not really help and I resorted to borrow RESTful Web Services from the shelf of nice software engineering books in the office.

The book finally enlightened me. The point of this blog post is to rehash what the authors are saying in layman terms.

	In RESTful architectures, the method information goes into the HTTP method. In Resource-Oriented Architectures, the scoping information goes into the URI.

Not very helpful again, but at least we got concrete concepts to unpack here.

What makes an architecture RESTful?

In web programming, we tell remote machines what to do by issuing HTTP requests. A HTTP request may look like this:

<img src="http://www3.ntu.edu.sg/home/ehchua/programming/webprogramming/images/HTTP_RequestMessageExample.png"></img>

The first word in the request line refers to one of the HTTP methods, in this case it is `GET`.

When we are talking about method information, we refer to the canonical CRUD methods that we use to manipulate data: Create, Read, Update, Delete. The HTTP equivalents are: `POST, GET, PATCH, DELETE` respectively.

For RESTful architectures, your method of manipulating state of data in remote machines (simply put, method information) must correspond to the HTTP method in the request message. That is, if you want to update data in a server, you should use HTTP `UPDATE` request, create data with HTTP `POST` request and so on.

Non-RESTful architectures do not follow this rule; their HTTP methods do not match the method information. For instance, you send a HTTP `GET` request message even when you want to update data.

For scoping information in URI, it is intrinsically linked to addressability. But addressability of what? The answer is resources. Here, a resource can be anything (a document, a row in a database etc.) that can be CRUD’d by the programmer. For starters, you may wish to think of resource as a more formally-sounding name for pieces of data residing in the server. With a formal-sounding name comes with other perks, such as addressability. When we adopt the RESTful way, we accord pieces of data in the server with identifiers to address them easily at a later time. Since resources are exposed through URIs, addressability is achieved through the scoping information in the URIs.

For example, when we want to retrieve search results related to jellyfish, we go to this URI: http://www.google.com/search?q=jellyfish . Alert readers may glean from this example that the reason they can also access the same page is that the URI has scoping information which makes the jellyfish search results addressable. Hence we conclude that Google Search’s web service is RESTful.

An example of a non-RESTful app would be Gmail web application. If you were to access Gmail, you would notice from the browser that there is only one URI: https://mail.google.com throughout the whole session.