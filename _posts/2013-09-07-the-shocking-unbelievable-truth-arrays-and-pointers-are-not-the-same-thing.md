---
layout: post
title: The shocking, unbelievable truth - arrays and pointers in C are not the same thing!
---

No, an array is not a pointer. And a pointer is not an array.

-----

> The name of the song is called: "Haddocks' Eyes."
> "Oh, that's the name of the song, is it?" Alice said trying to feel interested.
>
> "No, you don't understand," the Knight said, looking a little vexed.
> "That's what the name is **called**. The name really **is** «The Aged Aged Man.»"
> "Then I ought to have said «That's what the song is called»?", Alice corrected herself.
>
> "No, you oughtn't: that's quite another thing! The **song** is called «Ways and Means»: but that's only what it's **called**, you know!"
>
> "Well, what **is** the song, then?", said Alice, who was by this time completely bewildered.
>
> "I was coming to that," the Knight said. "The song really is «A-sitting On A Gate»: and the tune's my own invention."

in Lewis Carroll, *Through the Looking Glass*, taken from *Expert C Programming - Deep C Secrets*, Peter Van Der Linden, page 63

Computer science people should be able to deal with this better than the average person, but it is very easy for the human mind to mix things up, and dive into a self contained loop of confusion around its own, wrong assumptions. This particular example illustrates a rudimentary use of type models in programming languages. You see, you basically have a song `x` of type `y`. There is also a `typedef y z`, so `y` is called `z`. The type (name) of the song is `y` ("The Aged Aged Man"), but the type is called `z` ("Haddocks' Eyes"). The song is called `x` ("Ways and Means"), and the song is "A-sitting On A Gate" - that's the value of `x`. Understanding this is not a matter of magic, it's a matter of brain washing yourself and format your head to think like a programmer. Laying this down in a table is easier to understand. It is fully applicable to a `typedef` example in C. 

Did you mix things up reading this *Alice In Wonderland* quote? Try showing it to other people next to you.

A very common phenomenon associated with mixing things up is the process of mass propagating the confusion that your brilliant head created to everyone else around you. And, surprisingly enough, hoaxes and rumors are really easy to spread around. That, together with the fact that, sometimes, those rumors are not totally wrong, is half way to total chaos, and suddenly, for example, everyone thinks that arrays and pointers in C are the same thing. And now everyone is brain damaged on this matter, and it will be harder to explain why that isn't true. Oops, I mean, it's not *always* true; luckily, most programmers don't realize this and everything works just fine, and the world is happy. Or is it?

Now, *you* try to go to a room full of computer science students in an algorithms and data structures class and tell them that. They will typically be learning C, and it so happens that their teacher told them arrays turn into pointers. Now you're telling them that that's not always the case. And not only they're learning C, this whole new algorithm stuff, recursion, pointers, arrays, *everything* is new, and if you try to explain why a pointer and an array are not the same thing, their head will just burn out. Kaput. Gone. Segmentation fault. Forget it. They will fall asleep, they will surf the web using their shiny little smartphone or tablet (and hopefully come and read another great post in codinghighway.com), they will stare at the clock waiting for the time to go home. The point is: *no one will listen to you*.

The problem, of course, is that they think pointers and arrays are the same thing, and it kind of seems to work for 99% of the programs they're coding, and so no one really cares about teaching them proper C. And now you have computer science graduates who "know" C. And then they start working in a bigger program, and suddenly their code stops working, or maybe it doesn't even compile, because they fell off in one of those nasty cases where pointers and arrays are different things. And then they dig for countless hours trying to figure What The Hell Is Wrong With My Code, with no success. Damn those kids, if only they had paid attention in class, they would save *days* of their life.

I wondered how this problem with arrays and pointers being the same in C showed up in the first place. Have you ever wondered? 

I think I know the answer. We, human beings, have a tendency to generalize everything. Our head loves routine, "it worked before, so it has to work now". Let me give you some more concrete examples. Since the day you were born, the sun rises every morning. What makes you believe that tomorrow the sun is going to rise? Because it always did? Why does that matter? See? You're assuming the sun is going to rise tomorrow morning because it always worked like that. There it is, the "it worked before, so it has to work now" spirit. The problem is that it works under certain circumstances, and it might be the case that circumstances change without you realizing it, and suddenly you will have an unpleasant surprise. 

