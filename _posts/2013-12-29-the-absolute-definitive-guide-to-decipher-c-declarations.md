---
layout: post
title: The absolute, definitive guide to decipher C declarations
---

How to read C declarations and not get confused

-----

Prepare yourself, it is going to be a long journey today. Once and for all, we're going to make an in-depth analysis of C declarations: where they come from, why they made it that way, how to read them, and last, but not least, how to make a program to read them (this is going to be the fun part). This article is going to be longer than usual, so you may want to be sure you're ready for the battle. Get some sleep, have a walk, go to the gym, whatever, just relax, and read this with a fresh mind. Take it easy on the topics, it is probably too much information to assimilate at once. Don't read everything from top to bottom - read a bit, then think about it. Make sure you understand it. There's no rush. If you want to finally understand how to read complicated declarations such as this one:

```c
char **(*(*(*x)[100])(int, char *, double *const**, void (*)(int **, char [])))[50];
```

Or even this one:

```c
unsigned const char volatile (*const(*volatile(*volatile const a)(unsigned int (*const)(const char, int, float), char *const*))[12])(int (**)[50]);
```

Or, a more real world example (taken from the telnet sourcee):

```c
char *const *(*next)();
```

Then stay with me. You will soon become a pro in C declarations, and the code above will not look like chinese to you. I am writing this article after reading *K&R*, *Expert C Programming - Deep C Secrets*, and *C - A Reference Manual*. These books cover C constructs that even experienced C programmers find difficult to understand. Therefore, this article is about how I see C after reading these amazing books.

Back in the 70's, when Kernighan and Ritchie were writing The C Programming Language, they were pretty honest about how declarations can quickly become unreadable by the untrained eye:

> C is sometimes castigated for the syntax of its declarations, particularly ones that involve pointers to functions. The syntax is an attempt to make the declaration and the use agree; it works well for the simple cases, but it can be confusing for the harder ones, because declarations cannot be read left to right, and because parentheses are over-used. [...]

(K&R section 5.12, page 122, 2nd edition)

C's declaration model dates back to the late 60's, when type models were a recent idea. C's grandfather, BCPL, was very type poor: its only data type was a binary word. Type models were a weird thing at that time, so C was born in an unstable world where there was just a vague idea of what advantages the use of type models could bring. At that ancient time, K&R made C adopt the philosophy that declarations should follow use, and it kind of seemed like a good idea. This means that the declaration

```c
int *x;
```

Declares a pointer `x` to an integer. Declaration follows use means that `*x` is of type `int`, hence, `x` is of type pointer to `int` (this is why you should never put spaces between the star and the identifier, or worse yet: don't even consider writing `int* x`). This decision makes sense: C declarations are relatively easy for a compiler writer to parse. When C was created, its primary customers were also its creators, so they had a strong reason to build a language easy to parse. On the other hand, it may have not been a brilliant idea. The rule that declarations follow use combined with precedence rules implies that we can't read a declaration left-to-right anymore. We, humans, are used to reading things left-to-right, and that is one of the primary reasons so many people have problems understanding C's declarations (the other reason is that they don't know the precedence rule, which I will discuss in a second). Peter Van Der Linden goes even further and suggests an alternative:

> The idea that a declaration should look like a use seems to be original with C, and it hasn't been adopted by any other languages. Then again, it may be that "declaration looks like use" was not quite the splendid idea that it seemed at the time. What's so great about two different things being made to look the same? The folks from Bell Labs acknowledge the criticism, but defend this decision to the death even today. A better idea would have been to declare a pointer as 
>
> int &p;
>
> which at least suggests that p is the address of an integer. This syntax has now been claimed by C++ to indicate a call by reference parameter.

It's a decision that had to be made; as usual, it's got some good points and some bad points. It's too late to revert that, and it is certainly useless to discuss it. While I understand that C grew in a deficient base and only compilers can love its syntax, I have to confess that I have a secret fetish for C declarations. They're cool, and it feels good to master them. And, after all, that's why I'm writing this.

Before moving on, we should look at some areas of C that are normally a little blurred among non experienced C programmers. C's declarations model got worse with the introduction of keywords like `const` and `volatile`. These keywords can only be used in a declaration, which means that "declaration follows use" is not really 100% true anymore.

## The confusion around `const`

`const` is used to denote a **read-only variable**. It does not mean that the variable cannot be written: that wouldn't be safe to assume, since we may have pointers to this variable that are free to modify it, or any other level of indirection. Or it can be the other way around, i.e., the variable itself can be written, but we are accessing it through a pointer to a value that is read-only, so we can't modify what the pointer references, but if we modify the original variable, changes will be visible when we dereference the pointer. `const` is often a source of confusion, and understanding it is obviously part of understanding a C declaration. For example, consider these 3 declarations:

```c
const int *x; /* 1 */
int *const x; /* 2 */
const int *const x; /* 3 */
int const *x; /* 4 */
```

The rule for understanding declarations with `const` is very simple: if it's preceded by `*`, it means that the pointer is read-only, so you can't change what it points to, but the *content* referenced by the pointer can be changed. This is the case for the 2nd declaration - we can assign to `*x`, but we can't assign to `x`. 

If the `const` keyword is not preceded by a pointer, then it means that the value is read-only. The 1st and 4th declarations are examples of this case, and they are both equivalent, it's irrelevant whether const is written before or after the type. In these declarations, we can assign to `x`, but we can't assign to `*x`.

And then of course, we can combine this - both the pointer and what it points to can be read-only, yielding a declaration like the 3rd one. Can you explain the following declaration?

```c
char **const *x;
```

It is a nice exercise to mentally parse this declaration and see what's going on here. Well, if it was:

```c
char ***x;
```

Then we would know for sure that `x` is a pointer to pointer to pointer to `char`. There's a `const` keyword together with the second pointer:

```c
char *    *const         *x;
```

