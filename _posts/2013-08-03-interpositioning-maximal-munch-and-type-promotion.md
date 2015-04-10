---
layout: post
title: Interpositioning, maximal munch and type promotion
---

Deep C Secrets

-----

C is a great old-school language, but as Peter Van Der Linden shows in his amazing book *Expert C Programming - Deep C Secrets*, C has got a few pitfalls and problems that are easy to get into. For example, it is often seen as a complicated language because of its terrible declaration syntax that only a compiler could love, especially when we're dealing with pointers. But you know what? I just got used to it. It's not a big deal for me anymore. Still, it is both a problem and a non-problem, because after all, declarations follow use. Huh, wait, *declaration follows use*, what the heck is that supposed to mean? So, this code:

```c
int *a;
```

Is saying that `*a` is of type `int`. That's the philosophy behind C declarations: declarations follow use. For example, this:

```c
int (*(*x)())[20];
```

Means that `(*(*x)())[20]` has type `int`. Therefore, `x` is a pointer to a function returning pointer to an array of 20 `int`. Because function calls and array indexing has bigger precedence than `*`, the parentheses are not optional. Had we used `**x()[20]`, this wouldn't even compile, because `x` would be a function returning an array of 20 pointer to pointer to `int`. That doesn't make sense, because functions in C cannot return arrays: they can return pointers, or pointers to arrays.

Also note that the *declaration follows use* rule perfectly fits with a rule that you sould always, always adopt when dealing with pointers in C: do not enter spaces after `*`. Always prefer `int *x` rather than `int* x`. You see, it is irrelevant for the compiler, but visually, something like

```c
int* x, y;
int *a, b;
```

will most likely make you believe that the first line declares two pointers to `int`, `x` and `y`, and the second line declares a pointer to `int`, `a`, and an `int` called `b`. **This is wrong**. `int* x, y` is the same as `int * x, y`, which is the same as `int *x, y`. Only `x` is of pointer type! Thus, in the example above, `x` and `a` are pointers, `y` and `b` are not. Tricky!

Pointers are one of the most hard-to-understand, bug-generator things in C. If you master pointers, you master C. Mastering pointers in C is an art, and if you want to join the club, you must definitely read K&R and then Peter's book. I find it so useful that this article and the next one will be talking about stuff I read in this book (finished yesterday!).

But for now, today, let's settle with 3 different topics: interpositioning, also known as interposing, maximal munch strategy, and type promotion.

Before moving on, I just want to make something clear. From now on, whenever I mention "K&R C" VS. "ANSI C", I am talking about pre-ANSI C vs. ANSI C. pre-ANSI C is usually refered to as K&R C, even though K&R 2nd edition doesn't have any K&R old C anymore (it even has a stamp saying "ANSI C" in the front cover!).

## Interpositioning

If you've been following my posts, you should have an idea of what interpositioning is. As I mention in [my article about linker wrappers]({% post_url 2013-07-13-the-magic-of-wrappers %}), interpositioning refers to the act of supplanting functions in your program with the same name as a standard library function. This is a very dangerous, yet powerful thing to do. As Bjarne Stroustrup said once, 

> C makes it easy to shoot yourself in the foot. C++ makes it harder, but when you do, it blows away your whole leg.

And it is so damn true! This article covers this kind of situations (and yes, it gets worse with C++ when you start messing around with multiple inheritance, for example). Interpositioning is a wonderful tool, but it can create very hard to find bugs. When you create a function with the same name as a standard library function, you are interpositioning. You are placing your own implementation, and it will be used instead of the library one. 

Read that again, because it's important: *it will be used instead of the library one*. Better, yet: *it will ALWAYS be used instead of the library one*. Say that out loud, because this includes functions that are itself part of the library. Allow me to exemplify.

Say there's a function, `foo()`, that belongs to the standard library. You go and create the same exact function, `foo()`, in your program. The C standard stipulates that standard library functions are reserved names, but this is not considered a constraint, so the compiler is not even supposed to warn you about it! In fact, gcc only issues a warning when your function declaration doesn't match the standard function prototype. For example, if you declare `malloc()` as `void *malloc(unsigned int nsize)`, you will get a warning when you try to compile, because `stdlib.h` declares it as `void *malloc(size_t nsize)`. If you change your implementation to match this prototype, no warnings / errors are issued. And what happens next? Your function is silently replacing every call to the old function.