In my freshman's year, I had a Calculus Professor who addressed this issue in the first classes. The first classes were about logic and proofs, and she was trying to convince us not to assume something is true just because we've seen lots of examples where it happened to be true. In Maths, if you want to prove something, you can't prove it by saying "All of these 1,000,000 numbers followed the rule, so it holds for every number". There's no such thing in Maths, that's just not valid. Her example was quite controversial: she said that just because lots of people have died, it doesn't mean *everyone* will die some day. That's the real life version of "it was true for 1,000,000 numbers, so it's always true" in Maths.

Ok, I know this is getting weird. I just told you that there's a chance someone is *immortal*. Of course that we all know that's not biologically possible, but in a pure rational, theoretical, mathematician point of view, we cannot be 100%, absolutely, positively sure that everyone will eventually die. This may seem like a pathetic stupid example, but try to abstract yourself from the real situation here, and you will understand the value of this elementary example and apply it in other true, more realistic cases.

Let's get down to it: when are arrays and pointers interchangeable? Let me give you a code example. Imagine, if you will, that you have two files, `file1.c` and `file2.c`. Those 2 files together hold your program. Here's how each one of them looks like:

file1.c:

```c
#include <stdio.h>
#define BUFFER_SIZE 1000

char buf[BUFFER_SIZE];

/*
  Lots of other functions that manipulate buf...
*/
```

file2.c:

```c
#include <stdio.h>
#include <string.h>

extern char *buf;

int main() {
	strcpy(buf, "Hello, world!\n");
	printf("%s", buf);
	return 0;
}
```

When you compile a binary with `file1.c` and `file2.c` together, what do you expect to see as an output when you run it? `Hello, world!`? Are you sure? Let's try it...

```
filipe@filipe-Kubuntu:~$ gcc -o test file1.c file2.c -Wall
filipe@filipe-Kubuntu:~$ ./test
Segmentation fault
filipe@filipe-Kubuntu:~$
```

Woohoo, what a nice surprise! Because you're reading this article, you probably have an idea on what might be wrong, but if you weren't, you can take for granted that more than 90% of those who claim to know C will stare at this with no freaking idea on what the hell is going on here.

This is one very good example where arrays and pointers are not interchangeable. Had we declared `buf` in `file2` as a `char` array instead of pointer to `char`, like this:

```c
#include <stdio.h>
#include <string.h>

extern char buf[];

int main() {
	strcpy(buf, "Hello, world!\n");
	printf("%s", buf);
	return 0;
}
```

And everything would have worked like a charm:

```
filipe@filipe-Kubuntu:~$ gcc -o test file1.c file2.c -Wall
filipe@filipe-Kubuntu:~$ ./test
Hello, world!
filipe@filipe-Kubuntu:~$
```

*What?!*, you say. *How come? I thought arrays and pointers were interchangeable?*. Well, they kind of are. But not always. This is one of the cases where they're not.

Conceptually, arrays and pointers are not the same thing. What's the difference between these 2 declaractions in C?

```c
char *a = "A string";
char b[] = "Another string";
```

The first one declares a pointer to `char`, and it initializes it to point to a string literal. String literals are *constant*: attempting to change the contents of `a[i]` results in undefined behavior. But it's just a pointer, so we can make it point anywhere else. 

The second declaration is creating an array of characters, and initializes it to `Another string`. The difference, apart from this being an array and the previous being a pointer, is that we are allowed to change the contents of `b[i]`. Moreover, because `a` is a pointer, we can make it point to the beginning of `b`, and then we can change `a[i]` just like we would with `b`. But we cannot do it the other way: `b` is not a pointer, it's an array, so we can't make it point somewhere else, because, let me say that again, **`b` is not a pointer, it's an array**. So you can't change what it points to, because it's not a damn pointer! 

I hope that last paragraph was enough to show you why arrays and pointers are different. So, why does everyone think they're not? Because of a rule that states a condition that is met most of the time: the rule is that arrays *decay* into pointers to the first element when used in an expression. 

To better understand the implications of this, let's stop for a moment here and see what this really means at assembly level. Yep, that's right, we're going down to the CPU assembly to see the difference. Well, I'm not going to show you real assembly code, but I can show you a piece of pseudo-assembly code, and that will be enough.