I split the pointers to make it visually clear. So what do we have? We have a pointer to a read-only pointer to pointer to `char`. What does that mean? It means that we can assign to `x`, but we can't assign to `*x`, since `*x` has type `char **const`, i.e., read-only pointer to pointer to `char`. However, we can change the value pointed to by `**x`, and we can also assign to `***x`. Go through this again, read the last paragraph until you get it, and consider what would change if the declaration was, instead, this one:

```c
const char **const*const x;
```

The correct answer is that it's a read-only pointer to a read-only pointer to pointer to a read-only char. This implies that we can't change `x`, and we can't change `*x`. However, we can change `**x` (which is of type pointer to `char`), but we can't change `***x`, since `***x` is of type `const char`. Pictorially, you can see it like this:

![Const PTRs]({{ site.baseurl }}{{ site.assets }}const_ptr.png)

The colors of each arrow match the colors of each pointer in the level of indirection. In the memory layout picture, `x` denotes `rvalue(x)`, that is, we refer to `x` as if it were in the right hand side of an assignment. See my earlier article about pointers and arrays, [The shocking, unbelievable truth - arrays and pointers in C are not the same thing!]({% post_url 2013-09-07-the-shocking-unbelievable-truth-arrays-and-pointers-are-not-the-same-thing %}), to learn about the lvalue and rvalue terminology.

The picture is self-explanatory and is a good help of how to keep this in your head. The assignment

```
x = ...
```

Is invalid, since that would change the position pointed to by the blue pointer, which is a read-only pointer. The same happens for the assignment

```
*x = ...
```

Because that would be the same as changing the red pointer, again, a read-only pointer. We can, however, change the location pointed to by `**x` (represented by the black pointer), but note that we can't change `***x`, since that's a `const char`.

Don't read the rest of the article until you are sure to understand this, or you'll be wasting your time.

## So, what about `volatile`?

Ok, now that you really know what `const` means, it's time to talk about `volatile`. It is not easy to find someone who knows what exactly `volatile` is, even amongst people teaching operating systems and computer architecture courses. I have asked two of my professors about `volatile`, only to find out that one of them, who teaches operating systems and embedded architecture courses, didn't know about it, and the other, always showing how good his programming experience is, got really confused with this simple concept. I'm guessing this is because of too much theoretical work, PhD's *love* writing super complex papers about some highly theoretical topic and often forget about real world concepts, usually because they have a very rough idea of how the world is like outside of My Academia.

The problem, commonly, is that nobody really knows anything very well. Everyone has a broad knowledge of a wide variety of topics, but no one bothers to study one thing and do it well (the UNIX philosophy), and the net result is that it is really hard to find someone very good at `X`.

In declarations, the rule for `volatile` is exactly the same as that for `const`. The keyword `volatile` is used to denote a variable whose value can change unexpectedly, without the control of the C implementation. This information will usually cause the compiler to refrain from optimizations that mess with this variable. 

The true definition of `volatile` is related to the concept of *sequence points*. The C standard states that references to and modifications of `volatile` objects cannot be optimized across sequence points, but optimizations between sequence points are permitted (although implementations will generally not do so). A sequence point is a point in a program's execution flow at which all previous side-effects of execution are to have taken place and at which no subsequent side effects will have occurred (an example of a statement with a side-effect is `x++`). A sequence point exists:

* At the end of expression statements
* After the control expressions of the `if`, `switch`, `while`, and `do` statements
* After each of the three control expressions in the `for` statement
* After the first operand of the logical `&&`, `||`, `?:` and `,` operators
* After `return` statement expressions
* After initializers
* At the end of a full declarator
* In function calls: immediately after all the arguments are evaluated, before library functions return, after the actions associated with `printf()` and `scanf()` conversion specifiers, and around calls to comparison functions supplied to `bsearch()` and `qsort()`

This means that in an expression such as:

```c
while (x,x) { ... }
```

If `x` is `volatile`, then the compiler will really generate code that fetches `x` twice. If `x` were not a volatile object, the compiler could optimize this to:

```c
while (x) { ... }
```

But note, however, that even if `x` is `volatile`, the expression:

```C
x+x;
```

May be optimized to fetch `x` only once from memory, because there is no sequence point for the `+` operator.

Side note: the C standard also states that modifying an object more than once between sequence points results in undefined behavior. So, this:

```c
a = ++x + x++;
```

is undefined behavior.

For a more practical example of what `volatile` is, imagine you have a pointer `p` to a memory region shared with a hardware resource, and you are waiting for a signal to come from the hardware. To do so, you can assign `0` to `*p`, and then keep polling the memory location, waiting for the hardware to overwrite it with something else. If you don't declare this pointer as a pointer to `volatile`, the compiler will not be aware that this variable can change, and it can go and optimize your code by entirely removing polling and assuming `*p` is always 0, since that's what you assigned it to, and never touch it again. The use of `volatile` is very common in device driver programming, or any other low level programming that deals with hardware resources.

So, can you describe the following declaration?

```c
const int **volatile x;
```

The correct answer, of course, is that `x` is a `volatile` pointer to pointer to `int`.

And then again, we can combine all of this to have something that is both `volatile` and `const`.

## The use of `restrict`

The `restrict` keyword was introduced in C99, its semantics in declarations are exactly the same as those used for `volatile` and `const`.

A `restrict`-qualified pointer is a hint to the compiler that the pointer is, for the moment, the only way to access the object to which it points to. This allows for some crazy optimizations with notable performance improvement (in fact, two restricted pointers, or a restricted and a nonrestricted pointer, can refer to the same object if the object is not modified during the lifetime of the restricted pointers). The implementation is free to ignore the `restrict` keywords. 

The updated manpages for `memcpy()` use the `restrict()` keyword in the function's prototype to implicitly say that the memory areas pointed to by `s1` and `s2` shall not overlap:

```c
#include <string.h>
void *memcpy(void *restrict s1, const void *restrict s2, size_t n);
```

The standard allows type qualifiers (`const`, `volatile`, `restrict`) to be repeated in a pointer or type declaration; this action has no real use other than confusing novice programmers.

Before moving on, I challenge the reader to test the assimilated knowledge by deciphering this declaration:

```c
const volatile int **restrict volatile const *const restrict const const x;
```

## Some important terminology

We are now ready to discuss some important declarations terminology. The table below lists the vocabulary we'll be looking at:

![Declarations Terminology]({{ site.baseurl }}{{ site.assets }}decls_terminology1.png)

Older implementations may not have `long long` and `restrict` (these were added in C99). This shouldn't be much of a problem though, nearly every C implementation nowadays conforms to C99 or C11 (the current standard).

Note that not all combinations in this table are valid. For example, we can't have an `extern auto void`, that wouldn't make sense. The keywords `struct`, `enum` and `union` are not valid C types, what I mean by `struct` is better described as a `struct` specifier. But this is a simplified table, so let's forget about those minor details. I will not talk about the type specifier, it should be pretty much known to the reader. However, it is important to note that only one type specifier is allowed in a declaration, but this type specifier can be composed by more than one token (for example, `unsigned long long` is a type specifier made of 3 tokens).

We have just discussed type qualifiers, so let's turn our attention to storage class specifiers. Storage class specifiers, with the exception of `typedef`, say where a variable is stored. 

`extern` says that the variable was defined elsewhere, which means that there's no need to allocate space for it. This keyword is normally used to access global variables declared in a different source file.

`static` has 3 different meanings depending on whether we're talking about a local or a global variable. In the former case, it means that the variable will hold its value between different calls of the same function - it works just like if it was a global variable, but it is only visible inside the function to which it belongs to. In the latter case, it means that the variable is not globally visible to other C files (it is not exported to the linker). This is also valid for functions, `static` functions will not be exported to the linker and consequently cannot be called from outside the source file where they are declared. C99 assigns a third meaning to `static`: it can be used in an array declaration within array brackets in a function parameters declaration to assert that the actual argument will be non-null and will have the declared size and type upon entry to the function. C99 also defines a whole load of new valid declarations for arrays because of variable length arrays. I will not cover them in this article.

A `register` variable is just like any other variable. The `register` keyword conveys a hint at the compiler that this variable will be intensively used and that it should be kept in a CPU register. The compiler is free to completely ignore such advice, and it often does. The use of this keyword is generally not considered good practice, because compilers are very smart these days, and are way better than the average programmer when it comes to improving performance. You can't take the address (i.e., apply the referencing operator `&`) of a `register` variable.

`auto` is a useless, redundant keyword that is used to say that the space for a variable is to be automatically allocated on block entry. Every local variable, as well as global variables that are not `extern` are implicitly automatically allocated. There is no real use for this keyword.

Last, but not least, `typedef` is used to create an alias for a type. Even though it is considered a storage class, it totally changes the meaning of the declaration: when you use `typedef`, you're not declaring a variable anymore, you're creating an alias for a type. The best way to learn how to use `typedef` is to always remember that it is a storage class specifier. If you wanted to create an alias `xpto` for the type `int *`, begin by declaring an `int *` named `xpto`:

```c
int *xpto;
```

And now prepend it with a `typedef`:

```c
typedef int *xpto;
```

From now on, you can refer to `int *` as `xpto`. This declares a pointer to `int` named `x`:

```c
xpto x;
```

Note that when you use an alias, you can't "add" any further type specifiers (but you are free to include other storage class specifiers or type qualifiers). This is not valid:

```c
unsigned xpto x; /* NO.... WON'T COMPILE */
```

`typedef` can be your friend when you're dealing with complicated declarations that mess with function pointers. With multiple indirection levels, `typedef` serves as an abstraction layer. One golden rule to keep in mind is that you should always include `typedef` in the beginning of a declaration. Never write code like this, unless, of course, you deliberately want to obfuscate your program:

```c
/* Not to be read by humans */
unsigned const long typedef int volatile *i, *j, k, **z;
```

To the curious reader, this declares three aliases:
`i`, an alias for the type `unsigned const long int volatile *`
`j`, the same as `i`. So, using `j` or `i` is equivalent
`k`, an alias for the type `unsigned const long int volatile`
`z`, an alias for the type `unsigned const long int volatile **`

For LALR(1) aficionados: yes, the `typedef` keyword introduces an ambiguity in C's grammar, because you can't distinguish type specifiers from identifiers during lexical analysis (the meaning of an identifier depends on the context). Because of this, no LALR(1) C compiler can parse a source file in one step; the syntactical analyzer must feed back the source file to the lexical parser.

Good practice recommends organizing C declarations such that storage class specifiers come first, followed by type qualifiers, and finally type specifiers. Using this rule, the previous declaration becomes:

```c
/* A little bit better */
typedef const volatile unsigned long int *i, *j, k, **z;
```

We recommend breaking down multiple declarations into multiple lines; in Computer Science, it is generally a good idea to keep program lines straight to the point and avoid statements with numerous simultaneous outcomes:

```c
/* Much better */
typedef const volatile unsigned long int *i;
typedef const volatile unsigned long int *j
typedef const volatile unsigned long int k
typedef const volatile unsigned long int **z;
```

## The precedence rule

Over the years, students, teachers, and software developers came up with techniques and algorithms to better understand C declarations syntax. No matter what method you want to use, the core principle to understand a C declaration is to always remember about the precedence rule. As stated by numerous programmers, the precedence rule is high on brevity, but low on intuition. Language lawyers *love* this rule, and anyone claiming to be a C expert will typically use this to learn how to read declarations (wondering what are language lawyers? They are those people who will find the 3 or 4 paragraphs scattered all over a 200-page standard that together imply the answer to some question you might have about a programming language, and they are usually pretty darn good programmers). Here's the precious rule, taken from *Expert C Programming - Deep C Secrets*:

![Declarations Terminology]({{ site.baseurl }}{{ site.assets }}p_rule.png)

This is all you will ever need to know in order to translate an abritrarily complex C declaration into an arbitrarily complex English sentence. Note that B.1 recursively calls the precedence rule to parse whatever is inside a pair of grouping parentheses in the first place. This is what makes this declaration:

```c
int (*x)[12];
```

different from:

```c
int *x[12];
```

Because with the former, B.1 indicates that `(*x)` has higher precedence than array indexing, and with the latter, we fall into case B.2 (array indexing). 

The precedence rule recursively applies to each of a function's arguments, enabling you to manually parse and understand some harder function types, as we will see in a few moments. It is also worth mentioning that the nature of C declarations is such that you always know where the identifier is supposed to be, even if it's not there, which happens with, for example, casting, or when a function prototype is declared (these are called abstract declarators).

Let's run the precedence rule with both of the above example declarations, to see the mechanics of this amazing algorithm working. Let's start with the first one. The rule states that declarations are to be read by starting with the identifier's name, and that's what we will do. In this case, the identifier is `x`, so we say out loud:

> x is...

Now, we can see that we have an expression enclosed in parentheses; it is visually easy to obtain this information. However, with complicated declarations, this may be hard to see. A good alternative approach is to start by checking if there are function call parentheses or array indexing to the right. In this case, there aren't, so we can go to B.3, and we see that this rule matches up: there's an asterisk to the left of `x`, so:

> x is a pointer to...

Now we continue reading our declaration right to left, and we realize that we finished parsing a parenthesized expression. And now the whole method can be repeated: is there array indexing or function call? Yes, there is: we have `[12]`, so:

> x is a pointer to an array of 12...

After reading the array dimension, the declaration is over. We just have to read the type specifier now, which always comes in the beginning, and it's `int` in this case:

> x is a pointer to an array of 12 int

What happens with the other declaration? Try to translate it as an exercise:

```c
int *x[12];
```

Remember, array indexing has higher precedence, so, after reading the identifier, we have to look at array indexing:

> x is an array of 12

Ok, we have read the array dimension, and we can now go back to the `*`:

> x is an array of 12 pointer to

And again, that's the end of it, so now we read the type specifier:

> x is an array of 12 pointer to int

It is a very nice exercise to practice with this rule. Go over this set of examples again, and carefully examine what's happening, because we'll be writing an implementation for this. If you still have trouble dealing with the precedence rule, think of it like this:

* Function call and array indexing are prioritary. Are there any of these immediately to the right of what I just read? If yes, append "array of" or "function returning" to my current translation
* Are there any `*` to the left? If yes, add "pointer to" for each `*`k until I find something that is not `*` (for example, an opening parentheses).

As I said before, the tricky part is that declarations are not entirely read left-to-right, nor right-to-left: your eyes have to alternate between left-to-right reading for point number 1, and right-to-left reading to apply point number 2: for `int **x[12][14][10]`, first you must read every array indexing, and only after that you can turn to the pointer declarators. Pointer declarators must be read right-to-left because declaration follows use.

In other words, declarations in C are read *boustrophedonically*. Hahahaha!! You didn't think there was an English word to describe how to read C declarations, did you? A boustrophedon, as described by wikipedia, is *... a kind of bi-directional text, mostly seen in ancient manuscripts and other inscriptions [...] Rather than going left-to-right as in modern English, or right-to-left as in Arabic and Hebrew, alternate lines in boustrophedon must be read in opposite directions*. I think it is now clear where K&R got their inspiration from.

Before reading the rest of this interesting article, make sure you can work your way towards a more complicated declaration, one that even advanced C students often struggle with:

```c
void (*signal(int, void (*)(int)))(int)
```

This is the POSIX function to set a signal handler. For example, you can change the default behavior or your program when a segfault occurs to call a function in your code rather than having the operating system abruptly stopping the process. This can be used to display a nice friendly user message with a "send bug report" button before gracefully stopping the process.

Don't be afraid of this declaration: if you understood the precedence rule, you should be able to decipher this easily. Please try to do so before reading on. Hint: the identifier is `signal`.

So, did you make it? Let's see what's happening here...

First, notice that there are function calling parentheses after the identifier. This has higher precedence than the pointer dereferencing operator; this means that `signal` is a function:

> signal is a function

