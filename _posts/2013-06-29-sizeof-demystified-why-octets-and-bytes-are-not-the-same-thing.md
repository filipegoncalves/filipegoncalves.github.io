---
layout: post
title: sizeof demystified - why octets and bytes are not the same thing
---

The confusion around what a byte is

-----

You know what `sizeof` does, right? And you do know what it returns, and why it is important to help you keep your program portable and fine-tuned for the most sloppy, highly weird, customised super duper platform that your code will ever have to run in.

It's all over the place: websites, blogs, books. Even in K&R, by page 135, you can read this about the `sizeof` operator:

> C provides a compile-time unary operator called `sizeof` that can be used to compute the size of any object. The expressions `sizeof object` and `sizeof(type name)` yield an integer equal to the size of the specified object or type in bytes.

This sounds fair enough. Suppose `sizeof(int)` is 4. Well, `sizeof` returns a size in bytes, a byte is 8 bits, so an integer uses 32 bits. Right? It's very straightforward to understand this. So, for any type name, we know that `sizeof(type name)*8` gives us how many bits are used to represent `type name`.

Wrong.

This is one of those unfortunate situations where the same name is being used to refer to different things. And we, computer guys, simply hate that, because ambiguous names and expressions are a programmer's number 1 enemy. What I want to tell you is that, in theory, it CAN happen that `sizeof(int)` yields 4 in machine `A` and `B`, and still, machine `A` uses 32 bits to represent an `int`, while machine `B` uses 64. Before moving on, I should make something clear before I forget to mention it. As you can see above, `sizeof` is an unary operator evaluated at compile time, meaning, it is not exactly a function, so it doesn't "return". In a purely pedantic perspective, it doesn't make any sense at all to talk about what `sizeof` "returns", because only functions return, and `sizeof` is not a function. So, from now on, I will say "yields" rather than "returns". Anyway, if I happen to use "returns" somewhere, you get the point, and it surely won't hurt too much.

Back to the `sizeof(int) == 4` in machine `A` and `B`. How is this possible? Am I kidding you? I mean, come on, if `sizeof` is the same in both machines, then they MUST use the same number of bits to represent it. Yes? No?

The problem is that for you, a byte is something that is 8 bits, and for gcc, that may not necessarily be true. When you think byte, you automatically think 8 bits. You just hardcoded that value, with a `#define`, inside your brain.

According to the ISO/IEC 9899:1999 standard, this is what you should think when you hear the word "byte":

> 3.6. byte: addressable unit of data storage large enough to hold any member of the basic character set of the execution environment.

Woha! So, basically, it all goes down to the old 1-char-is-8-bits myth that the fairy tale told you at night. But don't fall for that crap. That's just plain **wrong**.

So that's what a byte is. An octet is 8 bits. But this is not always the case with a byte.

So, coming back to our example with machine `A` and `B`, hopefully, you can now understand why an `int` can be represented with a different number of bits even though `sizeof` gives the same result in both machines. Just think of `sizeof` as something that lets you know how many chars you would need to represent something of a specific type. `sizeof(int)` being 4 means that an `int` takes as much space as 4 chars, and a `char` is the basic and smallest unit in C you can have. `sizeof(char)` is guaranteed to be 1 on every machine, regardless of the number of bits that compose a char. This is consistent with our definition of byte: if a byte is something large enough to hold a `char`, then `sizeof(char)` can only be 1!

Now, suppose that machine `A` uses 8-bit chars. If `sizeof(int)` is 4, then an `int` uses \\(4\times8 = 32\\) bits. What if machine `B` doesn't use 8-bit chars? Imagine it needs 16 bits to represent a `char`. Even though `sizeof(char)` is 1, each char takes up 16 bits. So, in machine `B`, a byte is 2 octets (16 bits). But again, `sizeof(int)` is 4, meaning, we have \\(16\times4 = 64\\) bits integers. So there you have it: an example of two machines where `sizeof(int)` is the same, but different number of bits are used.

What if you really want know how many bits are in a `char`? You can use the `CHAR_BIT` constant from `limits.h`, and find out how many bits are used for an `int`:

```c
#include <limits.h>
#include <stdio.h>

int main() {
  printf("int uses %d bits.\n", sizeof(int)*CHAR_BIT);
  return 0;
}
```

Or, if you want to compute `CHAR_BIT` by yourself, you can do this:

```c
#include <stdio.h>

int char_bits(void);
int main() {
  printf("int uses %d bits.\n", sizeof(int)*char_bits());
  return 0;
}

int char_bits(void) {
  unsigned char c;
  int j;
  for (j = 0, c = ~0; c; j++, c >>= 1);
  return j;
}
```

Let's turn our attention to the `sizeof` "return" type. `sizeof` doesn't return sizes as an `int`. The `sizeof` operator returns an unsigned integer type required to be at least 16 bits. This type is known as `size_t`. `size_t` is an unsigned integer type that is big enough to hold the size of any object. It is common that `size_t` is nothing more than something like

