# Common Errors

In this section, we will explore some common mistakes and errors that may occur
when programming in Standard ML.

## Casing Issues
### Match Nonexhaustive
Though the code will still compile and run with this warning, it often indicates that the logic in your code does not cover all cases. It is always a good idea to check if you are missing some cases in your thinking, or if the warning comes from a scenario that will never happen according to the function specification. Even if there is a case that may not happen according to the function specification, it may be useful to raise an exception in the "impossible" case to get rid of this warning, because if your code somehow violates the specification and ends up in this case, then you can catch the error and debug it.

### Nested Cases
When you nest case expressions within case expressions, it's good to wrap your case statements with parentheses. SML will continue to look for patterns to case on, so using parentheses will let it know when to "stop".

For example, the following code will compile without warnings:
```
case 0 of
     0 => (case "x" of _ => 3)
   | _ => 5
```
but this code will have a match redundant error and a match nonexhaustive error:
```
case 0 of
     0 => case "x" of _ => 3
   | _ => 5
```
Obviously, this example is very contrived, but adding parens may help in nested case expressions.

## Associativity Issues
Since function application is left associative, making sure you have correct parenthesization is very important. It is almost always a good idea to double check your parenthesization in your code, since it can cause very confusing bugs, but can be fixed with a simple check.

For example, the following code will not compile:
```
fun f [] = 0
  | f x::xs = 1
```
This fails to compile because function application is left-associative, so the SML compiler thinks that `x` is the only argument to the function `f`, and does not know how to parse the extra `::xs`. A fix for this issue would just to put parens around the `x::xs`, like so:
```
fun f [] = 0
  | f (x::xs) = 1
```
Similarly, if some expression is an argument to another function, it is usually good to put parens around that expression. For example, suppose we had some functions `f` and `g` of type `int -> int`, and we would like to apply the function `g` to the value `f 1`. Then, if we wrote `g f 1`, this would not typecheck, since function application is left-associative (writing `g f 1` would be the same thing as writing `((g f) 1)`). To fix this, we would write this as `g (f 1)` to fix the associativity issues.

## Equality Type Warnings (i.e. `''a` vs `'a`)
Sometimes, code will fail to typecheck because it expects something of type `'a` but instead gets something of type `''a` instead. A plain type variable like `'a` can be substituted with any type, but something like type `''a` (with two apostrophes in front) can only be substituted with an _equality_ type. An _equality_ type is a type that can use operators like `=` and `<>` to compare their values (`int`, `string`, and `bool` are good examples of equality types). Thus, if your code has an `''a` instead of an `a`, it is likely that you are using `=` or `<>` to compare values.

For example, the following function has type `''a list * 'a -> bool`:
```
fun contains (x, []) = false
  | contains (x, y::ys) = x = y orelse contains (x, ys)
```
Because `x` and `y` are compared by `=` in the function, then any inputs into the `contains` function must consist of equality types, so the type is `''a list * ''a -> bool`.

However, we might want a function that takes in an `'a list` instead because it is more general. We still need some way of comparing whether two elements are equal or not, so we will pass in an extra parameter to compare two elements. We can rewrite the function as follows:
```
fun contains (cmp, x, []) = false
  | contains (cmp, x, y::ys) =
    case cmp (x, y) of
      EQUAL => true
    | _ => contains (cmp, x, ys)
```
The type of the function will now be `('a * 'a -> order) * 'a list * 'a -> bool`.

In general, it's preferable to use `case` expressions instead of `if-then-else` statements with `=` in the condition.

## ?.t Type Errors

```
errors.sml:9.7-10.18 Error: right-hand-side of clause does not agree with function result type [tycon mismatch]
  expression:  ?.t
  result type:  ?.t
```

Clearly, this is not a very helpful error message. The code that induces this
error is shown below.

```sml
signature T =
sig
  type t val x : t
end

functor Foo (structure A : T
             structure B : T) =
struct
  fun bar (0 : int) = A.x
    | bar n = B.x
end
```

The reason for this error is because the compiler does not distinguish (in terms
of name) structures that are given as argument to the functor `Foo`. Both are
referred to by `?`. In other words, it is saying that, on the second line of
`bar`, it expected a value of type `A.t` to be returned, but received one of
type `B.t`, namely `B.x`. Since `A` and `B` do not really have names, however,
it just referred to either structure as `?`, causing the confusing error.

If you receive this error, check your structures to make sure that you are not
conflating types from different structures.

## Re-declaring Datatypes

Some datatypes are already present in the SML Basis Library, meaning that you do
not have to declare them, as they are already present at the top level.
Re-declaring datatypes anyways, however, can cause type issues that are somewhat
difficult to debug. Consider the following code:

```sml
fun opt_wrap x = SOME x

datatype 'a option = NONE | SOME of 'a

fun wrap_again (x : int) : int option = opt_wrap x
```

Although it looks innocuous, we will run into the following type error:

```
errors.sml:5.5-5.51 Error: right-hand-side of clause does not agree with function result type [tycon mismatch]
  expression:  int ?.Assembly.option
  result type:  int option
```

The reason for this is because the declaration of `'a option` on the third line
creates a new type `'a option` that "shadows" the basis' definition. This means
that on line 1, `SOME` is of the type of the original option, whereas the
function `wrap_again` is type-annotated to expect a value of the type of the
_new_ option.

In general, re-declaring datatypes is a bad idea that will cause conflicts down
the road, as it makes your code incompatible with any other code expecting the
original datatype, such as autograders. Be sure to not re-declare datatypes that
already exist to avoid this issue.

## Forgetting a Bar

Recall that a bar `|` is required to delimit different clauses in a function
definition. For instance, take the following (incorrect) implementation of the
`fact` function:

```sml
fun fact (0 : int) : int = 1
    fact n = n * fact (n - 1)
```

Compiling this will result in the error:

```
errors.sml:2.24 Error: unbound variable or constructor: n
errors.sml:2.14 Error: unbound variable or constructor: n
errors.sml:2.10 Error: unbound variable or constructor: n
errors.sml:1.29-2.30 Error: operator is not a function [overload - bad instantiation]
  operator: 'Z[INT]
  in expression:
    1 fact
errors.sml:1.6-2.30 Error: right-hand-side of clause does not agree with function result type [tycon mismatch]
  expression:  bool
  result type:  int
  in declaration:
    fact =
      (fn 0 : int =>
            (1 fact) <errorvar> = <errorvar> * fact (<errorvar> - 1): int)
```

If you see `unbound variable or constructor` errors where they don't make sense,
and should not be unbound, it may be the case that you are having a deeper
syntax issue!

## Let in without an end

Consider the following code snippet:

```sml
fun foo x =
  let
    val y =
      let
        val z = 3
      in
  in
    4
  end
```

Compiling this will result in the error:

```
errors.sml:6.7-6.9 Error: syntax error: replacing  IN with  SEMICOLON
errors.sml:11.1 Error: syntax error found at EOF

uncaught exception Compile [Compile: "syntax error"]
  raised at: ../compiler/Parse/main/smlfile.sml:19.24-19.46
             ../compiler/TopLevel/interact/evalloop.sml:45.54
             ../compiler/TopLevel/interact/evalloop.sml:306.20-306.23
```

due to the missing `end` at the end of the inner `let`. In general, an error
that says "replacing" is trying to signal that you have probably put an
unexpected form of syntax where it shouldn't be - look at precisely what is
missing to determine what the error is.