We see that `signal()` is receiving some arguments. Function arguments are separated by commas, and we can recursively apply the precedence rule on each. The first is straightforward; the second is a little more trickier because the identifier is missing. By applying the rule, it can be seen that the second identifier is a pointer to a function receiving `int` and returning `void` (that is, it doesn't return anything). I'm not covering the recursive call to avoid further confusion. This is what we have so far:

> signal is a function receiving: [an int and a pointer to a function receiving: [an int] and returning void] and returning...

Now that we have parsed the function call to the right of `signal`, we can process pointer dereferencing, since there's a `*` to our left. Thus, we get:

> signal is a function receiving: [an int and a pointer to a function receiving: [an int] and returning void] and returning pointer to...

Now, we are at a point where we finished processing a parenthesized prioritary expression. This is what we're left with to process:

```c
void (*....)(int)
```

We have processed `(*....)`. We now see a function call parentheses to something that receives an `int`. Also, this function returns `void`. So: 

> signal is a function receiving: [an int and a pointer to a function receiving: [an int] and returning void] and returning pointer to function receiving: [int] and returning void.

As I said before, `typedef` can be your friend. In fact, the manpage for `signal()` is a bit more friendly and provides an alias for the type `void (*)(int)` to better explain the idea behind `signal()`:

```c
#include <signal.h>
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
```

It is definitely easier to read, especially for the untrained eye. The alias `sighandler_t` is created to denote the type "pointer to function receiving `int` and returning nothing". The declaration

```c
sighandler_t signal(int signum, sighandler_t handler);
```

Is much more cleaner to read, and conveys the correct idea about `signal()`: it receives the signal identifier, and a pointer to the function which shall be called when this signal arises, and returns the previous function associated to this signal. Note: this function is part of ANSI C, but it is now considered obsolete. As of this writing, you should use `sigaction()` instead for full portability. I just chose `signal()` for this example because `sigaction()` is not an interesting declaration to examine.

Try to decipher these declarations, taken from K&R:

```c
char **argv;
int (*daytab)[13];
int *daytab[13];
void *comp();
void (*comp)();
char (*(*x())[])();
char (*(*x[3])())[5];
```

See K&R page 122 for solutions.

## A C declarations parser

A traditional and extremely useful exercise for those learning about C declarations is to build a C declarations parser that takes a declaration and translates it into English using the precedence rule. This program is often known as *cdecl* (as in "C declaration"), or *dcl*.

Nothing can replace good old K&R section 5.12, where a simplified grammar to describe C declarations is presented, as well as a primitive recursive descent version of *cdecl*. I really encourage you to go through that section. In fact, I really encourage you to read K&R: no one can claim to be a C expert if they didn't read K&R AND Expert C Programming - Deep C Secrets. These are the 2 C bibles written by top programmers. Learn with them. Of course, this is not to say that other C books are crap (well, some actually are). Another good known resource is C - A reference manual, by Harbison & Steele. 

The *dcl* program presented in K&R section 5.12 does not support functions with arguments, and it always requires the identifier to be there (i.e., you can't just enter a type, like `int *[12]`). Also, it doesn't support type qualifiers (`const` and `volatile`). In this section, I'll be presenting 2 alternative implementations for the *dcl* program: one of them is based in the approach taken by Peter Van Der Linden in Expert C Programming - Deep C Secrets, and the other is based in the approach taken by K&R. Neither of them considers error handling. One of the subsequent exercises in K&R is to add some basic error detection and recovering in *cdecl*, but we're not going to be doing any of that in this article. You can assume the input is always well formed.

Here's how we're going to do this: I'm going to give a general description of the program design, and present some useful and important guidelines to implement the parser. Then, *you* will try to implement it before looking at my solution. Implementing a C declarations parser is an excellent way to finally understand how C declarations work. It is also educational and very satisfying. In his book, Peter points out that:

> In any event, when you program this parser, you are implementing one of the major subsystems in a compiler - that's a substantial programming achievement, and one that will really help you to gain a deep understanding of this area.

Obviously, it is desired that you have some background and familiarity with compilers development. At the very least, you should be comfortable with recursion, because it will be widely used: the second approach is based off a recursive descent parser.

This is the grammar we'll be working with:

![DCL Grammar]({{ site.baseurl }}{{ site.assets }}dcl_grammar.png)

This is a custom grammar based on the one provided on K&R. I added basic support for type qualifiers and functions with arguments. Later, we will expand it to rudimentarily handle abstract declarators. It is assumed that you are familiar with grammars (you remember that from your compilers course, right?). Take some time to go through this grammar and explore it. Note that it is not perfect: it doesn't permit the use of storage class specifiers, it allows invalid declarations like `restrict int x;` (`restrict` is only for pointer declarations), there is no support for `enum`, `union` and structure creation and tag declaration (we will just denote structures, unions and enums by the keywords `struct`, `union`, or `enum`), it doesn't allow abstract declarators, and the size expression syntax for arrays was not specified - we will assume it's a number in base 10, but in reality this would be much more complex, especially if we consider C99 variable-length arrays (variable-length arrays declarations have different rules when they are declared as a function parameter from when they are declared as a local variable). But this is fine for our educational purposes.

A token is considered to be either a string starting with an alphabetic character followed by any number of alphanumeric characters, or a single character (e.g. `int (*x)[12]` is decomposed into the tokens `int`, `(`, `*`, `x`, `)`, `[`, `1`, `2`, and `]`).

First of all, let's use this grammar to parse declarations manually. Here are some useful annotations that make this grammar a handy C declarations guide to carry around. Just print it and stick it in your pocket!

![DCL Grammar Extended]({{ site.baseurl }}{{ site.assets }}dcl_grammar_and_code.png)

Declarations have fancy terms to denote each part of it; you can read about declarators and direct declarators in K&R or C: A Reference Manual. The annotated grammar shown above includes a set of instructions associated with each production. These pseudo-instructions are typically placed in the end of a production, but they can appear somewhere between the tokens. This happens with the rule `direct-dcl -> direct-dcl(args)`: after processing the opening parenthesis, and before expanding `args`, we must execute `say("function receiving ");`, then we process arguments and the closing parentheses, and then we execute `say(" and returning ");`. All the other rules have instructions at the end; they shall be executed only after every token in that rule has been processed.

The subscripted index `2` in the tokens for `type_qualifier` indicate that the token refers to the token on the right hand side of the expansion.

`ptr` rules must use a queue to keep track of the pointer declarations seen. We can't say anything in the moment we're processing pointer tokens, we need to save this for when every token in `dcl -> ptr direct-dcl` is finished processing. This is because of the precedence rule - note that by the time we have finished processing a `direct-dcl`, we have looked at every prioritary operations (array indexing, function call, and `dcl`s in parentheses to change evaluation order). Thus, `dcl` and `direct-dcl` rules are the core of our algorithm, and they fully reflect the precedence rule.

With this annotated grammar, you can parse and translate an arbitrarily complex C declaration into an English statement of the same complexity.

Consider what happens with the following declaration:

```c
char **argv;
```

The following image shows the tokens processing sequence:

![Ptr-to-ptr tokens]({{ site.baseurl }}{{ site.assets }}dcl1.png)

The token processing is naturally recursive, as you can see. As usual, recursion is simulated by indenting the context of a function call in the same level - `type` and `dcl` expansion both happen inside `decl`, and `ptr` and `direct-dcl` happen inside `dcl`.

As you can see, the sequence of instructions executed was:

```
say("argv: ");
say("pointer to ");
say("pointer to ");
say("char");
```

Which translates to the sentence:

> argv: pointer to pointer to char

Play around with the grammar for a while, and make sure you understand it. Practice is the key to success.

Did you get it? Are you mastering declarations now? So here's a harder one: use the annotated grammar to decipher this declaration:

```c
char (*(*x[3])(float (*y)(int a)))[5];
```

Don't look down! Do it for yourself first.

This is highly recursive, but don't get confused. The following image shows the token processing sequence:

![Complex token processing]({{ site.baseurl }}{{ site.assets }}dcl4.png)

The sequence of instructions executed was:

![Complex token instructions]({{ site.baseurl }}{{ site.assets }}dcl5.png)

Which yields:

> x: array of 3 pointer to function receiving y: pointer to function receiving a: int and returning float and returning pointer to array of 5 char

Now, isn't *THAT* beautiful?

Part of understanding declarations is to picture them in your head. The declaration syntax looks horrible, but you can see this declaration like this:

![Complex declaration decomposition example]({{ site.baseurl }}{{ site.assets }}dcl6.png)

In the next section, we will be implementing a parser for this grammar. It is important that you learn how the grammar is built and how it works. The annotations are very close to the behavior of the implementation; `say()` is just `printf()`.

If you look carefully at the grammar, you can see that the first call to `say()` is always in the expansion `direct-dcl -> identifier`. Therefore, the implementation will read tokens until it finds an identifier, keeping track of the tokens behind in a stack, so that we can remember them when it is time to read them. The examples posted above should be enough for you to get a feeling of how the recursion is going to work. There are two approaches to implement the stack of tokens: either we code a stack library for our program to use, or we use the stack frame mechanism, i.e., everything is recursive, and we use the the call stack. Let's explore both options.

Both implementations make use of a lookahead token. What does that exactly mean? Well, sometimes, computer programs can't know that they've read too much input until they actually read too much. To know when to stop, we have to keep reading until we read something that is not intended. Thus, every function in our parser will read one more token than it was supposed to. Consequently, every function, upon entry, assumes that the first token was already consumed (with the exception of `main()`).

## Using a stack of tokens

This is taken from Expert C Programming. The main data structure is a stack, on which we store tokens that we have read, while we are reading forward to the identifier. Then we can look at the next token to the right by reading it, and the next token to the left by popping it off the stack. The data structure looks like:

```c
struct token {
    unsigned char type;
    char string[MAXTOKENLEN];
};

/* holds the token just read */
struct token last;
```

The functions you have to implement are:

* `gettoken()`
* `read_to_ID()` - repeatedly stores tokens in a stack until an identifier is found. When that happens, it prints the identifier name just like we did in the annotated grammar
* `parse_declarator()` - implements the `direct-dcl` expansions. Must check for trailing array index or function call characters. In case it's a function call, it must be smart enough to differentiate between a function with an empty arguments list and a function with arguments. After processing these prioritary constructs, it uses the stack to pop previous arguments so that it can print pointer declarations. Also, it must use the stack to recognize recursive `( dcl )` expansions
* `main()` - initializes a stack to store tokens. Calls `read_to_ID()` and then `parse_declarator()`

That's most of what you'll be needing. Of course, I encourage you to define a few more auxiliary functions.

Some words of advice: don't try to implement everything at once. Use the annotated grammar to guide yourself. Start by coding `gettoken()`, and test it. See if it's working properly. Hand it a valid declaration and check if the tokens returned match what you expected. Then you can implement `read_to_ID()` - this is not very hard. After that, you're ready to code `parse_declarator()`. Functions that deal with arrays and pointers are small and easy to implement. Finally, you can write `main()`, and then see the beautiful results of running this code.

So, did you make it? Let's look at my solution. It's available on the blog's git repository.

```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>
#include <stdlib.h>
#define strcmp(a,R,b) (strcmp(a,b) R 0)
#define MAXTOKENLEN 50
#define STACKSIZE 100
#define TYPE 0
#define QUALIFIER 1
#define IDENTIFIER 2
#define push(s,t,a) s[(t)++] = a
#define pop(s,t) s[--(t)]
#define stackempty(t) ((t) == 0)
#define stackfull(t) ((t) == STACKSIZE)

struct token {
	unsigned char type;
	char string[MAXTOKENLEN];
};

struct token last;

void read_to_ID(struct token *stack, int *top);
void parse_declarator(struct token *stack, int *top);
void do_parse(void);

int main(void) {
	while (1) {
		printf("> ");
		do_parse();
		printf("\n");
	}
	return 0;
}

void do_parse(void) {
	struct token stack[STACKSIZE];
	int top = 0;
	read_to_ID(stack, &top);
	parse_declarator(stack, &top);
}

int classify_string(void) {
	char *str = last.string;
	if (strcmp(str, ==, "const")) { /* We call const "read-only" to clarify */
		strcpy(last.string, "read-only");
		return QUALIFIER;
	}
	else if	(strcmp(str, ==, "volatile"))
		return QUALIFIER;
	if (strcmp(str, ==, "int") ||
		strcmp(str, ==, "short") ||
		strcmp(str, ==, "char") ||
		strcmp(str, ==, "float") ||
		strcmp(str, ==, "double") ||
		strcmp(str, ==, "void") ||
		strcmp(str, ==, "signed") ||
		strcmp(str, ==, "unsigned") ||
		strcmp(str, ==, "long") ||
		strcmp(str, ==, "struct") ||
		strcmp(str, ==, "union") ||
		strcmp(str, ==, "enum"))
		return TYPE;
	return IDENTIFIER;
}

void gettoken(void) {
	int c;
	int i = 0;
	/* Ignore white spaces */
	while (isspace(c = getchar()) && c != EOF && c != '\n');
	if (c == EOF)
		exit(0);
	last.string[i++] = c;
	if (isalpha(c)) {
		for (; isalnum(c = getchar()) && i < MAXTOKENLEN-1; last.string[i++] = c);
		ungetc(c, stdin); /* pushes back the excess character read */
	}
	last.string[i] = '\0';
	if (isalpha((unsigned char) last.string[0]))
		last.type = classify_string();
	else {
		last.type = last.string[0];
		if (last.type == '*') {
			/* Make string contain "pointer to", otherwise, qualified pointers would be printed as '*' */
			strcpy(last.string, "pointer to");
		}
	}
}

void read_to_ID(struct token *stack, int *top) {
	for (gettoken(); last.type != IDENTIFIER && !stackfull(*top); push(stack, *top, last), gettoken());
	printf("%s: ", last.string);
	gettoken();
}

void read_array_size(void) {
	char buf[MAXTOKENLEN];
	int c, i = 0;
	while ((c = getchar()) != ']' && i < MAXTOKENLEN-1)
		buf[i++] = c;
	buf[i] = '\0';
	gettoken();
	printf("array of %s ", buf);
}

void read_function_args(void) {
	struct token stack[STACKSIZE];
	int top, c;
	if ((c = getchar()) == ')') {
		printf("function returning ");
		return;
	}
	ungetc(c, stdin);
	printf("function receiving ");
	do {
		top = 0;
		read_to_ID(stack, &top);
		parse_declarator(stack, &top);
		if (last.type == ',')
			printf("and ");
	} while (last.type == ',');
	if (last.type != ')')
		fprintf(stderr, "Warning: missing close parentheses in function\n");
	gettoken();
	printf("and returning ");
}

void check_ptr(struct token *stack, int *top) {
	struct token t;
	if (stackempty(*top))
		return;
	for (t = pop(stack, *top); t.type == '*'; t = pop(stack, *top))
		printf("pointer to ");
	push(stack, *top, t);
}	

void parse_declarator(struct token *stack, int *top) {
	struct token t;
	if (last.type == '[')
		while (last.type == '[')
			read_array_size();
	else if (last.type == '(')
		read_function_args();
	check_ptr(stack, top);
	while (!stackempty(*top)) {
		t = pop(stack, *top);
		if (t.type == '(') {
			if (last.type != ')')
				fprintf(stderr, "Warning: missing close parentheses\n");
			gettoken();
			parse_declarator(stack, top); /* Recursively parse this ( dcl ) */
		} else
			printf("%s ", t.string);
	}
}
```

It kind of creates a shell waiting for declarations to be entered (without a trailing semicolon). Abstract declarations are not supported. Structure, enums and unions are identified by the single keyword `struct`, `enum` and `union`. There is no error recovery, it you feed it invalid declarations, it gets really confused.

Play around with the code. There is not much to say about it if you understood the grammar.

## Using a lot of recursion

The problem is naturally recursive; we don't need additional data structures nor do we need a custom stack. The following implementation absolutely reflects the grammar definition. It was extended to allow abstract declarators, and function arguments are now reported in a newline with proper spacing that increases as the nesting level increases, making complex declarations easy to break down into logical blocks and easy to read for the human eye.

It is educational in many ways to analyze the code for this approach, and it is extremely interesting to note that this can be implemented without additional structures: the function call stack is all we need.

To support abstract declarations, we add an empty production in `direct-dcl`. This implies that when processing a `direct-dcl`, in the function `direct_dcl()`, we must use our lookahead symbol when we read an opening parentheses to distinguish the case when it is a function call from the case `( dcl )`, because we can't rely anymore on the fact that an opening parenthesis before reading an identifier must be a `( dcl )`.

And now, as Peter Van Der Linden says, *the piece of code that understandeth all parsing*:

```c

/* This code has been developed for educational purposes only
 * It makes dangerous assumptions to be used in a production environment (no buffer limit tests are performed)
 * This must be compiled with gcc, or any other compiler supporting __VA_ARGS__ extension in macros definition
 * HOW TO USE:
 * Compile; run the program
 * Write a valid C declaration (there is no error checking, if input is mal-formed, the parser gets confused)
 * Any declaration is supported. You can omit identifiers names if you want. The parser supports structs, enums and unions
 * in a very primitive way, by the sole use of the keyword struct, enum, or union. So, instead of struct foo x, input should be
 * struct x. Do NOT include ; in the end
 * Please don't specify any storage-class specifiers (extern, static, register, typedef, auto)
 * Enjoy! And study the code!
 */

#include <stdio.h>
#include <string.h>
#include <ctype.h>
#define strcmp(a,R,b) (strcmp(a,b) R 0)
#define type_or_qualifier(token) (strcmp(token, ==, "volatile") || strcmp(token, ==, "int") || strcmp(token, ==, "short") || strcmp(token, ==, "char") || strcmp(token, ==, "float") || strcmp(token, ==, "double") || \
	strcmp(token, ==, "void") || strcmp(token, ==, "signed") || strcmp(token, ==, "unsigned") || strcmp(token, ==, "long") || strcmp(token, ==, "struct") || strcmp(token, ==, "union") || strcmp(token, ==, "enum") || \
	strcmp(token, ==, "const"))
#define print_spaces(d) do { \
	int x = d; \
		while (x--) { \
			putchar(' '); \
			putchar(' '); \
		} \
	} while (0);
#define pprintf(str, ...) do { \
		if (init) { \
			print_spaces(depth); \
			init = 0; \
		} \
		printf(str, ##__VA_ARGS__); \
	} while (0);
#define MAXTOKEN 512

int depth;
int init;

void decl(void);

int main(void) {
	while (1) {
		depth = 0;
		init = 1;
		printf("> ");
		decl();
		printf("\n");
	}
	return 0;
}

void type(char *buf);
void dcl(void);
void gettoken(void);

char token[MAXTOKEN];

void decl(void) {
	char decl_type[MAXTOKEN];
	decl_type[0] = '\0';
	gettoken();
	type(decl_type);
	dcl();
	pprintf("%s\n", decl_type);
}

void gettoken(void) {
	int c, i;
	while (isspace(c = getchar()) && c != '\n');
	i = 0;
	token[i++] = c;
	if (isalpha(c)) {
		for (; isalnum(c = getchar()); token[i++] = c);
		ungetc(c, stdin);
	}
	token[i] = '\0';		
}

void type(char *buf) {
	for (; ; gettoken()) {
		if (strcmp(token, ==, "const")) { /* We call const "read-only" to clarify */
			strcat(buf, "read-only ");
		}
		else if	(type_or_qualifier(token)) {
			strcat(buf, token);
			strcat(buf, " ");
		}
		else
			break;
	}
}

void direct_dcl(void);
char *ptr(char *pointers);
void dcl(void) {
	char pointers[MAXTOKEN];
	ptr(pointers);
	direct_dcl();
	pprintf("%s", pointers);
}

char *ptr(char *pointers) {
	char *p;
	if (token[0] == '*') {
		gettoken();
		p = ptr(pointers);
		return p+sprintf(p, "pointer to ");
	}
	else if (strcmp(token, ==, "const")) {
		gettoken();
		p = ptr(pointers);
		return p+sprintf(p, "read-only ");
	}
	else if (strcmp(token, ==, "volatile")) {
		gettoken();
		p = ptr(pointers);
		return p+sprintf(p, "volatile ");
	}
	else {
		*pointers = '\0';
		return pointers;
	}
}

void read_array_size(void);
void read_function_args(void);
void direct_dcl(void) {
	if (token[0] == '(') {
		gettoken();
		if (token[0] == ')' || type_or_qualifier(token))
			read_function_args();
		else
			dcl();
		if (token[0] != ')')
			fprintf(stderr, "Warning: missing close parentheses\n");
		else
			gettoken();
	}
	if (isalpha((unsigned char) token[0])) {
		pprintf("%s: ", token);
		gettoken();
	}
	if (token[0] == '[') {
		gettoken();
		read_array_size();
	}
	if (token[0] == '(') {
		gettoken();
		read_function_args();
		if (token[0] != ')')
			fprintf(stderr, "Warning: missing close parentheses in function\n");
		else
			gettoken();
	}
}


void read_array_size(void) {
	pprintf("array of ");
	while (isdigit((unsigned char) token[0])) {
		putchar((unsigned char) token[0]);
		gettoken();
	}
	putchar(' ');
	if (token[0] != ']')
		fprintf(stderr, "Warning: missing close bracket\n");
	else
		gettoken();
	if (token[0] == '[') {
		gettoken();
		read_array_size();
	}
}

void read_function_args(void) {
	char decl_type[MAXTOKEN];
	int args_left = 1;
	if (token[0] == ')') {
		pprintf("function returning ");
		return;
	}
	pprintf("function receiving:\n");
	depth++;
	while (args_left) {
		init = 1;
		decl_type[0] = '\0';
		type(decl_type);
		dcl();
		pprintf("%s\n", decl_type);
		if (token[0] == ',')
			gettoken();
		else
			args_left = 0;
	}
	depth--;
	print_spaces(depth);
	pprintf("and returning ");
}
```

This shouldn't be very difficult to understand for anyone mastering the grammar. The hardest part is to get the tokens management right to follow that rule of reading one more token than is needed.

Finally, I have a small Christmas gift:

## The obfuscated version

This code won the 1988 IOCC. It was submitted by Gopi Reddy, and it fully implements a rudimentary C declarations translator. Type qualifiers and function arguments are not supported.

```c
#include<stdio.h>
#include<ctype.h>
#define w printf
#define p while
#define t(s) (W=T(s))
char*X,*B,*L,I[99];M,W,V;D(){W==9?(w("`%.*s` is ",V,X),t(0)):W==40?(t(0),D(),t(41)):W==42?(t(0),D(),w("ptr to ")):0;p(W==40?(t(0),w("func returning "),t(41)):W==91?(t(0)==32?(w("array[0..%d] of ",atoi(X)-1),t(0)):w("array of "),t(93)):0);}main(){p(w("input: "),B=gets(I))if(t(0)==9)L=X,M=V,t(0),D(),w("%.*s.\n\n",M,L);}T(s){if(!s||s==W){p(*B==9|*B==32)B++;X=B;V=0;if(W=isalpha(*B)?9:isdigit(*B)?32:*B++)if(W<33)p(isalnum(*B))B++,V++;}return W;}
```

Yes, I decoded that, and I can tell you how it works. I am that crazy. First of all, this code assumes ASCII representation. 9 and 32 are the ascii codes for tab and space, you will see these numbers hardcoded in this program.

There are 7 global variables, 3 of which are pointers to a location in a character array, one is a character array with space for 99 characters, and the other 3 are integers. Here's what these variables are used for:

* `I` - a characters buffer where the whole declaration is stored. The code calls `gets()` in the beginning to read the whole declaration and store it in `I`. (Note: never use `gets()`; this code has buffer overflow all over the place. Use the safer `fgets()` variant instead.)
* `X` - When an alphanumeric string token is read, `X` will point to the beginning of the token in `I`
* `B` - pointer to the next character, in `I`, to process
* `L` - pointer to the first character of the type specifier in `I`
* `M` - size of the type specifier (number of characters)
* `V` - used to store sizes of important strings
* `W` - holds the token type just read. 9 indicates it was a string; 32 means it's a string starting with a number (for array size declarations). In either case, `V` holds the string's length. All other codes are the ascii code of the character read. 9 and 32 can be used without conflict because these represent tab and space, which are ignored by the parser

There are two routines, `T()`, which lexes the next token and says whether it is an identifier, number, etc., and `D()`, which does the parsing. `D()` is recursive, and it assumes that the next token was already read by `T()` (the same as we did for our implementation), thus, it can only be called after `main()` processes the type specifier. `main()` reads a line from input, reads and stores the type specifier, grabs one more token, and then calls `D()`, and the party begins.

Note that this parser allows nonsense declarations like `int f()()()()` (so does ours).

## The end

I hope you enjoyed this article. We've been through some very important concepts. Read this again, do the exercises, and learn it all properly. Very few programmers master C declarations. People tend to be scared at anything more complex than `int *x`, and will often get away with the excuse that complicated declarations are not used in real world. They forget about the Linux Kernel, and other core code that is the base of the operating system running on their computer. If you don't understand pointers and C very well, you will not understand a single line of the Linux Kernel.

In the Code Snippets section, you can access my code repository where I posted the code shown in this article, as well as other interesting stuff. For the `dcl` parser, I provide a sample input file, and the corresponding expected output.

Oh, and of course, **have fun**!