```c
typedef unsigned int size_t;
```

But obviously, that's not necessarily true. It depends on the implementation. Every function that returns or expects sizes (like `malloc`), uses `size_t`. There are some interesting facts worth mentioning about `sizeof`. The first one is that passing it an object of type `T` or passing it type `T` directly is absolutely irrelevant. That is, in this code:

```c
int a;
printf("%zu %zu", sizeof(int), sizeof(a));
```

`sizeof` is guaranteed to be the same for both cases, because when you pass it an object, you get the size of the type of that object. Normal pointer dereferencing works as expected. Let's pick a somewhat harder to read declaration that will allow us to illustrate important facts about `sizeof`. Imagine you have something like

```c
char (*(*(*a)(int (*)(char, int, float), char **))[12])(int);
```

Anyone who claims to know C must know how to read this, and must have the mental capability of dealing with it. I know that this will hardly ever be used in real life with production code, but it's valid C code and, in theory, you *can* use it, even if it's a nightmare for maintainability.

If you have no clue what this is, I advice you to read section 5.12 on K&R, where a very interesting program (dcl) is developed that translates these declarations into words. It starts on page 122. Just to speed things up a little bit, I'll interpret this for you. We're declaring a variable named `a`. I'm going to describe `a` using some indentation to help your eyes reading my definition. So here it goes:

```
a is a pointer to a function receiving:
    - A pointer to a function receiving: 
        - a char, an int, and a float
        - and returning an int
    - A pointer to a pointer to a char
And returning a pointer to an array with 12 pointers to functions receiving int and returning char.
```

That's not very complicated, is it? I mean, I could have done it harder. I could have made it to return a pointer to a function that returns a pointer to an array with 12 pointers to pointers to functions receiving `int` and returning a pointer to a function receiving a multi-dimensional array. So, don't complain about my simple, highly readable, easy-to-use example.

Then you know that these are equivalent:

* `sizeof((*(*(*)(int (*)(char, int, float), char **))[12])(int))`
* `sizeof(a)`

Now, imagine that `b` is a pointer to a function receiving `char`, `int` and `float` and returning `int`, and that `argv` is the normal bi-dimensional array passed to `main()` with the program arguments. Then these are also equivalent:

* `sizeof((*a)(b, argv))`
* `sizeof(char (*(*)[12])(int))`

Since the type of the first expression is pointer to an array of 12 pointers to functions receiving `int` and returning `char`. Furthermore, you actually know that

* `sizeof((*(*(*a)(b, argv))[3])(9))`

Is 1, since its type is `char`. This is easily checked with a small example program. If you don't believe me, do it, it's pretty cool to actually see it working. Check it with your own eyes:

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
	char (*(*(*a)(int (*)(char, int, float), char **))[12])(int);
	printf("%zu %zu\n", sizeof(a), sizeof(char (*(*(*)(int (*)(char, int, float), char **))[12])(int)));
	printf("%zu %zu\n", sizeof((*a)(NULL, NULL)), sizeof(char (*(*)[12])(int)));
	printf("%zu\n", sizeof((*(*(*a)(NULL, NULL))[0])(0)));
	return 0;
}
```

In my machine, this prints:

```
4 4
4 4
1
```

This can make you think that all pointers have the same size, but that's not entirely true. Although it is extremely uncommon nowadays, different pointers can have different sizes; there's nothing in the standard preventing that from happening. I mean, virtually every desktop has got the same size for any kind of pointer, but different memory schemes can define different pointer sizes depending on what they point to. It depends on the [memory model](http://en.wikipedia.org/wiki/Flat_memory_model). But because any pointer can be casted to `void *`, and casted back to its original type without loss of information, then you know that `sizeof(void *)` must be enough to hold the largest pointer type. Function pointers are an exception to this (for example, C in DOS had 16-bit function pointers and 32 bits data pointers), and even nowadays, it is not recommended to cast function pointers to `void *` and vice-versa. The standard says that function pointers can be casted to any other type of function pointers (for example, you can cast a `char (*)(int, int)` to a `double (*)(int)`), but you should always set the function pointer to the correct type before dereferencing it. Maybe this means that the size of any function pointer is always the same, even though I never read that anywhere.

The example code posted above also emphasizes an interesting property about `sizeof`. Remember, `sizeof` is a compile-time unary operator (except when used with variable-length arrays), meaning that the code you pass it as an argument is not executed. That's why you can use pointer dereferencing and array indexing without needing to initialize them, because the code is not executed. Think of it like this: the compiler will basically "predict", or infer, the type of the result of the code you give it. If `X` is a pointer to `int`, it doesn't matter if you initialized it to `NULL`, or if you never initialized it; `sizeof(X)` will basically give you the size of `int *`, and `sizeof(*X)` gives you the size of an `int`, because `*X` is an `int`, and that's basically it.

The sizeof operator can be used to get the size of an array. When applied to an array, `sizeof` computes the total size of the whole array. Thus, the following expression

```c
sizeof(array)/sizeof(array[0]);
```

computes the number of elements in `array`. Indeed, this program:

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
  int arr[225] = { 0 };
  printf("%zu\n", sizeof(arr)/sizeof(arr[0]));
  return 0;
}
```