This can be very, very tricky. Back to the `foo()` example: imagine you call `printf()` inside `foo()`. Furthermore, imagine that `printf()` also calls `foo()` under some conditions. But you interpositioned the official `foo()` with your own `foo()`, and you weren't aware that `foo()` is a standard function. And now you really need to call `printf()` inside `foo()`, and you do it, and that's where problems begin to arise. Your `foo()` calls `printf()`, but `printf()` calls `foo()` because it "thinks" it's talking to its good old friend, but it isn't, it's calling YOUR `foo()`, and your `foo()` will call `printf()` again, and this keeps happening recursively forever. Well, technically it is not forever, you will eventually run out of stack space, but you get the point. It stinks. 

At this point, I'm sure you have a question bothering you about the warning for unmatched declaration with prototype. What is supposed to happen when you declare a function prototype, but the arguments types in the function declaration don't match the prototype? Don't worry, we'll get into this in a few moments. Just relax.

My point? Be very, very, very careful with interpositioning. You really must know what you're doing, and it can create very serious bugs. You were warned!

## Maximal munch strategy

Everyone will tell you that you can have as much or as little spaces as you like in your C programs. *Sure*, you think. After all, writing this:

```c
int a;
```

or this:

```c
int                a;
```

is exactly the same. Well, yeah, it is, but take it easy, spaces are not that irrelevant. What about macros that are expanded into the next line? Remember that you can use backslash to expand the declaration of a macro to the next line; every newline that is preceded with `\` will be treated as a single logical line. But if you accidentally type in a space after `\`, you are not escaping a newline anymore, you're escaping a space. So the macro definition will not expand to the next line, and god knows what will happen when you compile it.

Luckily, nowadays, this is a pretty easy to find bug. Even if you don't use `-Wall` (which you should!), gcc will tell you about it with something along the lines of:

```
warning: backslash and newline separated by space
```

If you use a good editor, you will notice that the lines after `\<space>` are not part of the macro (they will have a different highlighting).

But that's no reason to relax. For example, what is the value of `z` in the following code?

```c
#include <stdio.h>
     
int main() {
    int x = 1, y = 1;
    int z = y+++x;
    printf("x: %d, y: %d, z: %d\n", x, y, z);
    return 0;
}
```

Is it `z = y++ + x`, or is it `z = y + ++x`? The ANSI standard specifies a convention that has came to be known as the maximal munch strategy. Maximal munch says that if there's more than one possibility for the next token, the compiler will prefer to bite off the one involving the longest sequence of characters. Thus, in the example above, after reading token `y`, it has to choose between `++` and `+` for the next token, so it ends up choosing `++` - making the whole instruction equivalent to `z = y++ + x`. So the code prints `x: 1, y: 2, z: 2`.

But beware of pathological cases. For example, the instruction

```c
int z = y+++++x;
```

Will be parsed as

```c
int z = y++ ++ + x;
```

and the program will not compile. Because the compiler is greedy, it always prefers the longest possibility for the next token. Even though the only valid combination is `int z = y++ + ++x;`, you can't expect the compiler to figure it out for you. Maximal munch is always there. This is actually funny. Very subtle errors can show up because of this. For example, assume `a` and `b` are pointers to `int`, you could make an integer division by writing

```c
*a/*b;
```

But because of maximal munch, after parsing `*a`, the next token will be `/*` instead of `/`, so it is taken as an opening comment sequence. Surprise! Had you used `*a / *b`, none of this would have happened.

When everybody thought that adding `//` comments to C didn't alter the meaning of any syntactically correct C code, everyone was wrong. That's why `//` comments are not allowed in ANSI C. Considering that `a` and `b` are integers, we can see that this code:

```c
a //*
//*/ b
;
```

is different whether we're considering `//` comments or not. If we are, this translates into `a`. Its value is evaluated and thrown away (useless instruction). On the other hand, if `//` comments don't exist, we will read `a /`, and then `/*` - a comment is openned, and everything is ignored until `*/` is read in the next line, so we end up with `a / b`. So, in ANSI C, where `//` comments don't exist, this is `a/b`. This rule is only enforced in gcc if you compile using the `-ansi` option. If you don't use it, this code will be interpreted as `a`.

