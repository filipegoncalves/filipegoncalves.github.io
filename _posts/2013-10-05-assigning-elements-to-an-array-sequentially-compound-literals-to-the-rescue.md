---
layout: post
title: Assigning elements to an array sequentially - compound literals to the rescue
---

C has compount literals, and they are useful!

-----

We're in 2013. Technology keeps expanding. Software is not provided as a stand-alone, off the shelf closed box with an installation CD anymore. Instead, we have the concept of software as a service, delivered in the web. Our files are in the cloud. Mobile market is growing, everything is "smart": you have smart TVs, smartphones, and, guess what, now you even have smartwatches. Whether the world is getting smarter or not is a mystery, I don't know if there are more smart people out there than a few years ago, but there surely are more smart objects laying around.

As time goes by, technology evolves, and some systems just die. Remember MSN? Dead. Everyone is using Facebook chat now. I wonder what everyone will be using in the future, after facebook dies. Remember IRC?

Jeeeeeeeeeeeeeez, IRC! Sweet mother of god, that is SO old. IRC was created in 1988, it's literally something from *another century*. It is totally dead.

Is it? Is IRC dead? Quick answer: absolutely not. I've been using IRC since I'm 13, and still do after 8 years. Sure, it is dead for most of the people who used it just for the fun of having a nice refreshing chatting session with someone else in the other side of the planet, but it's not dead for us, programmers.

In IRC, you can find some of the most hardcore, geek, expert programmers that you otherwise would never find, and exchange some really interesting ideas. This article is about something I learned a few days ago in freenode. Someone was having problems understanding this piece of code:

```c
#define MAX_STRING_SIZE 1024

char *p_string(char *buf, int pos) {
    /* does some stuff */
    return buf;
}

#define err_str(pos) p_string((char[MAX_STRING_SIZE]){0}, pos)
```

The question was about what exactly happens with the first argument to `p_string()` when it is called with the `err_str()` macro. In other words, what kind of sloppy way is that to call a function? What does `(char[MAX_STRING_SIZE]){0}` even mean? Is it allocated in stack or heap? 

I am pretty confident with my C knowledge, but when I saw that code, I must admit it was, indeed, intimidating. I even opened up a terminal just to be sure it compiled. 

Well, even though we don't exactly know what's happening, it's more or less obvious that it is somehow passing a char array with `MAX_STRING_SIZE` chars to `p_string()`. The programmer asked if it was being allocated in stack or in heap. How would you check it quickly?

If you have ever studied compilers and operating systems, you probably know that a binary file is composed of several segments: the BSS segment, the data segment, and the text segment. BSS stores information for every global and static variables that are not initialized. Many people like to think of BSS as an acronym for "Better Save Storage", because it just saves how many bytes each variable will need, as opposed to the data segment, which holds information for initialized global and static variables. So, for example, if you declare a global array of 1000 integers and do not initialize it, BSS will have an information saying "We will need `1000*sizeof(int)` bytes for this variable". On the other hand, if you initialized it, the data segment would really have to hold every 1000 integers and set their value accordingly. The text segment is read-only, and it's where executable instructions are placed.

Local function variables are allocated in stack. The heap can provide way more memory than the stack, so a nice quick way to see if our char array is using heap or stack storage is to set `MAX_STRING_SIZE` to be a very big number. Something like 10,000,000 will do. When you run it, does the program crash with segfault? If it does, then it's allocated in stack - assuming 8-bits chars, 10,000,000 chars will use about 10 megabytes, no stack can grow so much. Heap storage is a lot bigger, and it can easily handle a 10-million char array.

