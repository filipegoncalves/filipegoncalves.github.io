---
layout: post
title: The magic of wrappers - running your own malloc implementation
---

Intercepting library function calls for your advantage

-----

It's been a tough week around here. I finally made it to the last pages of K&R. I am now going through every exercise again, rechecking my solutions, before putting them online. I am preparing a nice repository to host my K&R exercise solutions. I will also include some test cases (input and expected output, and a script to run the tests) for the exercises I found trickier. So far, I've double checked everything up to 1-23. 

Section 8.7 is about developing a storage allocator. The example given could be a possible `malloc()` implementation, but it would not be very efficient because of fragmentation problems. The idea of `malloc()` is to reduce the number of times `sbrk()` (a system call to allocate memory) is invoked. The idea is to maintain free memory blocks in a circular single linked list.

In `malloc()`, each free block has a known size that will most likely reduce every time you ask for a new big chunk of memory, and for reasons that should be obvious, the free blocks list will tend to stay fragmented over time, until eventually a large block of memory is requested by the user program, and there's no block in the free list with the size required, yet, the request could be satisfied if only we could merge some of the tiny little pieces left.

In real life, `malloc()` is smart enough to understand this, and instead of calling `sbrk()` again, it does some house keeping and reorganizes the list so as to bring together fragmented blocks. This is an expensive and long operation, which, along with `sbrk()` calls, is really the bottleneck of `malloc()`.

The `malloc()` example implementation in K&R is based in a first-fit memory management algorithm, which means that the free block list is traversed until a free block with enough size is found (in constrast with a "best-fit" algorithm, which processes the whole list to find the smallest block that can fulfill the request). To avoid shocking fragmentation issues, blocks search starts where the previous search ended; this will prevent the list from getting more and more fragmented in the beginning and untouched in the end.

I was very happy to see a functional, simple `malloc()` implementation in K&R. Even though it's not the real `malloc()`, it's pretty nice to see it, and it's not THAT hard to understand it. In this article, I'll briefly look at the code, show you a funny bug in `printf()`, and talk about my most recent discovery: the magic of function wrappers.

Let's start by looking at `malloc()` code.

```c
typedef long Align; /* for alignment to long boundary */

union header { /* block header: */
  struct {
    union header *ptr; /* next block if on free list */
    unsigned size; /* size of this block */
  } s;
  Align x; /* force alignment of blocks */
};

static Header base;
static Header *freep = NULL;

/* malloc: general-purpose storage allocator */
void *malloc(unsigned nbytes)
{
  Header *p, *prevp;
  Header *morecore(unsigned);
  unsigned nunits;

  nunits = (nbytes+sizeof(Header)-1)/sizeof(Header) + 1;
  if ((prevp = freep) == NULL) { /* no free list yet */
    base.s.ptr = freep = prevp = &base;
    base.s.size = 0;
  }
  for (p = prevp->s.ptr; ; prevp = p, p = p->s.ptr) {
    if (p->s.size >= nunits) { /* big enough */
      if (p->s.size == nunits) /* exactly */
        prevp->s.ptr = p->s.ptr;
      else { /* allocate tail end */
        p->s.size -= nunits;
        p += p->s.size;
        p->s.size = nunits;
      }
      freep = prevp;
      return (void *)(p+1);
    }
    if (p == freep) /* wrapped around free list */
      if ((p = morecore(nunits)) == NULL)
        return NULL; /* none left */
  }
}
```

This is copied from K&R. If you want to see the function `morecore()`, just go to page 188. It does nothing more than asking for more memory using `sbrk()`, filling the `Header` structure with the correct size of the new block, and adding the new block to the free blocks list by calling `free()`. The variable `freep` points to the block before the most recently allocated one, and the search for a new block starts at `freep->s.ptr`. I'm not going to explain every detail of this code, because K&R explains it pretty well, but there's something that may be a little confusing at first:

```c
nunits = (nbytes+sizeof(Header)-1)/sizeof(Header) + 1;
```

First, note that this ought to be read as:

$$
\frac{nbytes+sizeof(Header)-1}{sizeof(Header)} + 1
$$

