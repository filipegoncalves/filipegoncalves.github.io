---
layout: post
title: The magical container_of macro and its use in the linux kernel
---

Yes, we can have type-agnostic linked lists in C

-----

*The Linux Kernel Primer - a Top-Down Approach for x86 and PowerPC Architectures*, by Claudia Rodriguez, Gordon Fischer and Steven Smolski, starts with a pretty damn good foreword section:

> «Here there be dragons». Medieval mapmakers wrote that about unknown of dangerous places, and that is likely the feeling you get the first time you type:
>
> `cd /usr/src/linux ; ls`
>
> "Where do I start?" you wonder. "What exactly am I looking at? How does it all hang together and actually work?"
>
> Modern, full-featured operating systemas are big and complex. [...]


The linux kernel is a beast itself, a mix of C and assembly, with binary trees everywhere, linked lists, hash tables, memory management stuff, filesystem, scheduler, and lots, lots and lots of other pieces. It's huge! And when you try to explore it, in the beginning, it's really scary. You feel very, very small, powerless and insignificant when you're faced with such a monster.

Diving into the kernel brings new challenges, new knowledge, and is one of the most hardcore stuff to explore. It also means that it's beautiful and extremely exciting to know how it all works.

I'm reading chapter 2 of that book, where we go over the basic data structures and tricks used throughout the rest of the code. I haven't had much time for those hobbies lately, because I'm doing a summer internship, so I still wasn't able to compile the `hellomod` example in the end of chapter 2. It's a simple "hello world" kernel module, 20 lines of code, just to print `Hello, world!` when it's loaded and `Goodbye, world` when it's unloaded. It just won't compile. I don't really know what's happening, but I think it has something to do with the fact that there are errors in the source code provided in the book (for example, they have `#MODULE_LICENCE("GPL");` in the source, as if it were a preprocessor directive, and the word *license* is mispelled, seriously, how do these things come out in a book?) plus I'm using kernel 3.2.5 and the book is for 2.6 series, PLUS the kernel is compiled with grsecurity.

Oh, and a few minutes ago I just found that my kernel is not compiled with `CONFIG_MODULES` enabled. So now I have to recompile it.

Grrrrr!

ANYWAY...

Today, I want to show you something that you probably never thought of, unless, of course, you work in the linux kernel. 

The kernel defines arbitrary, generic data structures that can be included within other structures. For example, it defines a double linked list data type in `include/linux/list.h`.

Last semester, my software engineering teacher told me and the other students in the class that the linux kernel source was pretty well organized given the poor features of C. He said that even though C is not an object oriented language, they were able to make an extendable, maintainable, readable code with emulated versions of OO design techniques.

*Well*, I thought, *that should be the rule, not an exception. Why is he making such a big deal out of this?*. I mean, come on, object oriented languages didn't save the world, C has been around ages before. Object oriented design stuff basically goes down to a bunch of terms like inheritance, polymorphism, overloading, ... what's that compared to, say, pointers, or the undoubtful power of macros (but really, pointers are more powerful)? But wait, there's more to it! C++ can make it extremely hard for you to track down what is happening with a statement like

```c
i = j+2;
```

Because `j` may be overloading the operator `+`, so you have to go and see the type of `j`, and, jesus, what if `j` is an instance of a superclass `X`? And god help you if `X` inherits from `Y` and `Z` and `W` (yeah! multiple inheritance party!), and then you have polymorphic calls for the operator overloading, and each of the polymorphic functions makes type conversion because `j` is not really an `int`, and it may even be the case that its is only known at runtime.... and .......... oh god. WHAT A MESS. 

This doesn't happen in C. Well, technically it does, but it is well documented. In C, `+` works well with different types - the fact that you can apply the sum operator to an `int` and a `float` *is* operator overloading - but in C this is part of the standard and is actually pretty easy to know when and how it works. And you can't overload it for your structs, for example. So, yeah, C is way more convenient to deal with in this kind of situation.

I'm not saying that OO languages are difficult, or useless, or crappy, no, I'm not. I'm just pointing out some of its primary drawbacks, because everyone tends to think OO programming is the future, C will go away, and OO solves all of our problems while keeping code clean, simple and easily extendable. No, it's not that simple. C won't go away, and OO design will not be there for everything because its powerful features are also its greatest weaknesses. We can't be delusional like that.

Either way, if you have ever developed something in Java, or C++, or, for that matter, any object oriented language, you must know about parametrized types. For example, in C# you can literally declare a list of *something*. They just have a built-in generic list type that you can use with any of your classes. This technique is usually referred to as template programming.