Using this technique, I could see that the parameter was allocated in stack. Within minutes, the magic trick was revealed: this is called a compound literal. You can [read about it](http://gcc.gnu.org/onlinedocs/gcc/Compound-Literals.html#Compound-Literals) in gcc's online manual. It's a C99 feature, which is why I had never heard of it. I still have to go and update myself with C99.

Compound literals work exactly like any other automatic variable declared in the innermost current block. Compound literals syntax is the same as the one you use for type casts, with an additional number suffixed in curly braces. This is the initializer, and it must always be there. As usual, for arrays, if the initializer size is less than the array size, the remaining positions are initialized to 0. Thus, `(char[MAX_STRING_SIZE]){0}` is a compound literal that creates a `char` array with `MAX_STRING_SIZE` characters and initializes every position to 0.

This means that `err_str(x)` will call `p_string()` with an anonymous local automatically allocated variable. `p_string()` returns its first argument, so, by doing `char *s = err_str(...);`, you'll have access to this anonymous variable. And it's clean, it even looks like `err_str()` is allocating space for you, but it's not, you're just creating one more local variable inside the current block.

It's a cool feature. Not super-cool, there are far more crazy things in C besides this, but it's cool. 

With this in mind, I challenge you to run this code and understand how it works:

```c
#include <stdio.h>
#define ASSIGN_ARRAY(dst, type, ...) { *(struct tmp_ ## type ## _ ## dst { type x[sizeof (dst) / sizeof (*(dst))]; } *)(dst) = (struct tmp_ ## type ## _ ## dst){{__VA_ARGS__}};}

int main() {
	int a[5];
	size_t i;
	ASSIGN_ARRAY(a, int, 1, 2, 3, 4, 5);
	for (i = 0; i < sizeof a / sizeof *a; ++i) {
		printf("%d\n", a[i]);
	}
	return 0;
}
```

The `ASSIGN_ARRAY()` macro allows you to sequentially assign a list of elements to an array, just as you're used to do when using initializers. It is a powerful macro: for an array of size `N`, you can provide a list of `N` elements that will be assigned to positions `0` through `N-1` or you can provide a list of `X` values, with `X < N`, in which case positions `0` up to `X-1` will be filled with the values provided, and positions `X` up to `N-1` will be zeroed out.

Cool, huh? This could be made without compound literals, but we would have to declare and name a variable inside the macro block, and initialize it with `__VA_ARGS__`. It would work, but it wouldn't be as concise as this solution.

Basically, it works like this: we declare a structure `T` composed of a single field: an array of exactly the same size and type as the array we want to assign to. Note that structures are passed by copy. When you have code like

```c
struct my_struct { int x[100]; };
/* .... */
struct my_struct example;
/* Fill "example"'s array...*/
struct my_struct y = example;
```

The last assignment will copy the entire memory contents of `example` to `y`, so the whole array is copied. Wrapping an array inside a structure is a clever way to force pass by copy and run away from arrays decaying into pointers.

With that in mind, it is easy to understand that if we create a structure to wrap an array of the same type and size and initialize it with the list of arguments passed to the macro, we are left with a structure wrapping an array with the desired values. This is what the right hand side of our macro is doing. Remember, if you will, that macros accept the operator `##` to concatenate text, so `tmp_ ## type ## _ ## dst` is the same as `tmp_type_dst`, with the difference that the preprocessor will replace `type` with the corresponding argument (had we not used the concatenation operator, we would end up with `tmp_type_dst` literally). 

Thus, the right hand side of the assignment declares an anonymous instance of a structure called `tmp_type_dst` using compound literals, where `type` and `dst` are replaced by the first two arguments. In the sample code above, the right hand side is expanded into `(struct tmp_int_a){{1,2,3,4,5}};`. The double curly braces are necessary. Recall that compound literals syntax is `(type)INITIALIZER`. Structure fields are split with commas, and arrays are themselves initialized with the values inside a pair of curly braces. Consider the following structure:

```c
struct x {
    int a;
    float b;
    int c[10];
};
```

To declare and initialize an instance of `struct x`, you'd have to do this:

```c
struct x test = {10, 0.3f, {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}};
```

If the structure `x` had only one field - the array - then we remove the first 2 elements in the initializer; initialization would look like this:

```c
struct x test = {{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}};
```

Ok, that was the right side of the assignment inside the `ASSIGN_ARRAY()` macro. The left side can be a little trickier to understand for not so experienced programmers, because it is doing a lot of work in the same line: it is declaring a structure, casting the array name to a pointer to that structure, and dereferencing it. Let's see it carefully:

```c
*(struct tmp_ ## type ## _ ## dst { type x[sizeof (dst) / sizeof (*(dst))]; } *)(dst) = ...
```

First of all, just for visual clearance, let's remove the concatenation operator:

```c
*(struct tmp_type_dst { type x[sizeof (dst) / sizeof (*(dst))]; } *)(dst) = ...
```

With the earlier sample code I showed, this will expand to:

```c
*(struct tmp_int_a { int x[sizeof(a) / sizeof(*(a))]; } *)(a) = ...
```

So, what's happening here? Type cast has higher precedence than dereferencing, so the first operation perfomed is to cast `(a)` to `(struct tmp_int_a { int x[sizeof(a) / sizeof(*(a))]; } *)`. If we take off the structure fields declaration, this is the same as `(struct tmp_int_a *)`. So, basically, we're casting `(a)` to `struct tmp_int_a *`. But because we never defined the structure in the first place, we need to do it, so we just define it at the same time. That's right - we're defining the structure as we cast `(a)` to a pointer to that structure. As I noted previously, the structure is comprised of a single field, and that is an array of the same size and type. So, instead of `(struct tmp_int_a *)`, we must use `(struct tmp_int_a { int x[sizeof(a) / sizeof(*(a))]; } *)`. We simultaneously define and use the structure. Cool, huh?

Because array names decay into pointers to their first element, this actually has a chance to work. If `a` is an array of `T`, we will trick the compiler and make it believe that `a` is, instead, a pointer to a structure containing an array of `T`. So, `*a` is a structure containing an array of type `T`. In other words, the expression

```c
*(struct tmp_int_a { int x[sizeof(a) / sizeof(*(a))]; } *)(a)
```

has type `struct tmp_int_a`, and because it is the left hand side of an assignment, the contents of the right hand side will be copied to this structure. And voila, you copied an entire array with an assignment.

Now, please, be careful when using this. It's an evil macro, and sometimes it can cause weird behavior. It is convenient to point out that this will only work with true arrays. You can't call `ASSIGN_ARRAY` with a pointer, even if it originates from an array being passed to the current function, because `sizeof(a)` will be the size of a pointer, and it must be the size of the whole array.

Other issues can arise because of padding. I ran several tests in different machines and didn't find this to be a real problem, but it is not necessarily true that `sizeof(struct tmp_type_dst)` is the same as `sizeof(array)`. K&R says this about structures, in chapter 6, section 6.4:

> Don't assume, however, that the size of a structure is the sum of the sizes of its members. Because of alignment requirements for different objects, there may be unnamed "holes" in a structure. Thus, for instance, if a char is one byte and an int four bytes, the structure
>
> struct {
>  char c;
>  int i;
> };
>
> might well require eight bytes, not five. The sizeof operator returns the proper value.

While this is definitely true for structures with more than one element, it is generally not the case for structures with a single element. It is almost certain that `sizeof(struct tmp_type_dst)` is the same as `sizeof(array)`, but the standard says that the compiler is free to pad a structure with a few extra bytes in the end. If that happens, then `sizeof(struct tmp_type_dst) > sizeof(array)`, and casting the array will cause undefined behavior because the compiler now thinks that `array` uses more space than it actually does, so it can end up accessing invalid memory locations that would otherwise be valid for a real `struct tmp_type_dst`, and no one knows what happens.

This is a somewhat nice example to demonstrate the use of compound literals, but keep in mind the portability issues that I just described.

Stay tuned!
