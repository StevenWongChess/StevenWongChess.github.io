---
layout: post
title:  "Tour of C++ Notes"
date:   2024-05-12
categories: C++
---

##### 1-The Basics

###### 1.1 Intro

Check [cppreference](https://en.cppreference.com/w/)

###### 1.2 Programs

c++20 use module `import std;` instead of `#include <iostream>`

###### 1.3 Function

function override

- same name but different parameters
- compiler will choose
- cannot be ambiguous

###### 1.4 Type, Variable, Arithmetic

digital separator `1'000'000`

type conversion in expressions e.g. `double + int`

- called usual arithmetic conversions
- remain highest precision

Order of `f(x)+g(y)` is unspecified

initialize

- should use `{}` over `=`
- e.g. `int i {2.1}` will report error
- `=` will type conversion (price paid for C compatibility)

###### 1.5 Scope

4 types

- local
- class
- namespace
- global

###### 1.6 Constants

`const` means not to change value

`constexpr`/`consteval` means evaluated at compile time

- in essence is c++'s pure function(like functional programming)
- for grammar(1.7, 1.8, 7.2) OR performance
- `constexpr` function can be used for both, but `consteval` function only compile time

###### 1.7 Ptr, Array, Ref

Size of array MUST be `constexpr`

function that take `const &` very common

No null reference, reference can't change

###### 1.8 Test

###### 1.9 Mapping to hardware

Initialize != assignment

###### 1.10 Advice

37 useful advice



##### 2-User-Defined Types

###### 2.1 Intro

User-defined is often preferred than built-in 

###### 2.2 Structures

###### 2.3 Class

###### 2.4 Enum

scope of enum is a class `enum class XX{}`

- XX::yy
- we can use `using enum XX;` to avoid repetition
- can define `enum XX` without class

###### 2.5 ==Union==

need review

`union` is for space optimization

- avoid access member directly
- can use `variant<T1, T2>` 



##### 3-Modularity

###### 3.1 Intro

2 parts

- Interface -> declaration, can be many
- Implementation -> only one definition

###### 3.2 Separate Compile

declaration

- header (traditional way), we have .cpp and .h has ==4 cons==
  - compile time ++
  - order dependency
  - Define same entity in 2 file will crash
  - Transitivity
- Module (new in c++20)
  - but module std is not available
  - .h will be auto generated

###### 3.3 Namespace

to avoid repeating `std::`

- `using std::swap;`  only for swap
- `using namespace std;` for everything

can work with module, using in module does not affect user

BUT do not put using in header

###### 3.4 Function args and return values

3 concerns

- copied or shared? 
  - Small value by-value, large by-reference
  - “the size of two or three pointers or less” is a good rule of thumb

- if shared, mutable?
- If moved, what about the "empty" one

Can have default arg value (just a notationally simpler alternative of overloading)

Example Matrix pp39

- 20th century old style
- now use a move constructor

Even return type can `auto`

- even better way `auto function(T1 arg1, T2 arg2) -> ReturnT`
- use with caution

Class is used to return multiple stuff, can use structured binding for better syntax

```c++
return {a, b};
// structured binding (fluent in leetcode lol)
auto [a, b]: ... 
const auto& [a, b]: ...
```



##### 4-Error Handling ==review==

Not so important right now



##### 5-Class

###### 5.1 Intro

Chap 5-8 about class

- Concrete class, abstract class, hierarchy
- constructor, destructors, assignmets
- template
- generic programming

###### 5.2 Concrete Class

simple operations should be inline

- for efficiency
- functions defined in a class is inline by default
- use `inline` keyword

###### 5.3 Abstract Types

`virtual` means may be redefined later in a class derived form this one

- `virtual` function (may not need to define)
  - A class with a virtual function should have a virtual destructor

- `virtual fun = 0;` , here `=0` means pure virtual function (derived class MUST define)
  - optional to use `override` in derived class
- abstract class = class with pure virtual function
  - cannot create an object of it
  - But, usually use pointer/reference

Def: polymorphic type

- better use ref/ptr

###### 5.4 Virtual Function

virtual function table tell which to use

###### 5.5 Class Hierarchies

An object of a derived class can be used wherever an object of a base class is required

use `dynamic_cast<>()` , "is kind of" operator, decide in runtime if the object pointed is of some type

Avoid new & delete naked ptr

- use smart ptr, i.e. `unique_ptr`, `shared_ptr`



##### 6-Essential Ops

###### 6.1 Intro

Constructor, destructor, copy, move are closely related

- A good rule of thumb (sometimes called the rule of zero) is to 
  - Either define all of the essential operations 
  - OR none (using the `default` for all)
- 5 usage of copy/move
  - source of assignment
  - object initializer
  - function arg
  - function return value
  - Exception
- when class has pointer member, should not use `default`
- `=delete` 
  - indicates an operation not to be generated
  - any usage -> cause compile time error

- `explicit constructor()` to avoid unexpected type conversion
  - `vector v(2)` is always safe, 
  - this is for `vector v = 2`

###### 6.2 Copy & Move

Default copy is memberwise copy

- Not good for class that handle resource
- hint, if destructor is needed -> do not memberwise copy

2 kinds of copy

- copy constructor
- copy assignment

```c++
Vector(const Vector& a);                // copy constructor
Vector& operator=(const Vector& a);     // copy assignment
```

function return value too large

- use move

Rvalue = a value that you can't assign to 

Why move?

- we can pass parameter by reference
- but we can't return local variable's reference

2 kinds of move

- move constructor
- move assignment

```c++
Vector(Vector&& a);                        // move constructor
Vector& operator=(Vector&& a);             // move assignment
```

###### 6.3 Resource Management

RAII, resource can be released at the end of its scope, and use move to move between scope

###### 6.4 Operator Overloading

`<==>`

###### 6.5 Conventional Operations

###### 6.6 User-defined Literals

```c++
constexpr complex<double> operator""i(long double arg)     // imaginary literal
```



##### 7-Templates ==review==

###### 7.1 Intro

###### 7.2 Parameterized Types

template is compiler time technique

- take type arg
- can also take value arg (string literal not supported yet)
- can deduct type (be careful) `pair xxx = {1, 1.2}` instead of `pair<int, double>`

second line below c++20 use "concept checking" (compile time)

```c++
template <typename T>
template <Element T> 
```

deduction guide

- better to design templates that deduction guide is not needed

```c++
template<typename Iter>
        Vector(Iter,Iter) -> Vector<typename Iter::value_type>;
Vector v3 {v1.begin(),v1.begin()+2}; // no ambiguous anymore

Vector vs3 {"Hello"s, "World"}; // ambiguous const char[6]
Vector vs2 {"Hello"s, "World"s}; // ok
```

###### 7.3 Parameterized Operations

3 parameterized operations

- function template
  - can be a member function in class
  - but can't be virtual
- function object / functor
  - widely used as arg to algo
- Lambda (lambda can have template as well)

###### 7.4 Template Mechanisms

4 language facilities

- Variable templates
- alias templates
- compile-time selection
- Compile-time to inquire



##### 8-Concepts & Generic Programming

###### 8.1 Intro

detailed benefits of templates, templates is to support generic programming 

###### 8.2 Concepts

example

```c++
template<Sequence Seq, Number Num>
        requires Arithmetic<range_value_t<Seq>,Num>
Num sum(const Seq& s, Num n);
```

simple concepts are not hard to define

- How to? see 8.2.4

Essence of `auto` is concept

- The keyword auto denotes the least constrained concept for a value:

Type vs concepts

- Author's ideal is that eventually most functions will be defined as template functions with their arguments constrained by concepts


###### 8.3 Generic Programming ==review==

###### 8.4 Variadic Templates

implementations

- traditionally, recursive with `...` parameter pack (hard and expensive)
- use fold expression C++17 (left/right)
- use `forward()` to move variables unchanged

###### 8.5 Template Compilation Model ==review==



##### 9-Library Overview

###### 9.1 Intro

###### 9.2 Standard-Library Components

A very long list

###### 9.3 S-L Organization

- namespace `std::xxx`
- old way `iterator`, new way `range`
- module (very likely in C++23)
- header (a very long list)



##### 10-String & Regex

###### 10.1 Intro

`string_view` does not care about `std::string` or `char[]`

###### 10.2 String

###### 10.3 String Views

`string_view` is (pointer, length) pair



###### 10.4 Regex ==review==



##### 11-I/O ==review==

##### 12-Containers

###### 12.1 Intro

###### 12.2 `vector`

If you have a class hierarchy (§5.5) that relies on virtual functions to get polymorphic behavior, do not store objects directly in a container. Instead store a pointer

Vector `[]` does not check for performance, `at` check but 10% slower

###### 12.3 `list`

###### 12.4 `forward_list`

(only allowing forward iteration, unlike list which is double-linked)

###### 12.5 `map`

`map`

- implemented by balanced BST (oftern red-black tree)

- fast for lookup & insertion

###### 12.6 `unordered_map`

`map` lookup cost `log(n)`

- to improve, use hased lookup rather than comparison
- Given a good hash function, an unordered_map is much faster than a map for large containers. However, the worst-case behavior of an unordered_map with a poor hash function is far worse than that of a map


###### 12.7 Allocators

default is `new()` `delete()`

- user can define better ones
- e.g. pool allocator

###### 12.8 Container Overview

More

- have container adaptors
- Container-like types 
- In common, basic operations
- emplace operation is different



##### 13-Algorithms

###### 13.1 Intro

- use `begin()` `end()` style
- don't have range for
- std containers + `back_inserter()` => avoid C-style `realloc()`

###### 13.2 Use of Iterators

###### 13.3 Iterator Types ==review example==

- stream iterator (for I/O), e.g. `ostream_iterator<string>`

###### 13.4 Use of Predicates

###### 13.5 Algorithm Overview

some algotithms listed in table

###### 13.6 Parallel Algorithms

2 kinds

- parallel execution
- vectorized execution

detail in `<execution>`

- seq
- par
- unseq
- par_unseq



##### 14-Ranges

###### 14.1 Intro

2 versions of algoriths

- use concepts, in `<ranges>`
- unconstranined for compatibility, e.g. `sort(v.begin(), v.end())`

###### 14.2 Views

A view is a way of looking at range

- `all_view`
- `filter_view`
- `transform_view`
- ... many many in std (see table)

###### 14.3 Generators

- many in std (see table)

e.g. 

```c++
for (int x: iota_view(42, 52)) // means 42, 43, ...51
```

###### 14.4 Pipelines

Can have multiple filters

e.g.

```c++
for (int x : r | views::filter(odd) | views::take(3))
```

###### 14.5 Concepts Overview ==review==

3 kinds for standard library (each has long list of table)

- properties of types
- iterators
- ranges



##### 15-Pointers & Containers

###### 15.1 Intro

###### 15.2 Pointers

- `unique_ptr<T>`
- `shared_ptr<T>`
- `weak_ptr<T>`
- `span<T>` = pointer to a contiguous sequence of T
- `string_view<T>` = const substring
- `X_iterator<C>` = sequence of elements from C, X is the name indicates the kind of iterator

`span` & `string_view` are non-owning ptr!

###### 15.3 Containers

a list of containers: not that useful

###### 15.4 Alternatives

4 kinds

- `union`
- `variant<T...>` this is a safer `union`
- `optional<T>` this is a special `variant`, it is just  `variant<A,nothing>`
- `any` just unconstrained `variant`



##### 16-Utilities

###### 16.1 Intro

###### 16.2 Time

skipped, `<chrono>`

###### 16.3 Function Adaption

3 kinds

- lambda
- `std::mem_fn()` before c++11
- `std:function`

###### 16.4 Type Functions ==review==

`<type_traits>`

###### 16.5 `source_location`

mainly for log

- after c++20, `source_location::current()`
- before, use `__FILE__` & `__LINE__` 

###### 16.6 `move()` & `forward()`

they are similar but different

###### 16.7 Bit Manipulation

skipped

###### 16.8 Exiting a Program

4 functions in standard library

- `exit()`
- `abort()`
- `quick_exit()`
- `terminate()`

Note: they don't call dustructors and don't cleanup



##### 17-Numeric (skipped)

###### 17.1 Intro

###### 17.2 Maths

###### 17.3 Numerical Algo

###### 17.4 Complex Number

###### 17.5 Random Number

###### 17.6 Vector

###### 17.7 Numeric Limit

###### 17.8 Type Aliases

###### 17.9 Maths Constants



##### 18-Concurrency

###### 18.1 Intro

###### 18.2 Tasks & `thread`

```c++
// 2 ways
Thread t1 {f};
t1.join(); // wait for thread to complete

// better way
jthread t1 {f}; // no need to join

// pass arg
jthread t1 {f, ref(arg)};

// return value
by passing arg by reference and change it in thread
more in 18.5.1
```

###### 18.3 Sharing Data

`mutex`

- Better to use RAII
- `lock_guard`
- `scoped_lock`
- for multi reader + 1 writer, use `share_mutex` for reader & `unique_lock` for writer

`atomic`

- simple & cheap

###### 18.4 Waiting for Events ==review==

`this_thread::sleep_for`

see example

###### 18.5 Communicating Tasks ==review==

some high-level abstractions in `<future>`

- `future` & `promise` : set some value in `promise` and can be read in `future`
- `packaged_task` : make `future` and `promise` easier to use
- `async()` : forget about above, easy use
- `stop_token` : politely request a `thread` to clean up and go away

###### 18.6 Coroutines ==review==

Still not available in C++20

A coroutine is a function that maintains its state between calls



##### 19-History & Compatibility (Skipped)

###### 19.1 History

see timeline
