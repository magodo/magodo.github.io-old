
---
layout: post
title: "Array, String, Pointer and Reference"
categories: "cpp"
---
<!--more-->

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents** 

- [Favor C++ std::string over C-style string](#favor-c-stdstring-over-c-style-string)
- [Pointers](#pointers)
  - [The address-of operator(&) returns a pointer](#the-address-of-operator&-returns-a-pointer)
  - [Declaration](#declaration)
  - [Null Pointer](#null-pointer)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Favor C++ std::string over C-style string

Unless you have a specific, compelling reason to use C-style strings, use std::string instead. std::string is easier, safer, and more flexible

# Pointers

## The address-of operator(&) returns a pointer

It’s worth noting that the address-of operator (&) doesn’t return the address of its operand as a literal. Instead, it returns a pointer containing the address of the operand, whose type is derived from the argument (e.g. taking the address of an int will return the address in an int pointer).

## Declaration

* When declaring a pointer variable, put the asterisk next to the variable name.
* When declaring a function, put the asterisk of a pointer return value next to the type.

## Null Pointer

* Initialize your pointers to a null value (0) if you’re not giving them another value.
* Don't use `NULL` in C++ since it's technically not part of C++(defined by C).
* Starting from C++11:
    * Use `nullptr` to initialize your pointer to a null value (`nullptr` is both a _keyword_ and an _rvaule constant_)
    * C++11 also introduces a new type called `std::nullptr_t` (in header <cstddef>). `std::nullptr_t` can only hold one value: `nullptr`!