You didn't think spaces could make a difference, did you?

## Type promotion

And then we have type promotion. Type promotion is different in ANSI C and in K&R C. Naturally, I will talk about ANSI C's approach.

First, we will have to talk about the usual arithmetic conversions that take place under the hood. Binary operators such as `>`, `==`, `+`, etc, need to make type conversions whenever you give it operands with different types. For example, what happens when you compare an `int` to an `unsigned int`? What's the type of the result? What conversions take place? K&R says this:

> A great many operators cause conversions and yield result types in a similar way. This pattern will be called the "usual arithmetic conversions."

ANSI C uses a value preserving approach, in the sense that it tries to use the floatest, longest operand, signed if possible, without losing bits. I'm not going to quote the standard section that talks about it, because it's a big block of text that gets you lost quickly. Let's do this step by step.

First, you have integral promotion. Integral promotion is performed in every type `T` for which `sizeof(T) < sizeof(int)`. These include both `signed char` and `unsigned char`, `short int`, bit-fields and enumerations. If an `int` can represent every value from the original type, then we convert it to `int`. Otherwise, we convert it to `unsigned int`. `Unsigned int` is guaranteed to always be able to hold every value from anything shorter because, well, it's got more bits and it's unsigned, duh! This is called integral promotion, and that's why code like this:

```c
printf("%zu", sizeof 'A');
```

won't print 1, instead, it will print `sizeof(int)` - probably 4 - because `'A'` is integral-promoted to `int`. Remember how integral promotion works. We will get back into it soon.

Now, for the actual rules, I could just copy-paste the standard and hope that you wouldn't get lost in the way, but it's a pretty dense paragraph with lots of condensed text that only its author could love. Instead, I decided to show it using a decision tree. Type conversion rules can be seen like this:

![Type conversion rules decision tree]({{ site.baseurl }}{{ site.assets }}type_promotion.png)

I know it can be intimidating at first. The easy way to remember it is that you start off with the "biggest floatest" type - `long double` - and while there is no argument that matches it, you keep "decreasing" it by order of magnitude: `long double` -> `double` -> `float`. If none of them matched, you perform integral promotions. At this point, the "smallest" type you have in your expression is an `int` (or `unsigned int`). And now you basically do the same thing you did for `float` types, except that you start with `unsigned long int`. `Double` can't be signed or unsigned, but `int` can, so the "equivalent" to the big-float-guy `long double` is `unsigned long int`. No `unsigned long int`? Then let's take care of a pathological case: the case in which we have a `long int` and an `unsigned int`. This is a special case because depending on the implementation, a `long int` may be able to store every `unsigned int`, but you can't take that for granted. As I said earlier, ANSI C takes a sign preserving approach, where possible, so, naturally, if a `long int` can store every `unsigned int` values, we choose to convert the `unsigned int` to a `long int`. Otherwise, we have no choice but to convert everything (and that's the case where both operands are converted) to `unsigned long int`. It must be `unsigned long int`, because that's the next "upper" type that we know for sure can hold all values from both `unsigned int` and `long int`. Finally, if none of this happened, we fall into the last case, and we can only have `unsigned int` and `int` at this point. Well, if we have `unsigned int`, there's no way we can convert it to `int`, `int` cannot represent all values inside `unsigned int`. So, if we do have `unsigned int`, bad luck, we have to back off on our sign preserving approach and live with the fact that the other operand must be converted to `unsigned int`. What if we don't have an `unsigned int`? Then it must be true that both operands are `int`, and no conversions need to be done. There! That's how you put it in your head. I always like to remember stuff by learning the logic behind it, not by simply memorizing it, especially because I have terrible memory.

An interesting point worth mentioning is that whenever you have an `unsigned long int` or an `unsigned int`, the other operand is converted to `unsigned long int`, or `unsigned int`, respectively. Someone who doesn't know this will have no freaking idea with what's wrong with this code:

```c
int array[] = { 23, 34, 12, 17, 204, 99, 16 };
#define TOTAL_ELEMENTS (sizeof(array)/sizeof(array[0]))
#include <stdio.h>

int main() {
  int d = -1, x = 0; /* x has a value that doesn't occur in array */
  /* ... */
  if (d <= TOTAL_ELEMENTS-2)
    x = array[d+1];
  printf("%d\n", x);
  return 0;
}
```

