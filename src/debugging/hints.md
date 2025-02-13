# Debugging Hints and Strategies
Debugging is often one of the hardest and most time-consuming parts of coding. Don't be frustrated if it takes longer than the actual coding at times; that is all part of the process! Luckily, SML is a type-safe language, so most errors are compilation errors (code does not typecheck) or errors in the logic of the code itself.

## Debugging Compilation Errors

### 1. Get a syntax highlighter
Syntax highlighting is tremendously helpful from both a debugging and readability standpoint. Also, finding typos and syntax errors is much easier with syntax highlighting, simply because the code is much easier to read. Many code editors have syntax highlighting plugins for SML. (For example, in VSCode, there is an extension providing SML language support).

### 2. Anatomy of an error message
<figure class="aligncenter">
    <img src="../assets/errors.png" alt="Instructions" width="1500"/>
    <figcaption><b>Fig 1.</b> Sample error message with a brief explanation of each component of the message. </figcaption>
</figure>

For more specific error messages, it may help to consult the errors page of this website.

### 3. Explicitly Type Annotate
If things do not quite typecheck, one thing that often helps is explicitly type annotating each value used within the function. Sometimes, the type error looks like it occurs on one line, but the real issue may have occurred earlier in the code. Thus, type annotation often helps pinpoint the source of the error, since the explicitly annotated type will fail on whichever line the expected (type-annotated) type does not match the actual type. However, make sure that, in explicitly type-annotating, you do not accidentally restrict the generality of the types (i.e. putting an `int` where you should have an `'a`, which will work in the short term because `int` is more specific than `'a`, but may fail later when something of type `'a` is needed). This can cause more errors later down the line, even if they do work immediately.

## Debugging Code Logic

### 1. Start with small test cases
First, you will want to write simple, small test cases, to test the most basic parts of your function. Identify the simplest combinations of inputs for your function, and test the behavior of the function on these small inputs. If you are working with lists, for example, you may want to write test cases involving `[]` and the singleton list, to test the `[]` and `x::xs` patterns, respectively. If you are working with ints, often the simplest inputs to your function may be `0` and `1`.

The basic structure of a test is as follows:

Suppose we are trying to debug function `f : int -> int`, and we expect `f 1` to evaluate to `0`. Then, we would write our test case as
```
val 0 = f 1
```
If `f 1` does not evaluate to `0`, then the SML compiler will raise the exception Bind when this code is run, since it will be unable to bind the result of `f 1` to the value `0`.

### 2. Write more comprehensive tests
Once you have tested some of the most basic functions, you should write more comprehensive tests. Typically, this involves writing a test for each clause in the code. However, sometimes writing a test for each clause in the code is not enough, since the function may still evaluate to an incorrect value on more complicated outputs. A good rule of thumb is that you should, at minimum, write tests for each function clause, but you will probably have to write more tests depending on the situation.

### 3. Debug "Bottom Up"
Often, your code will either call helper functions, or embed smaller functions within larger ones using `let` and `local` keywords. When debugging, first ensure these smaller, lower-level functions work before debugging top-level functions, as ensuring the correctness of these lower-level functions allows you to rely on their correctness when debugging top-level functions.

### 4. Step through code
Often, simply just walking through each step in the code to ensure it evaluates in the way you expect is useful. Sometimes, it may help to print out values after a certain step in your code. However, print statements should not be your first choice in debugging, as printing has some limitations. Because the SML `print` function has the type `string -> ()`, you can only print out strings. (Note: the `print` function causes _side effects_, i.e. the printing of its input in the REPL, which is why it evaluates to a unit. However, if you haven't learned what _side effects_ are, don't worry- just know that the `print` function returns `()`). In order to print out values that are not strings, you have to convert the value to a string, either by using an SML library function (like `Int.toString`), or by writing your own `toString` function.

Suppose we want to see what some function `f : int -> int` evaluates to on the input `1`. Then, we would write the following code:
```
val () = print (Int.toString (f 1))
```
Then, once the code is compiled and run, the result of `f 1` will be printed out in the REPL.

If you decide to use print statements while stepping through code, you may have to add an extra `let` statement so that you can add the `val` declaration for printing.

In general, print statements in SML can be unwieldy and difficult to use, so print statements are generally not suggested as a way to debug. However, stepping through each line and clause of code is generally very helpful and highly encouraged.

## Debugging Strategies
### Rubber Duck Method
One useful method of debugging is called the "rubber duck method," in which the programmer explains their code, line by line, to a rubber duck (though any object, inanimate or animate, will work equally as well). The premise of this method is that, by explaining what the code is supposed to do and seeing what it actually does, any incongruity between the two becomes apparent and the error is found. Also, this method is helpful because, through explaining, you gain a much better understanding of the concept/code being explained.