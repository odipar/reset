---
layout: post
author: Robbert van Dalen
title: "Relational algebra"
---
After receiving some excellent [feedback](https://old.reddit.com/r/ProgrammingLanguages/comments/uukxvp/reset_set_based_programming_language/) on Reddit, I learned that both ReSeT and CUE are relational at their core.
This wasn't immediately obvious to me, nor the CUE community. 

So that's why I will show and explain, what I believe, are very interesting logical properties of both ReSeT and CUE.

Let's start with 4 example 'relations' `R1`, `R2`, `R3` and `R4`, each containing one 'tuple':
```javascript
R1: {a: 1}
R2: {a: 2}
R3: {a: 3}
R4: {a:>1}
```
Relation `R4` is already interesting. Although we only count 1 tuple, `R4` *represents* an infinite set of tuples with `a > 1`.

#### Union
We can build more complex relations, by taking the union, or by *joining* them together with `|`:
```javascript
R5: R1 | R2 | R3 => {a:1} | {a:2} | {a:3}
```
Note that `R123` is a relation itself.
                    
#### Least Upper Bound
But it seems that nothing really happened! That's because both CUE and ReSeT must adhere to the Least Upper Bound (LUB) 
property of their relational lattices. 
So in the case of `R5`, we cannot reduce to a smaller relation, otherwise we would break the LUB property.

Of course, one can think of other upper bounds, for example `U1` and `U2`:
```javascript
U1: {a: 1|2|3}
U2: {a: >=1 & <=3}
```
#### Intersection
Interestingly enough, we can join two relations by intersecting them with `&`.
This exactly follows the 'natural join' definition that can be found in [relational algebra](https://en.wikipedia.org/wiki/Relational_algebra). 

Here is an example:
```javascript
R54: R5 & R6 => ({a:1} | {a:2} | {a:3}) & {a:>1} => 
                {a:1} & {a:>1} | {a:2} & {a:>1} | {a:3} & {a:>1} =>
                {a:1 & >1} | {a:2 & >1} | {a:3 & >1} =>
                {a:_|_} | {a:2} | {a:3} =>
                _|_ | {a:2} | {a:3}
                {a:2} | {a:3}
```
Notice that tuples that contain `_|_` will be reduced to `_|_` and ultimately dropped from the disjunction. 
#### Greatest Lower Bound
There are two important rules to remember when intersecting relations:

* `&` distributes over `|`
* `&` distributes over tuples/values

By abiding to these rules, the *meet* operator `&` adheres to the Greatest Lower Bound property of ReSeT's relational lattice.
#### Complex joins
Now let's try to intersect some more interesting relations. Here is working [example](https://cuelang.org/play/?id=TCFw0ZL5DN9#cue@export@cue) that I created in CUE.
```javascript
A: 
 {a:0, b:1} | 
 {a:0, b:2} | 
 {a:1, b:1} | 
 {a:1, b:2}
 
B: 
 {a:0, c:2 } |
 {a:1, c:<4}

C: 
 {a:>0, b:>1}

AB: A & B => {a:0, b:1, c:2} | {a:0, b:2, c:2} | {a:1, b:1, c:<4} | {a:1, b:2, c:<4}
  
AC: A & C => {a:1, b:2}
  
BC: B & C => {a:1, b:>1, c:<4}
```
Note that relation `A` and `B` intersect only on key `a`, so `AB` can be considered a *single-key join*.
Similarly, `AC` can be considered a *multi-key join*, as relation `B` and `C` intersect on both keys `a` and `b`. 

Another interesting interpretation of `AC` is that `C` **filters** `A`, given the constraints `a>:0, b:>1`.
All this follows naturally from how `&` is defined!

#### Negation
Now things get a bit more hairy when we also allow the negation `!` of relations. 
In my previous [post](https://odipar.github.io/reset/2022/05/20/introduction.html), I've already covered negation a bit, but how does that work with relations?

To understand negation, we need to turn to the canonical representation of values in ReSeT. 

Here is the (slightly abbreviated) canonical representation of `{a:1, b:2}`:
```javascript
{
 <a     : _
 a      : 1
 >a & <b: _
 b      : 2
 >b     : _ 
}
```
Now if we take the negation `!{a:1, b:2}` we get:
```javascript 
{
   <a     : !_
   a      : !1
   >a & <b: !_
   b      : !2
   >b     : !_ 
} =>
{
   <a     : _|_
   a      : <1|>1
   >a & <b: _|_
   b      : <2|>2
   >b     : _|_
}
```
So `!` distributes over values. Does that mean that for any value `A`, double negation `!!A` can always be rewritten to `A`?
Well, it turns out that this is not the case.
#### De Morgan
Because ReSeT must follow the lattice rules, it follows that negation must be a 'weak' form of negation.
We can see why, when we look at the following De Morgan law:
```javascript
!(A | B) => !A & !B
```
then, if we apply De Morgan on `A` we see that `!!A` is not equal to `A`:
```javascript
A: {a: 1} | {a: 2}

B: !A => 
  !({a: 1} | {a: 2}) => (applying De Morgan) 
  !{a: 1} & !{a: 2}  => 
  {<a: _|_, a: <1|>1, >a: _|_} & {<a: _|_, a: <2|>2, >a: _|_} =>
  {<a: _|_, a: <1|>1 & <2|>2, >a: _|_}

C: !B => 
  {<a: _|_, a: <1|>1 & <2|>2, >a: _|_} => 
  {<a: !_|_, a: !(<1|>1 & <2|>2), >a: !_|_} =>
  {<a: _, a: 1|2, >a: _} =>
  {a: 1|2}
```
That said, `!!A` is still an upper bound of `A` which is good enough.
#### Heyting algebra
From what I understand from logic, negation in ReSeT is similar to weak negation in a [Heyting algebra](https://en.wikipedia.org/wiki/Heyting_algebra). 
In this algebra, the law of excluded middle doesn't hold. I find this very appealing as I'm a finitist at heart :).

So is ReSeT a Heyting algebra on relations? I'm not sure, but my intuition says: yes!

#### What's next?
In my next post I will delve deeper into ReSeT's evaluation model. Until then!