First of all, what *is* a pointer? By definition, a pointer is a variable that holds the address of another variable. It's really that simple. How does the compiler see this?

There's a thing called *symbol table* in a compiler, which is basically a table where each variable name is associated with a memory address. In reality, it's a little more complicated than that, because each entry must hold the variable name, its type, its address, and a whole load of other informations, but for us it suffices to know that it's a name <-> address association. 

When you make an assignment like `x = y`, several things happen. The left side of the assignment is called an l-value, and the right side is an r-value. An l-value is known at compiletime, and it says where to store the result. An r-value is not known until runtime. In an assignment such as this one, the l-value represents an address where something is going to be stored. The r-value represents the contents of an address (that's why it's unknown until runtime). Because `x` is an l-value in this example, `x` really means *The address of `x`*, and `y` means *The contents of the address where `y` is stored*. So, `x = y` means *pick the address of `x`, and store in there the contents of the address of `y`*. This is how the compiler sees it. How does it know each address? It uses the symbol table. Let's assume that `x` is in address `2000`, and `y` is in address `3000`. In other words, this is the compiler's symbol table:

```
x <-> 2000
y <-> 3000
```

Then, `x = y` is translated into assembly code similar to this one:

```
LOAD lvalue(x) into R1
LOAD rvalue(y) into R2
STORE R2 into R1
```

`lvalue(x)` is just `2000`, and `rvalue(y)` will basically pick the address of `y` and read its contents:

```
; code for rvalue(y)
LOAD lvalue(y) into R0
LOAD [R0] into R2
```

And again, `lvalue(y)` is the address of `y`. The square brackets notation, as in `[R0]`, mean *the contents of address `R0`*. The compiler allocates an address (l-value) to each variable, and this address is known at compiletime. However, the value stored in a variable is obviously known at runtime only, so the compiler generates code to read whatever is stored at that variable's address like I showed you above.

So, imagine you declare an `int`:

```c
int my_number = 122;
```

Furthermore, imagine the compiler chooses to store that in address `8100`. The symbol table will now have a new entry, `my_number <-> 8100`. Oh, and we can't forget to initialize it! The generated assembly code will be something similar to:

```
LOAD lvalue(my_number) into R1
store 122 into R1
```

The C programming language introduced the term *modifiable l-value*, which designates an l-value that can appear on the left-hand side of an assignment statement. The C standard stipulates that every assignment's left-hand side must be a modifiable l-value. This term was introduced to cope with array names, which are l-values that locate objects, but are not modifiable l-values, because they can't be assigned to. They are l-values because an array name represents an address, but they're not modifiable, so you can't have an assignment with an array on the left hand-side. That's why you can't write `b = a;` in the strings example above, but you can write `a = b;`.

What happens when the compiler sees a statement like `b[i]`, assuming `b` is a `char` array? The symbol table entries for array names hold the address of the first element of the array. Let's assume `b[0]` is stored in address `5500`. The symbol table entry for `b` is `5500`. `b` is an array, so it means that its address in the symbol table holds a memory location where the characters in the array can be found. Furthermore, because arrays decay into pointers in an expression, the r-value of the array `b` is the address of its first element, unlike other "normal" variables, where their r-value is the content of the address they're stored in. That's why you can't assign to an array and it doesn't work like a pointer: you'd be trying to change a value in the compiler's symbol table in runtime, which makes no sense at all.

Accessing `b[i]` involves:

* Reading the entry for `b` in the symbol table - `5500`
* Adding `i` to that address
* Get the contents of `5500+i`

What happens with pointers? Remember, a pointer is a variable like any other. The difference is that it stores the address of another variable. Consider a pointer to `int`, `int *x`. `x` is a variable like any other, so it has an address. Let's suppose the addreess of `x` is `3369`. What is stored inside `3369`? Well, it's another address, it's the address of an integer, so `x` is a pointer to `int`.

If you see it that way, making `x` point to something, e.g., `x = y`, amounts to:

* Read the entry for `x` in the symbol table - `3369`
* Store `rvalue(y)` in address `3369`

Assuming that the assignment is legal and `y` is a pointer to `int`, `rvalue(y)` is the value stored in the address of variable `y` in the symbol table. What value is that? It's another address. What if we had `*x = 5;` instead? Well, because `x` is a pointer, it holds an address (let's assume, `9981`), so this is translated to:

* Read the entry for `x` in the symbol table - `3369`
* Read the value of `x` into R1 (the value of `x` is whatever is stored in address `3369`, and that value is `9981`, another address)
* Take *that* as an address, and store the value `5` in there.

It is crucial to understand these two distinct processes if you really want to know why arrays and pointers are different. Read that as many times as you need until you are sure to get the whole picture.

We've seen how `b[i]` is accessed when `b` is a `char` array. Going back to our strings example, what about `a[i]`? `a` is a pointer to `char`, it's a variable like any other, so its r-value is whatever is stored in the address of `a` in the symbol table. For example, if it's `2237`, then `a[i]` is accessed like this:

* Read the entry for `a` in the symbol table - `2237`
* Read the value of `a` - another address - suppose `4410`
* Add `i` to `4410`
* Read the contents of `4410+i`

Compare this to the steps for reading `b`. Once more, the value of `b` is just `b`'s entry in the symbol table; `a`'s r-value is the *content* inside the memory address of `a`'s symbol table entry, because `a` is not an array, it's a variable like any other, and it stores the address of another variable. Here's a table comparing the steps taken for indexing `a` and `b`:

| `a[i]`; `a` is `char *` | `b[i]`; `b` is `char []` |
| :---------------------: | :----------------------: |
| Read the entry for `a` in the symbol table - `2237` | Read the entry for `b` in the symbol table - `5500` |
| Read the value of `a` - another address, suppose `4410` | Add `i` to `5500` |
| Add `i` to `4410` | Get the contents from address `5500+i` |
| Read the contents of `4410+i` | |

In other words, the sole use of `b` as an r-value, when `b` is an array, means *pick the address in the symbol table*, whereas the use of `a` as an r-value works just like any other variable: go to symbol table, pick the address of `a`, and load the contents of that address (and, because `a` is a pointer, the *load the contents of that address* yields itself another address).

What happened with `file1.c` and `file2.c` is that `extern char *buf;` is not a declaration, it's a definition. In `file1.c`, `char buf[BUFFER_SIZE]` is a declaration that allocates space for an array of `BUFFER_SIZE` characters. `file2.c` mistakenly thinks that `buf` is a pointer. The end result is that when you do `buf[i]` in `file2`, `buf` is thought to be a pointer, so the compiler goes to the symbol table, picks the address where `b` is stored, loads that address contents, uses *THAT* as another address, adds `i` units, and then tries to dereference. But because `buf` is an array, that second indirection step should have never happened in the first place, so the program tries to dereference an invalid address, and crashes. Pictorially, you can see it like this:

![Array VS Pointer comparison]({{ site.baseurl }}{{ site.assets }}array_vs_ptr.png)

The problem with `file2` is that it uses `buf` as a pointer, so it ends up executing the steps for pointer indexing, when it should execute the steps for array indexing. The erratic instruction is the one in step 2 of pointer indexing: the contents of `8771` shouldn't be loaded before summing `i`. When we declare `buf` as a `char` array instead of pointer to `char`, the compiler will successfully identify `buf` as an array, and will therefore execute the steps for array indexing, so when `strcpy()` indexes `buf[i]`, everything works fine. 