Some people are tricked into thinking that the final ` + 1` is part of the denominator. Second, remember that this is integer division, meaning that things like \\(\frac{1}{10}\\) evaluate to `0`. Third, and most importantly, notice that what this is really doing is rounding up the integer division. For example, say you wanted to round up the integer division \\(\frac{n}{10}\\), so that \\(\frac{10}{10} = 1\\), \\(\frac{22}{10} = 3\\), and \\(\frac{1}{10} = 1\\). That is, if `n` is not a multiple of 10, then you want \\(\frac{n}{10}+1\\), otherwise, you're fine with \\(\frac{n}{10}\\). If you think for a moment, you will realize that this is equivalent to \\(\frac{n-1}{10}+1\\). Think of it this way: if `n` is a multiple of 10, then \\(\frac{n-1}{10}\\) is equal to \\(\frac{n}{10}+1\\). If, on the other hand, `n` is not a multiple of 10, then \\(\frac{n-1}{10}\\) will fit as many times into 10 as \\(\frac{n}{10}\\), and because you want to round up, you sum one more after that. So, pverall, `n-1` is basically preventing multiples of 10 to round up too high; if you used \\(\frac{n}{10}+1\\) to round up, then \\(\frac{40}{10}\\) would be 5, when the expected result is 4.

Now, instead of 10, `malloc()` is allocating blocks in `sizeof(Header)` units, so we want to round up the integer division of `n` by `sizeof(Header)`, where `n` is the total number of bytes we need. The total number of bytes we need is `nbytes` (requested by the caller), and an extra `sizeof(Header)` bytes to hold the header information for the block, so `n = nbytes+sizeof(Header)`. The denominator is also `sizeof(Header)`, so we end up with the previous expression for `nunits` as our formula to round up this integer division.

The rest of the code is pure pointer manipulation, and I will not be covering it in here, this should be familiar for anyone wanting to understand how `malloc()` works. It can be a little tricky even for someone who is used to deal with pointers because this is a circular single linked list, and if the list only has 1 element, then `next` points to itself. Oh, and this implementation initializes the list with a block pointing to itself with a size of 0. Pretty elegant.

With this in mind, let's try to test K&R's `malloc()` implementation. I grabbed their code and inserted a `main()` with a few tests, nothing fancy: allocate a block of 256 bytes and write an alphabetic letter in each of the 256 positions, and then print all of the 256 chars. Here's how it looked like:

```c
#include <stdio.h>

typedef long Align;

union header {
	struct {
		union header *ptr;
		unsigned size;
	} s;
	Align x;
};

typedef union header Header;

static Header base;
static Header *freep = NULL;

#define NALLOC 1024
#include <unistd.h>
static Header *morecore(unsigned nu) {
	char *cp;
	Header *up;
	void free(void *);
	
	if (nu < NALLOC)
		nu = NALLOC;
	cp = sbrk(nu * sizeof(Header));
	if (cp == (char *) -1)
		return NULL;
	up = (Header *) cp;
	up->s.size = nu;
	free((void *)(up+1));
	return freep;
}

void *malloc(unsigned nbytes) {
	Header *p, *prevp;
	unsigned nunits;
	
	nunits = (nbytes+sizeof(Header)-1)/sizeof(Header) + 1;
	if ((prevp = freep) == NULL) {
		base.s.ptr = freep = prevp = &base;
		base.s.size = 0;
	}
	for (p = prevp->s.ptr; ; prevp= p, p = p->s.ptr) {
		if (p->s.size >= nunits) {
			if (p->s.size == nunits)
				prevp->s.ptr = p->s.ptr;
			else {
				p->s.size -= nunits;
				p += p->s.size;
				p->s.size = nunits;
			}
			freep = prevp;
			return (void *)(p+1);
		}
		if (p == freep)
			if ((p = morecore(nunits)) == NULL)
				return NULL;
	}
}

void free(void *ap) {
	Header *bp, *p;
	
	bp = (Header *)ap - 1;
	for (p = freep; !(bp > p && bp < p->s.ptr); p = p->s.ptr)
		if (p >= p->s.ptr && (bp > p || bp < p->s.ptr))
			break;
	if (bp + bp->s.size == p->s.ptr) {
		bp->s.size += p->s.ptr->s.size;
		bp->s.ptr = p->s.ptr->s.ptr;
	} else
		bp->s.ptr = p->s.ptr;
	if (p + p->s.size == bp) {
		p->s.size += bp->s.size;
		p->s.ptr = bp->s.ptr;
	} else
		p->s.ptr = bp;
	freep = p;
}

int main() {
	char *ptr = malloc(256);
	int i;
	for (i = 0; i < 256; i++)
		ptr[i] = 'a'+(i%26);
	for (i = 0; i < 256; i++)
		printf("%c", ptr[i]);
	printf("\n");
	printf("hello");
	printf("\n");
	return 0;
}
```

I ran this in 2 different machines, one of them was running a native GNU/Linux distribution, and the other was a windows machine with a cygwin environment. When I executed this code, the cygwin machine segfaulted. In the native GNU/Linux machine, this was the output:

```c
abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuv
Segmentation fault
```

This time, I'm not going to ask you where is the bug, because *It's Just Not Fair*. With the help of gdb and valgrind, I found that `printf()` occasionally calls `free()` with a `NULL` pointer. The standard `free()` is expected to handle `NULL` pointers gracefully, but the same doesn't happen with K&R's `free()`. Here's the output after running valgrind:

```
==60596== Invalid read of size 4
==60596==    at 0x4007B8: free (test.c:70)
==60596==    by 0x4E6AE0C: vfprintf (vfprintf.c:2023)
==60596==    by 0x4E75A59: printf (printf.c:35)
==60596==    by 0x40090D: main (test.c:91)
==60596==  Address 0xfffffffffffffff8 is not stack'd, malloc'd or (recently) free'd
```

Notice the last message about address `0xfffffffffffffff8`. In my machine, `sizeof(Header *)` is 8, and because `ap` is `NULL`, `ap-1` will go back 8 bytes, thus arriving at the address -8, which is represented as `0xfffffffffffffff8` in two's complement.

Let's stop for a moment and think about what's happening: it looks like our `malloc()` implementation is being used, even though the standard library also defines a `malloc()` and `free()` implementation. So, which one will our program be using? Looking at gcc documentation, you can read this:

> GCC includes built-in versions of many of the functions in the standard C library. The versions prefixed with _builtin will always be treated as having the same meaning as the C library function even if you specify the -fno-builtin option. (see C Dialect Options) Many of these functions are only optimized in certain cases; if they are not optimized in a particular case, a call to the library function will be emitted.


It turns out that if you compile this with `-Wall`, as all good programmers do, you'll get this:

```
test.c:36: warning: conflicting types for built-in function 'malloc'
```

That stinks. And it happens because K&R's `malloc()` prototype is `void *malloc(unsigned)`, rather than `void *malloc(size_t)`. If you change it to `size_t` instead, the warning is gone. 

But if you use `-fno-builtin` option and leave `malloc()` as receiving `unsigned`, the warning is gone as well. In-ter-es-ting. This means that gcc uses a builtin `malloc()` implementation.

Either way, which implementation will be used? Yours, or the standard library's? There's a rule that says that all identifiers declared in the C standard header files are reserved for the implementation, so you may not declare a function called `malloc()` because a standard header file already has a function of that name. Now, the funny part is that the compiler is not actually supposed to warn you about this mistake, because the C standard doesn't impose this rule as a constraint, and compilers only need to produce error messages for constraints. So the short answer is: no one knows what will happen. Maybe it will use yours, maybe it won't. God knows. Have you ever heard about interpositioning? As Peter mentions on his book, *Expert C Programming - Deep C Secrets*,

> The problem of too much scope interacts with another common C convention, that of interpositioning. Interpositioning is the practice of supplanting a library function by a user-written function of the same name. Many C programmers are completely unaware of this feature, so it is described in the chapter on linking. For now, just make the mental note: "I should learn more about that."

And you definitely should. So do I.

When struggling with this problem and trying to debug `fprintf()` at the same time, I came across a a very interesting and useful concept provided by `ld`, the typical GNU/Linux linker used in the last stage of the compilation process. Remember, the linker is held responsible for linking your compiled object file with other required object files. For example, when you use `printf()`, you must include `stdio.h`, but `stdio.h` only declares the function prototype; the actual function body must be provided somewhere - that's what the linker does, it "joins" your program together with other functions needed that are defined elsewhere (library files, or even other files created by you, in which case you must explicitly include them when you call gcc).

Imagine that, for debugging purposes, you'd like to have some easy-to-use system that allowed you to quickly swap between stdlib's `malloc()` and your implementation. Ideally, you wanted something as simple as compiling `foo.c` with `-DUSE_MY_MALLOC` to use your `malloc()` implementation. Of course, your `malloc()` implementation is a cheap copy of the real `malloc()`, but the function names are the same, `malloc()` and `free()`, and you want to make sure there are no naming conflicts and that every call to `malloc()` will be forwarded to your `malloc()` implementation.

Wrappers to the rescue! Wrappers can do this for us. In gcc, there's an option, `-Wl`, that you can use to pass options to the linker. In a typical linux environment, the linker used by gcc will be `ld`, which accepts an option, `-wrap`, followed by a function name you want to wrap. For example, say you wanted to write a wrapper for `foo()`. You'd have to compile your program using `-Wl,-wrap,foo`. How do wrappers work? Well, now you can define a function called `__wrap_foo()`, and you can call the original `foo()` by invoking `__real_foo()`. Keep in mind that the wrapper signature must be the same as the real signature. Also, as a side note, there have been some problems with this approach reported on MAC OS X platforms.

