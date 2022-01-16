# **`Functional Programming Notes`**

# Table of contents

- [**Functional Programming Concepts**](#functional-programming-concepts)
    - [**First-class functions**](#first-class-functions)
        - [**Closures**](#closures)
    - [**Pure functions**](#pure-functions)
        - [**Memoization**](#memoization)
        - [**Lazy evaluation**](#lazy-evaluation)
    - [**Recursion**](#recursion)
    - [**Referential transparency**](#referential-transparency)
    - [**Currying**](#currying)

# **Functional Programming Concepts**

## **First-class functions**

A programming language is said to have first-class functions when the functions in that language
are treated like any other variables; can be passed as an argument to other functions, it can 
be returned by another function, and it can be supported by a variable.

**Example functions as argument in Javascript:**

```javascript
function sayHi() {
    return "Hola ";
}

function greet(greeting, name) {
    console.log(greeting() + name);
}

// Pass `sayHi` as argument to `greet`
greet(sayHi, "JavaScript!");
```

**Example assign a function(anonymous) to a variable in Go:**

```go
package main

import (  
    "fmt"
)

func main() {  
    a := func() {
        fmt.Println("hello world first class function")
    }
    a()
    fmt.Printf("%T", a)
}
```


### **Closures**

A closure **is a function that stores references to the adjacent state (lexical scope)**.
In other words, a closure allows access to the scope of an outer function from an inner 
function. 

**In JavaScript**, closures are created every time a function is created.

```javascript
function makeFunc() {
  var name = 'Mozilla';
  function displayName() {
    alert(name);
  }
  return displayName;
}

var myFunc = makeFunc();
myFunc();
```

At first glance, it might seem unintuitive that this code works. In some programming languages,
the local variables within a function exist for just the duration of that function's execution.
Once `makeFunc()` finishes executing, you might expect that the name variable would no longer be
accessible. However, because the code still works as expected, this is obviously not the case
in JavaScript.

**In Go**, closures are a special case of anonymous functions.

```go
package main

import (  
    "fmt"
)

func main() {  
    a := 5
    func() {
        fmt.Println("a =", a)
    }()
}
```

In the program above, the anonymous function accesses **the variable _a_** which is present outside
its body in line #10. Hence, this anonymous function is a closure.

**Closures in Rust** are anonymous functions with a nice syntax. A closure `|x| x + 2` takes an 
argument and returns it with `2` added. Note that we don't have to give types for the arguments 
to a closure (they can usually be inferred). We also don't need to specify a return type. If we
want the closure body to be more than just one expression, we can use
braces: `|x: i32| { let y = x + 2; y }`. We can pass closures just like functions: _bar(|| 42)_.

```rust
let x = 42;
bar(|| x);
```

Note how `x` is in scope in the closure.

E.g., to add a value to each element of a vector:

```rust
fn baz(v: Vec<i32>) -> Vec<i32> {
    let z = 3;
    v.iter().map(|x| x + z).collect()
}
```
Here `x` is an argument to the closure, each member of `v` will be passed as an `x`. `z` is 
declared outside the closure, but because it's a closure, `z` can be referred to.


**Uses for Closures:**

Closures are useful because they let you associate data (the lexical environment) with a 
function that operates on that data. This has obvious parallels to object-oriented programming,
where objects allow you to associate data (the object's properties) with one or more methods.

Consequently, you can use a closure anywhere that you might normally use an object with only a 
single method.

## **Pure functions**

Pure functions are the atomic building blocks in functional programming. They are adored for
their simplicity and testability.

A function must pass two tests to be considered “pure”:

* Same inputs always return same outputs
* No side-effects

This is how functions in math work: `Math.cos(x)` will, for the same value of `x`, always 
return the same result. Computing it does not change `x`. It does not write to log files, do
network requests, ask for user input, or change program state.

When a function performs any other “action”, apart from calculating its return value, the 
function is impure. It follows that a function which calls an impure function is impure as 
well. Impurity is contagious.

A pure function can only access what you pass it, so it’s easy to see its dependencies. We
don’t always write functions like this. When a function accesses some other program state,
such as an instance or global variable, it is no longer pure.

### **Memoization**

Memoization or memoisation is an optimization technique used primarily to speed up computer
programs by storing the results of expensive function calls and returning the cached result
when the same inputs occur again.

Because pure functions are referentially transparent, we only need to compute their output once
for given inputs. Caching and reusing the result of a computation is called memoization, and
can only be done safely with pure functions.

### **Lazy evaluation**

Invoking a pure function means you specify a dependency: this output value depends on these
input values. But what if you never use the output value? Because the function can not cause 
side effects, it does not matter if it is called or not. Hence, a smart system can be lazy and 
optimize the call away.

Some languages, like _Haskell_, are completely built on _lazy evaluation_. Only values that are
needed to achieve side effects are computed, the rest is ignored.

_Haskell:_

```haskell
add x y = x + x
```
`add 4 5`

**Result : 8**

`add 10 (89/0)`

**Result : 20**

_Python:_

```python
def add( x , y ) :
    return x + x
```

`print( add(4 , 5) )`

**Result : 8**

`print( add(10 , (89/0) ) )`

**Result : Traceback (most recent call last):
File “test.py”, line 7, in <module>
print( add(10 , (89/0) ) )
ZeroDivisionError: division by zero**

## **Recursion**

Recursion is the process of defining a problem (or the solution to a problem) in terms of
(a simpler version of) itself.

For example, we can define the operation "find your way home" as:

* If you are at home, stop moving.
* Take one step toward home.
* "find your way home".

Here the solution to finding your way home is two steps (three steps). First, we don't go home 
if we are already home. Secondly, we do a very simple action that makes our situation simpler
to solve. Finally, we redo the entire algorithm.

The recursion technique is prone to stackoverflow problem, because each recursion call adds
context variables to the call stack.

## **Referential transparency**

In maths, referential transparency is the property of expressions that can be replaced by other
expressions having the same value without changing the result in any way.

In programming, referential transparency applies to programs. As programs are composed of 
subprograms, which are programs themselves, it applies to those subprograms, too. Subprograms
may be represented, among other things, by methods. That means method can be referentially
transparent, which is the case if a call to this method may be replaced by its return value:

```java
class Sample{
int add(int a, int b) {
  return a + b;
}

int mult(int a, int b) {
  return a * b;
}

int x = add(2, mult(3, 4));
}
```
## **Currying**

Currying is the process of transforming a function that takes multiple arguments in a tuple as 
its argument, into a function that takes just a single argument and returns another function 
which accepts further arguments, one by one, that the original function would receive in the 
rest of that tuple.

```haskell
f :: a -> (b -> c)     -- which can also be written as    f :: a -> b -> c
```

and it's curried form:

```haskell
g :: (a, b) -> c
```
Both forms are equally expressive. It holds:

```haskell
f x y = g (x,y)   
```

However, the curried form is usually more convenient because it allows partial application.

# **References/Bibliography**

>[1]
>«MDN Web Docs». [Online]. Disponible en: https://developer.mozilla.org. [Accedido: 16 de enero de 2022]
>
>[2]
>«HaskellWiki». [Online]. Disponible en: https://wiki.haskell.org/Haskell. [Accedido: 16 de enero de 2022]