The `sizeof` operator returns an unsigned type - `size_t` to be more precise - thus, the `if` test fails because `d` is converted to `unsigned int`, and `-1` in a 2's complement notation is represented by every bit set to 1. On a machine with 32-bit integers and 2's complement, `-1` converted to `unsigned int` is interpreted as `4294967295`, so the test fails. Kaput. Gone. Forget it. The code prints `0`, instead of `23`. The correct way to do it is to cast `TOTAL_ELEMENTS-2` to `int`.

By the way, if you're wondering how to see the real type behind `size_t`, you can compile a stupid program that just includes the header file with its declaration:

```c
#include <stddef.h>

int main() {
  return 0;
}
```

And then:

```
gcc -E test.c | grep 'typedef .* size_t;'
```

This assumes your program is saved as `test.c`. It uses gcc's super-useful option `-E`, that forces it stop after preprocessing. I find this technique much faster and efficient than digging through the system header files to find what `size_t` is. If you try to do that, you will see that it's not really defined in file X, but instead file X includes file Y, so it must be there, but file Y doesn't define it and includes file Z and T, and, oh god ... just give up on that.

And finally, we have type promotion. Type promotion takes place in any expression. You see, the rules I told you above are the conversions that take place when an operation is invoked with operands of different types, and the final type is normally the type of the result, but not always. Apart from integral promotions, `float` is promoted to `double`, and array of `T` is promoted to pointer to `T`.

A function argument is an expression, so, type promotion occurs in function calling as well, unless you use ANSI prototypes / function declarations, and not the old, deprecated K&R function definition and declaration. There is probably nobody writing fresh new C code with old K&R functions, but the take home message is that type promotion does not happen when ANSI prototypes or function declarations are used. Either way, optional arguments in functions are always promoted. That's why you can use `%f` in `printf()` to print both a `float` and a `double`, and it magically works because if you pass it a `float`, it will be promoted to `double`. And that's also the reason why calling `va_arg()` with `short`, `char`, `unsigned char`, and other types smaller than `int` is an error - all of these are promoted to `int`, so the smallest thing `va_arg()` can give you is an `int`. If you want a `char`, you must remove the bits that were added in type promotion by casting it back to the original type *after calling `va_arg()`*. That's how it's done when you use `%c` on `printf()` to print a `char`. You pass it a `char`, it's converted to `int`, `printf()` picks it as `int`, and casts is back to `char` when it sees `%c`, cutting the extra bits that were added along the way. This may seem stupid, but it works that way to make the compiler's work easier.

By now, we can get back to an earlier question. What happens when your function declaration doesn't match the prototype? Every call you make uses the function's prototype to pass the correct type arguments. Imagine we had this in `file1`:

```c
int foo(int, char *);

int main() {
  foo(52, "hello");
  return 0;
}
```

And this in `file2`:

```c
#include <stdio.h>

int foo(unsigned long long int a, char *b) {
  printf("%llu %s\n", a, b);
  return 0;
}
```

When you compile a binary file using `file1` and `file2`, the code in `file1` calls `foo()` without any type promotion. In this particular case, it pushes `52` as an `int` and then a pointer to the string `hello`. In `file2`, `foo()` thinks you pushed an `unsigned long long int` into the stack, so it pops an `unsigned long long int`, even though you just put an `int`. What happens? Well, it ends up "eating" part of the next argument, our character pointer. It prints garbage. 

It is very important to ensure that prototypes match with function declaration, especially when functions are declared in other files. Had we declared everything in the same file, gcc would complain about incompatible types and abort compilation. When you split functions across files, there's no way to check argument matching in compile-time (things only get together in the linking phase), so it is easy to create hidden, weird bugs. This is why everyone declares the functions in a single, common header file that is included everywhere. If you include it in the file where you implement it, the compiler can easily catch incompatible prototypes and definitions.

It has been a pleasure to read Peter's book and learn stuff like this. It is really enlightning. You can't call yourself a C expert without knowing this stuff. Learning C (and I mean learning, not "learning") enables you to learn any language without struggling too much. Java? Well, once you master object oriented programming concepts, Java is easy. C#? No problem. The thing about C is that it is complicated enough to format your brain to quickly learn and evolve with a moderately accessible language.