Let's play a little bit with what we just learned. We can now implement our `malloc()` library like so:

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#define NALLOC 1024

void *my_malloc(size_t);
void my_free(void *);
void *__real_malloc(size_t);
void __real_free(void *);

void *__wrap_malloc(size_t nbytes) {
  #ifdef USE_MY_MALLOC
    return my_malloc(nbytes);
  #else
    return __real_malloc(nbytes);
  #endif
}

void __wrap_free(void *ptr) {
  #ifdef USE_MY_MALLOC
    my_free(ptr);
  #else
    __real_free(ptr);
  #endif
}

typedef long Align;

union header {
  struct {
    union header *ptr;
    unsigned size;
  } s;
  Align x;
};

typedef union header Header;

static Header base;
static Header *freep = NULL;

static Header *morecore(unsigned nu) {
  char *cp;
  Header *up;
  
  if (nu < NALLOC)
    nu = NALLOC;
  cp = sbrk(nu * sizeof(Header));
  if (cp == (char *) -1)
    return NULL;
  up = (Header *) cp;
  up->s.size = nu;
  my_free((void *)(up+1));
  return freep;
}

void *my_malloc(size_t nbytes) {
  Header *p, *prevp;
  size_t nunits;
  
  nunits = (nbytes+sizeof(Header)-1)/sizeof(Header) + 1;
  if ((prevp = freep) == NULL) {
    base.s.ptr = freep = prevp = &base;
    base.s.size = 0;
  }
  for (p = prevp->s.ptr; ; prevp= p, p = p->s.ptr) {
    if (p->s.size >= nunits) {
      if (p->s.size == nunits)
        prevp->s.ptr = p->s.ptr;
      else {
        p->s.size -= nunits;
        p += p->s.size;
        p->s.size = nunits;
      }
      freep = prevp;
      return (void *)(p+1);
    }
    if (p == freep)
      if ((p = morecore(nunits)) == NULL)
        return NULL;
  }
}

void my_free(void *ap) {
  Header *bp, *p;
  
  bp = (Header *)ap - 1;
  for (p = freep; !(bp > p && bp < p->s.ptr); p = p->s.ptr)
    if (p >= p->s.ptr && (bp > p || bp < p->s.ptr))
      break;
  if (bp + bp->s.size == p->s.ptr) {
    bp->s.size += p->s.ptr->s.size;
    bp->s.ptr = p->s.ptr->s.ptr;
  } else
    bp->s.ptr = p->s.ptr;
  if (p + p->s.size == bp) {
    p->s.size += bp->s.size;
    p->s.ptr = bp->s.ptr;
  } else
    p->s.ptr = bp;
  freep = p;
}

int main() {
  /* Place your tests here... */
  return 0;
}
```

Let's assume this is saved in a file called `malloc_tests.c`. Now, we can either compile it to use our `malloc()` implementation:

```
gcc -o malloc_tests -Wall -Wl,--wrap,free -Wl,--wrap,malloc malloc_tests.c -DUSE_MY_MALLOC
```

Or we can choose to use stdlib's `malloc()` by removing `-DUSE_MY_MALLOC`:

```
gcc -o malloc_tests -Wall -Wl,--wrap,free -Wl,--wrap,malloc malloc_tests.c
```

This is very powerful, and extremely elegant and beautiful. You can use this to wrap any library function and test your implementation. It's very, very cute. I LOVE it. Or you can log every call to `malloc()`. It's a whole new, fascinating world where the sky is the limit. But keep in mind that this has some limitations. It doesn't seem to work very well on Cygwin. I tried it, and nothing happened. I mean, literally nothing. I executed the program, and nothing happened - not even a segfault - total, absolute silence. And I read on the web that it doesn't work properly on OS X as well.

By the way, now that I finished reading K&R, I turned my attention to yet another C book: *Expert C Programming - Deep C Secrets*, by Peter Van Der Linden. I only had time to read the first pages before the beginning of chapter 1, and I'm already inlove with the book. His sense of humor is hilarious, and I really feel that this book has lots of things to teach. It's a pandora box.

Fun fact: Did you know that `tunefs` manpage (linux command to change filesystem parameters) says *You can tune a file system, but you can't tune a fish.*, and that its word-processor source has a comment threatening anyone who removes it that says:

> Take this out and a UNIX Demon will dog your steps from now until the time_t's wrap around.

Mhuahuahuahuahuahuahu!

Go and buy the book! Totally worth it!
