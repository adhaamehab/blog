---
title : "You are Writing Python the Wrong Way"
date : 2024-02-03T10:30:00+03:00
draft : false
description : "A rant about how Python is a functional, declarative language and why the language itself hates complexity and imperative code."
summary : "A discussion on Python's true nature as a functional and declarative language."
toc : true
readTime : true
autonumber : true
tags : ['python', 'functional-programming']
showTags : true
---

# You are Writing Python the Wrong Way
> Or I have a high ego and I think you should listen to me.

## Introduction

For years, I've seen Python code that makes me cringe. It's not because the code is inherently bad, but because it misses the essence of what Python is meant to be. Python is not just a language; it's a philosophy. And yet, many developers write Python in a way that goes against its core principles.

## Multi-Paradigm Language

Since Python is a multi-paradigm language, you can write Python code in a variety of ways. You can write procedural code, object-oriented code, or functional code. But Python is designed to be simple and readable. That's probably the only common thing between all these paradigms.

To quote the official Python docs:
> Python is an interpreted, interactive, object-oriented programming language. It incorporates modules, exceptions, dynamic typing, very high level dynamic data types, and classes. It supports multiple programming paradigms beyond object-oriented programming, such as procedural and functional programming. Python combines remarkable power with very clear syntax.

So Python at its core can be used in a variety of ways, but it's designed to be simple, readable, and elegant. So why choose complexity over simplicity?

## Declarative Programming

Python is a declarative language. I usually use Python in a functional way, but this is about more than just using one paradigm over another. This is about making all Python code look the same. When you write Python in an imperative way, you make the code harder to read and maintain. You make the code harder to reason about.

The core idea behind declarative programming is that you are trying to declare what you want to do, not how you want to do it. That's it. It's that easy.

## Pythonic Code

Python is a battery-included language. It has a rich standard library that provides a lot of functionality out of the box. So when you write Python code, you should always try to use the standard library as much as possible. This will make your code more readable, more maintainable, and more efficient.

This is what Pythonic code is all about. It's about minimizing cognitive overhead as you read the code. So, instead of having to read the inside of a for loop to understand that you are just trying to filter a list, you can just use a list comprehension. Instead of having to read the inside of a function to understand that you are just trying to map a function over a list, you can just use a lambda function.

This use of common idioms and patterns (not just in Python, but in any language) is what makes code more readable and maintainable. And what creates a better communication channel between developers as they read and write code.

## Vouching for Functional Programming

I'm not a Haskell or Lisp developer. And I'm not trying to explain what's a monad or a functor. But I'm a big fan of functional programming. For me, functional programming is about writing declarative code that's preferably stateless and immutable with minimal side effects. And Python is a great language for that.

When you write Python in a functional way, you make it easier for you (the writer) to think about the code and for others (the readers) to understand it. You make it easier to test and debug. You make it easier to scale and maintain.

Pairing data classes with functional programming is a great way to write Python code. You can write code that's very efficient and self-explanatory. That doesn't require other people to have extensive experience or domain knowledge to understand what's going on.

## That's How the Language is Designed

If you look at the Python standard library, large frameworks (like Django and Flask), and popular libraries (like Pandas and Numpy), you'll see that they all follow the same principles. They provide declarative, human-driven APIs that are easy to understand and use. I've seen a lot of excellent engineers and data scientists write very complex algorithms in a declarative way that makes someone as dumb as me understand what's going on in minutes. While I've also seen a lot of engineers who would make me spend an hour trying to understand a simple for loop.

## Final Advice

So, my advice to you is this: write Python in a way that's easy to read. Use the standard library as much as possible. Spend time to invest in the quality of your code. Read a lot of Python code from big projects. This will help you understand the value of writing declarative, efficient code.
