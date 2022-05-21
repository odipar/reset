---
layout: post
author: Robbert van Dalen
title: "An introduction"
---
I like to think about alternative ways of expressing computations.
With the Reset language, I'm experimenting with a new approach that is based on sets.

Reset is greatly inspired by the CUE language, which I stumbled upon early this year.
What sets Reset apart from CUE however, is that Reset is more focussed on semantics, and less on use cases.
Please take a look at the excellent CUE documentation to get a better idea about the theory behind Reset.

#### Sets
Let's first start with a fundamental property of all values in Reset:

`every value in Reset is a Set of key:value pairs`

..or alternatively: 

`every value in Reset is a Map that maps keys to values`

To make Reset easier to implement, I've constrained keys to be values of type `boolean`, `number` or `string`
(this restriction may be removed in the future).

#### Types are values
Like CUE, types are values in Reset. 
And like CUE, I believe this set-theoretic approach to types to be both natural and powerful.

Let's see!
                                                                                         
#### Numbers
In Reset, `number` represents the *infinite* Set of all numbers. 
Likewise, `1` is synonymous with the *finite* Set that contains only the single number `1`.
We can also specify other infinite Sets of numbers, for example, all numbers that are greater or equal to 3 (`>=3`).

It is important to understand that, although you can specify infinite Sets in Reset, you can only do so 'constructively'.
For example, you cannot construct the set of all sets (leading to a paradox), 
because Reset doesn't allow recursion or loops, but more about that in later posts.

#### Set operators
We can create other number sets by combining 3 set operators, AND `&`, OR `|` and NOT `!`.

Here are some examples:
```javascript
1|2|3|4 & 3|4|5|6 == 3|4
>2 & number == number
number & !4 == <4|>4
>4 | >8 == >8
!(1|2) & number == <1|(>1 & <2)|>2
1 & 2 == _|_
```           
Notice the bottom(`_|_`) value. Similar to CUE, this value indicates the empty Set (of numbers).
So to check whether a value `v` is of a certain 'type' `t` we just need to intersect `v & t`.
If the result is `_|_` we know that `v` is not of 'type' `t` (or `v` is not a subset of `t`).

#### Number operators
By applying the cartesian product on two sets, number sets can be added, multiplied and divided. 

Here are some examples:
```javascript
1|2|3 * 3 == 3|6|9
1|2|3 + 10|20|30 == 11|12|13|21|22|23|31|32|33
>3 + >8 == >11
>3 * >8 == >24
4 / 6 == 2 / 3
<10 / >2 == <5
3 / 0 == _|_
```
To prevent loss of precision with division, I've decided single numbers to be rationals in Reset,
with division by zero resulting in `_|_` (a natural result, in my opinion).

#### Strings
The *infinite* Set of all strings is represented by `string` in Reset.
Strings work similarly as numbers, as you can specify single strings, or infinite sets of strings.
As with numbers, set operators can use to create new string sets:

```javascript
"a"|"b"|"c" & "b"|"c"|"d" == "b"|"c"
>"a" & string == >"a"
string & !"d" == <"d"|>"d"
!("a"|"b") & string == <"a"|(>"a" & <"b")|>"b" 
"a" & "b" == _|_
```
               
There are currently no other operators defined on strings, except set operators.

#### Booleans
And finally we have the *finite* `boolean` set, which contains `true` and `false`. 

#### Top
Top `_` represents all possible values in Reset, currently the *union* of `boolean | number | string`. 
An interesting property of `_` is that its negation is `_|_` and vice-versa.
Note that the *intersection* of `boolean & number & string` is `_|_`.

#### Maps
Remember that every value in Reset is a Map that maps keys to values (a set of key:value pairs). To define a
map in Reset, you need to create a list of key:value pairs and put them in between curly braces:

```javascript
{
  true: <4
  0: "a"
  1: "b"
  "x": "y"
  >"x": false
}
```
Note that keys are sets themselves, but keys contained by the pairs in a map should not intersect (if they do, they are merged into another key:value pair).

#### Canonical representation
I've already said that every value is a Map in Reset, but what about `boolean`, `number` and `string`: are they Maps too?
Yes they are, and to understand why, we need to switch over to canonical representation mode.

Here is a canonical representation of the number set `1|2`:
```javascript
{
 boolean: _|_
 <1: _|_
 1 : _
 >1 & <2: _|_
 2 : _
 >2: _|_
 string: _|_
}
```
.. and a canonical representation of `number`:
```javascript
{
 boolean: _|_
 number: _
 string: _|_
}
```
Another way of looking at it is that a map represents a *set membership function* `f(x) -> boolean` 
with `true` (*is* member) and `false` (is *not* member) being mapped to `_` and `_|_`.

Of course this is not a new idea. Have a look at [ordset](https://github.com/earogov/ordset) and [intervalmap](https://github.com/rklaehn/intervalset/blob/master/IntervalMap.md) for some related work.

#### A canonical intersection
Now let us look at a simple CUE program and translate that to a canonical Reset representation. Note that, unlike Reset, CUE doesn't allow
keys other than 'concrete' strings. In that sense, Reset is a superset of CUE (if we don't take the other CUE data types into account)
.

Here is a simple CUE program:
```javascript 
{ a: 1 + 2, b: 3 * 4 } & { a: >2, b: 10|11|12, c: 3 } == { a: 3, b: 12, c: 5 }
```
The internal Reset representation of that same program would be:
```javascript
{
 boolean: _
 number: _
 <"a": _
 "a": 1 + 2
 >"a" & <"b": _
 "b": 3 * 4
 >"b": _
} & {
 boolean: _
 number: _
  <"a": _
 "a": >2
 >"a" & <"b": _
 "b": 10|11|12
 >"b" & <"c": _
 "c": 5
 ">c" _
}
```
When Reset intersects two maps, it will intersect all the values between the two maps, that have intersecting keys.

```javascript
{
 boolean: _ & _             == _
 number: _ & _              == _
  <"a": _ & _               == _
 "a": (1 + 2) & >2          == 3
 >"a" & <"b": _ & _         == _
 "b": (3 * 4) & (10|11|12)  == 12
 >"b" & <"c": _ & _         == _
 "c": _ & 5                 == 5
 ">c" _ & _                 == _
}
```
#### What's next
Of course, there is much more to Reset (and CUE!) than just merely the manipulation of sets of values, with the next most interesting feature being references and indexing.

With references and indexing, you can build abstractions and re-use them somewhere else. 
References also introduce a 'spreadsheet-like' computational model with surprising consequences. That said, references are also very hard to implement correctly,
as I will show in my next posts.

Until then!