Will print 225. Nice! Can we make a function that computes the number of elements in an array? After all, apparently it's just a matter of putting

```c
sizeof(arr)/sizeof(arr[0])
```

Inside a function. Let's give it a try.

```c
#include <stdio.h>
#include <stdlib.h>

size_t array_count(int arr[]) {
  return sizeof(arr)/sizeof(arr[0]);
}

int main() {
  int arr[225] = { 0 };
  printf("%zu\n", array_count(arr));
  return 0;
}
```

The first problem is that you have to declare a type for `arr`. Well, that's not very cool, because you'd like your function to be able to compute the size of any array holding any type of objects. And, using my previous highly contrived example, you'd want the expression

```c
array_count(*(*a)(int (*)(char, int, float), char **)))
```

To evaluate to 12 without needing to declare `arr` as `char (*)(int)`.

Still, let's leave this issue aside for now. If you run this code, it will print 1, not 225. Or some other value. But not 225.

What's happening here? Well, inside `main()`, `arr` is known to be an array of 225 integers. And even though you know that it's a pointer to the first array element, i.e., `arr` is of type `int *`, and `*arr` is the same as `arr[0]`, you also know that `arr` is an array of 12 integers, and arrays are not the same thing as pointers, because they are constant, i.e., you can't change the location they point to. You can't execute things like `arr++;`. Consider this piece of code:

```c
int array[10] = { 0 };
int *a = array;
a++;
```

What's the difference between `array` and `a`? `a` is of type `int *`, whereas `array` is of type `int [10]`. `array` is really a true array. `a` is just a pointer to some `int`, and you can make it point anywhere, anytime. You can increment it, decrement it, make it point to the same place as another pointer, etc. You can't change `array` (but you can change elements inside it). And you can't assign `array` to anything else. Remember, this is inside `main()`.

What happens when you pass `arr` to a function? Well, parameters are passed by copy, so what you get for `arr` inside `array_count` is a normal pointer to `int`, and you have no clue if this pointer is really an array, or if it's a "normal" pointer to an `int`. In fact, the compiler translates `array_count(int arr[])` into `array_count(int *arr)`. So, when you call `array_count`, you can call it with `a` or `arr`, since `a` points to the same place as `arr`. This is a common pitfall that makes beginners very confused, but you just have to keep this in mind:

**Arrays and pointers are not 100% interchangeable** (although most of the times, they are).

At this point, you may be wondering how can we fix our `array_count` function. *Well*, you think, *if the problem is that parameters are passed by copy, I can pass it by reference*. What do we do in C when we don't want a copy to be passed? We send the memory address of the object (that is, the pointer), of course!

This does not work the way we want. Although it might be a good idea, the problem is that a pointer to `array` has type `int (*)[10]`, so, you'd have to specify the array size in `array_count` arguments list, which is what we were trying to compute in the first place, so this makes our function absolutely useless. Or, if you don't specify the size, you will end up with `int **`, getting the same problem recursively. Pretty cool, huh?

You may have come up with a different solution to this problem that uses macros. Because macros are preprocessed before the code is actually compiled, avoiding normal function calls overhead, this seems to be the perfect solution to our `array_count` problem. And you don't even have to declare a type for array! Perfect. You think about something like this:

```c
#define array_count(arr) (sizeof(arr)/sizeof(arr[0]))
```

And you're right. This works perfectly with any code that deals with arrays of any type: arrays of `int`, arrays of `float`, arrays of pointers to functions receiving `int` and returning `char`, as our previous example. Thus, this code:

```c
#include <stdio.h>
#define array_count(arr) (sizeof(arr)/sizeof(arr[0]))

int main(int argc, char *argv[]) {
  char (*(*(*a)(int (*)(char, int, float), char **))[12])(int);
  printf("Array has %zu elements.\n", array_count(*(*a)(NULL, NULL)));
  return 0;
}
```

Will correctly print `Array has 12 elements.`. And this code:

```c
int main() {
  int arr[225];
  printf("Array has %zu elements.\n", array_count(arr));
  return 0;
}
```

Will correctly print `Array has 225 elements.`. Macros to the rescue!!!

There is, however, a point to keep in mind that brings some limits to our solution. `array_count` will only work if you "call" it inside the function that declared the array. This happens because macros are a simple text replacement tool in a pre-compilation step, it's as if you manually typed `sizeof(arr)/sizeof(arr[0])` every time you call `array_count`. So, if you do it with an array that was received as a function parameter, you will be back to our earlier problem of arrays vs. pointers.

Nevertheless, this might become handy in some situations, and it surely is the only proper way of doing it.