It's a very good exercise to see this implemented in the linux kernel. The linux kernel, written in C, accomplishes the same thing with the sole use of the power and magic of macros. Let me rephrase that:

*There's a generic list type in C implemented in the linux kernel*

You shouldn't be impressed by this. Or should you?

I remember in my freshman's year, when I had an algorithms and data structures course, that we sat down and looked, as a visitor, as someone who was really far far away, at a possible way of doing this: by using `typedef`. That must have been your first thought. We just define a list structure, with pointers to `next` and `prev`, and one of the structure fields is the actual value in that node of the list, and its type is `Item`, then it's a matter of creating a type alias `Item`, and tada! - we have a list of `Item`.

Yep. REALLY USEFUL.

There's just one slight problem with it. What if I want a list of `my_struct`, and also a list of pointers to array, oh and I'm also going to need a list of `char *`, and a list of pointers to functions. Sorry, `Item` can't be all of that at the same time!

The kernel developers made it in such a way that this is possible. In C! This is amazing. How on earth could they do that? Is that even possible?! C doesn't have parametrized types! It's not possible!

Yeah, again, macros to the rescue. I will let you think for a week, and next week I'll let you know. BWHUAHAHUAUHAUHAUHAHUAUHAUHAUHA!

Nah, I wouldn't do that. 

First of all, the secret is that you don't actually store an item in the list node structure. Let's do it the other way: every structure that is to be part of a list must include a list struct. So for example, if you're going to want a list of `my_struct`, then `my_struct` will have a field that is a list node. Of course, there's no free lunch: if you want a list of a primitive type, you are forced to define a new struct just to hold that type and a list node. Live with it.

Let's call the list node structure `list_head`. This is a very simple structure:

```c
struct list_head {
    struct list_head *next, *prev;
};
```

You can have a look at it in `include/linux/list.h`.

There are hundreds of macros defined in that file that manipulate this generic list type. Three of them are used to create an empty list:

```c
#define LIST_HEAD_INIT(name) { &(name), &(name) }

#define LIST_HEAD(name) \
    struct list_head name = LIST_HEAD_INIT(name)

#define INIT_LIST_HEAD(ptr) do { \
    (ptr)->next = (ptr); (ptr)->prev = (ptr); \
} while (0)
```

As you can see, an empty list is defined as one where `head->next` points to the list's head element. In other words, an empty list is a `list_head` that points to itself. You can either declare a new list by calling `LIST_HEAD(lst)`, or if you already declared it before, you can initialize it with `INIT_LIST_HEAD(your_list)`.

Ok. So far, so good. You're probably wondering what the heck is that `do { ... } while(0)` thing in `INIT_LIST_HEAD`. This shows up a lot of time in the kernel macro definitions, and it is a way to create a block statement instead of a compound statement. If this doesn't tell you anything, let's try it another way. It's a way to define a block of code where you can declare local independent variables, but instead of being a block, it's a statement in C. The natural way to implement `INIT_LIST_HEAD` would be:

```c
#define INIT_LIST_HEAD(ptr) { (ptr)->next = (ptr); (ptr)->prev = (ptr); }
```

This creates a compound statement. It is troublesome if you have code like this:

```c
if (something)
    INIT_LIST_HEAD(ptr);
else
    /* do something else, who cares... */
```

If we used the compound statement thing, this would expand to

```c
if (something)
    { (ptr)->next = (ptr); (ptr)->prev = (ptr); };
else
    /* do something else, who cares... */
```

which is wrong because of that final `;` after the closing brace `}`. This `;` effectively closes the `if`, so you're left with an `else` without a previous `if`. It won't compile. On the other hand, if you use the original version, the one with the `do { ... } while(0)` hack, it will expand to:

```c
if (something)
    do { 
      (ptr)->next = (ptr); 
      (ptr)->prev = (ptr); 
    } while (0);
else
    /* do something else, who cares... */
```

and that is not problematic at all because the semi-colon is closing the `do {...} while` block, not the `if`, so you didn't change the semantic meaning of the program. In fact, this is so common that gcc added an extension to the C language, allowing the same thing to be achieved without the `do {..} while(0)` trick. It's called, guess what, Statement-expressions. Using this extension, `INIT_LIST_HEAD` would be defined as:

```c
/* ATTENTION - NOT ANSI C! Only works with gcc */
#define INIT_LIST_HEAD(ptr) ({ \
    (ptr)->next = (ptr); \
    (ptr)->prev = (ptr); \
})
```