And if you're still with me, you have just reached a new milestone in the highway to understanding C. This is what I love about in-depth C knowledge: to know C really well, you don't just have to know C, you have to know how a CPU works, you have to know assembly, you have to know how a compiler works, and that's why for me it is much, much more impressive and interesting to master C than to master, say, Java, or C#, where you don't have issues like this because every dark detail is hidden from you. And, bear with me, someone who really masters C will catch up with Java or anything similar in a few days and produce *better* code than someone who has been using it for 5 years but doesn't touch C because *It's Too Damn Complex And My Programs Crash Mysteriously*. What about the other way around? Never. Anyone who "masters" Java will never catch up with C in a few days (and no, I'm not saying that Java is rubbish). Go read K&R, and then go read a Java book. Go see the kind of questions they ask in those brilliant Java certification exams. While K&R is implementing a binary search tree and discussing its efficiency issues, the Java folks are asking you what's the difference between an `integer` and an `Integer`, or what is the purpose of `toString()`. Yep. REALLY USEFUL, to know something you can find out online in about 10 seconds.

## When arrays are pointers

Array names decay into pointers to the first element when used in expressions. A function call is an expression, that's why you can declare a function with array arguments as either receiving pointer or array, and in fact, if you declare the argument as array, the compiler will replace it by a pointer declaration.

Every time an array name is used in an expression, its value is replaced by its memory address stored in the symbol table, *and that's just it*. Ordinary variables are usually replaced by the *contents* stored in the memory address in the symbol table. I keep emphasizing that because it's really important. That's also why you can declare a function to receive a pointer to `int`, and either pass it a real pointer to `int` or an array of `int`. If `x` is a pointer to `int` and you pass it to a function call that receives a pointer to `int`, the compiler will go to the symbol table, pick the address where `x` is stored, load the contents of that address, and push that (another address) into the call stack as an argument for the function. However, if you call the function with a real array `y`, the compiler will merely pick the address that is associated with `y` in the symbol table and push it directly into the call stack as an argument. *The 2nd indirection step is just not there*, and it's as simple as that. The bottom line is that most of the time, the compiler knows how to make the distinction transparently for the programmer, but if you don't know what's going on under the hood, sometimes you may fall into a trap.

Oh, I'm sorry, I must rephrase that, or nitpickers will eat me alive. There are exceptions to the pointer decayment rule. An array name in an expression is not replaced by a pointer to the first element in 3 situations:

* The array is the operand of `sizeof()` - it computes the size of the whole array instead of the size of a pointer.
* The array's address is taken with the `&` operator. If `x` is an array of 20 integers, `&x` has type `int (*)[20]` (pointer to array of 20 integers), *not* `int **`
* The array is a string or wide-string literal initializer.

The pointer decayment rule is not recursive; if you have

```c
int arr[10][19][20];
```

Then `arr` used in an expression decays into *pointer to `array[19][20]` of `int`*, namely, `int (*)[19][20]`. `arr[0]` decays into `int (*)[20]`. Note that this is different when you have an array of pointers. That's the case, for example, with `argv`, which is an array of pointer to `char`, that's why you can declare it in `main()` as `char *argv[]` or `char **argv`: `argv[0]` has type `char *`, so when you use `argv` in an expression, it decays into pointer-to-first element, which is pointer-to-pointer-to-char, `char **`.

As a consequence of this rule, every array indexing `a[i]` is rewritten as `*(a+i)` at compile time (and again, even though the code looks the same for the case when `a` is array and `a` is pointer, remember that for the first case, the compiler replaces `a` by its address in the symbol table; in the latter, it must go to the address in the symbol table, and load the contents of that address, which is another address). A good approach is to see the brackets operator as an operator that takes an integer and pointer-to-type-`T` and yields an object of type `T`. Because it is rewritten to `*(a+i)`, it works like the addition operator. In particular, it is commutative, so `a[i]` and `i[a]` are both valid C. This means that code like this:

```c
int a[] = { 1, 2, 3 };
int x = a[2];
int y = 1[a];
```

Compiles and works properly, though it looks really weird and is rarely seen.

The compiler must scale the value `i` according to `sizeof(a[0])`, if `a` is an array of `int`, it will typically scale `i` by 4 to meet the proper memory location. This is why you can only drop the size of the first dimension of an array in function argument declarations:

```c
void f(int a[][19][20]); /* Valid, is rewritten to int (*)[19][20] */
void f(int a[][][]); /* WRONG */
void f(int a[][][20]); /* Also wrong, you can only drop the size of the first dimension */
```

Multi-dimensional arrays are sequentially stored in memory, hence the compiler needs to know the size of every dimension after the first. In the example above, `a[0]` is an array of 19 arrays of 20 integers. `a[0]` uses `19*20*sizeof(int)` bytes, that's the scaling factor for `i` in `a[i][4][5]`, for example. `a[4]` is `4*19*20*sizeof(int)` bytes away from `a[0]`.

A corollary of this is that it is pointless to declare the size of the first dimension, and in fact, the compiler completely ignores it.

It's your call to learn C properly. Learning C and understanding every nitty gritty detail of it is an art. For me, mastering C is a must for anyone who claims to be a good computer scientist (that's such a horrible title. As Gerald Sussman says, it's not a science, and it's not about computers). To master C is to master your skills, is to master your head. And before you go, remember that your CPU is always there for you and is never tired, and it is your job to use it every day and every hour to learn new, fascinating stuff that will make you a better professional and will make you happy.