The final result is exactly the same. The gcc extension might be slightly more readable, but remember that it is not standard C. Gcc's statement-expressions are also better because you can "return" something from a macro. According to the gcc manual,

> The last thing in the compound statement should be an expression followed by a semicolon; the value of this subexpression serves as the value of the entire construct. (If you use some other kind of statement last within the braces, the construct has type void, and thus effectively no value.) 

That's nice! It means that our new `INIT_LIST_HEAD` macro returns the value of the last statement, which is an assignment, and an assignment's value is the left side, so `(ptr)->prev` (which is `ptr` anyway) is "returned" from our macro. Cool!

In the linux kernel, new list elements are either added to the end of the list (using the `list_add_tail()` macro), or to the beginning (using `list_add()`). When retrieving elements from the list, they are always retrieved from the beginning (head towards tail). This implies that you can easily create stacks or queues: for a stack, you always insert new elements using `list_add()`. For a queue, you must add them with `list_add_tail()`. And you can take it into more weird combinations. In the linux kernel, in `kernel/workqueue.c`, you actually see something like this:

```c
list_add(&wq->list, &workqueues);
```

As I said, `list_add()` will insert a new element in the head, so we have a stack of something. What are we adding to the stack? We're adding `wq->list`. So we have a stack of queues. Read that again: a stack in which each element is a FIFO queue. So if you want to insert something in `wq->list`, you must do it with `list_add_tail()`, but if you want to add that list to the stack, you must do it with `list_add()`.

But that's not what this post is about anyway. If you glance through `list.h`, you will come across this macro to traverse a whole generic list:

```c
/**
* list_for_each_entry - iterate over list of given type
* @pos: the type * to use as a loop counter.
* @head: the head for your list.
* @member: the name of the list_struct within the struct.
*/
#define list_for_each_entry(pos, head, member)
  for (pos = list_entry((head)->next, typeof(*pos), member),
     prefetch(pos->member.next);
   &pos->member != (head);
   pos = list_entry(pos->member.next, typeof(*pos), member),
     prefetch(pos->member.next))
```

First, let's see this working with an example. Imagine we have a struct `foo`:

```c
struct foo {
    char *word;
    int value;
    char *a;
    struct list_head list;
};
```

Now, we created some `foo` instances structures that are all connected through the `list` field. We can go through every element of our `foo` list by doing this:

```c
struct foo first_foo = { /* Initialize... */ };
/* Add more elements to tail of list in first_foo... */

struct foo *list_element;
list_for_each_entry(list_element, first_foo->list, list) {
    /* list_element is now a pointer to the current struct foo node of the list 
    Do some work...
    */
}
```

We're practically creating our own language to deal with this generic list thing! How beautiful is that? 

Several things might be bugging you with this macro. For example, what is that `typeof` thing?

It's another extension to C provided by gcc. Again, it's not standard C, only gcc recognizes it. `typeof` works more or less like `sizeof`, it doesn't evaluate the expression it receives as an argument, but it knows what type it is. You could do something like this:

```c
int a;
typeof(a) b;
```

and `b` would be an `int`. `typeof` is just a nice way of determining the type of something. But now we're getting to the core of my post, to the point where it gets more interesting. What does `list_entry` do? Apparently, it returns a structure of type `typeof(*pos)` that corresponds to the structure that includes the current list node. In other words, it picks the structure where our current list node is embedded. 

Huh?! How? Oh my god... this is not possible. Let's look at `list_entry()`...

```c
/**
 * list_entry - get the struct for this entry
 * @ptr:        the &struct list_head pointer.
 * @type:       the type of the struct this is embedded in.
 * @member:     the name of the list_struct within the struct.
 */
#define list_entry(ptr, type, member) \
        container_of(ptr, type, member)

```

Damn it! No magic, it just calls `container_of()`. So let's go after `container_of()`. It's defined in `include/linux/kernel.h`:

```c
/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:        the pointer to the member.
 * @type:       the type of the container struct this is embedded in.
 * @member:     the name of the member within the struct.
 *
 */
#define container_of(ptr, type, member) ({                      \
        const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
        (type *)( (char *)__mptr - offsetof(type,member) );})

```

Sweet mother of god, WHAT IS HAPPENING HERE? I swear, the first time I looked at this, it nearly looked like black magic. First, notice that it is using that Statement-expressions extension provided by gcc that we talked about earlier. Remember that this allows macros to "return" something - the value of the last statement, so `container_of()` is returning:

```c
(type *)( (char *)__mptr - offsetof(type,member) )
```

Ok, that's good for a start. Now let's look at a line at a time:

```c
const typeof( ((type *)0)->member ) *__mptr = (ptr);
```

This line is really just declaring a pointer named `__mptr` and making it point to the `list_head` field. In our previous example, it would expand to:

```c
const struct list_head *__mptr = (ptr);
```

where `ptr` is the `next` field inside `struct list_head`. Remember that `typeof` is similar to `sizeof`; it doesn't evalute the expression inside parentheses, so the null reference in `((type *)0)->member` is not going to crash because the code is not executed, the compiler kind of figures out what type the `member` field inside a structure `type` will be. 

The next line is:

```c
(type *)( (char *)__mptr - offsetof(type,member) );
```

We will look at `offsetof()` in a second, but for now it suffices to know that it returns the byte offset of member `member` in a structure `type`. For example, if we had a structure `bar` defined like this:

```c
struct bar {
    char c;
    int a;
};
```

`offsetof(struct bar, a)` will typically be 4. Why? Because of alignment issues, even though `c` will take 1 byte, integers can only be stored in even addresses, you cannot store a `char c`, and then "a little bit" (3 bytes) of the integer in the remaining 3 bytes of `c`'s address, and then "just one more byte" in the next address. Of course, I'm assuming we're in a 32-bit architecture, in which case c will look like it eats up 4 bytes instead of 1. `offsetof()` will return 4, because `a` is 4 bytes away from the beginning of the structure elements in memory.

So this beautiful `container_of()` macro is grabbing `__mptr` - the pointer to the list element inside `struct foo` in our earlier example - and *subtracting* it its own offset, thus it will now point to the beginning of the structure in memory. It will be pointing to the start of `struct foo` where `__mptr` is embedded.

WOOOOOOOOOOOOOOOOOOOOOOOOOOOW. Black Magic!

The cast to `(char *)` is necessary before subtracting (and yes, the cast has higher precedence than the subtraction), otherwise, pointer arithmetic would scale `offsetof()` by `sizeof(struct list_head)`. `offsetof()` returns a value in bytes, so we shut off pointer arithmetic by making the compiler believe that `__mptr` is pointing to a `char`, and because `sizeof(char)` is 1, no scaling will take place. Tada!

The final cast to `(type *)` just ensures that our final pointer is interpreted as a pointer to `struct foo`, even though it was originally a pointer to a `struct list_head` inside `struct foo`. Hell, we went through all that work just to get to the beginning of the structure, we must show our happiness and achievements by at least casting it to the correct type! Yay!

You might notice that declaring `__mptr` is useless. We could use `ptr` directly instead of declaring `__mptr` to point to `ptr` and use it in the next statement. Why did they do it this way? It turns out that it ensures that you pass consistent values to the macro. The declaration of `__mptr` will issue a warning if for some reason you called the macro with the wrong pointer, or the wrong types. It's a way to give you a meaningful warning when you compile it if you called it with bad arguments.

So now let's look at `offsetof()`. It is defined in `include/linux/stddef.h`:

```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```

Again, this is picking up a null pointer, converting it to a pointer to `TYPE`, accessing `MEMBER` (in our example, it would be accessing the `list` field inside `foo`). Because we're using the address of operator, `&`, there is no null pointer dereferencing. The compiler optimizes it right away and doesn't dereference a null pointer, in fact, it just gives you the address of `MEMBER` in a hypothetical `TYPE` structure that is stored at address 0. The compiler "thinks": well, there's a `TYPE` struct at address 0. What's the address of `MEMBER` in that struct? Of course, it will be 0 plus some constant value. That's exactly what we want.

Note that, again, this is not standard C. Dereferencing a null pointer results in undefined behaviour (although, as I said, one could argue that it is not really dereferenced). This is the danger of looking inside kernel implementations, some of the code is targeted to a specific platform. In my platform, kernel programmers were sure that this `offsetof()` macro is safe to use.

In fact, this is so common that gcc provides yet another extension so that you can use `offsetof()` without implementing it. `offsetof()` is part of the standard library and is defined in `stddef.h`. gcc provides this through the use of `__builtin_offsetof`. As you can see in gcc's manual, `offsetof()` can be easily defined in a portable way (as long as gcc is used - these are extensions, not part of the C standard!) like this:

```c
#define offsetof(type, member)  __builtin_offsetof (type, member)
```

And that's basically all there is to it. 

I will soon be releasing my K&R exercises resolutions. As I mentioned in previous posts, I'm reviewing everything before posting them online. I'm somewhere in the middle of chapter 7 now, so I'm almost done. It's been a hell of a ride. Loved it.
