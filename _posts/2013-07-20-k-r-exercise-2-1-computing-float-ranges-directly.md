---
layout: post
title: K&R exercise 2-1 - Computing float ranges directly
---

Why K&R exercise 2-1 is not as easy as it looks

-----

If you've been following my latest posts, you are probably aware that I've gone through every K&R exercise and am now rechecking all of my solutions before putting them online in here. Uppon reaching exercise 2-1 (page 36), I remembered the long, challenging moments that provided me great learning experiences. Here's the exercise:

> Write a program to determine the ranges of char, short, int, and long variables, both signed and unsigned, by printing appropriate values from standard headers and by direct computation. Harder if you compute them: determine the ranges of the various floating-point types.

Challenge accepted! 

Today, I want to share this with you. I want to go down again. Down to the byte level. No, sorry, *down to the bit level*. We are going to directly compute the ranges of `float` and `double`. Before moving on, you might want to read, and I mean *really* read and understand, [my post about how floating point is represented]({% post_url 2013-06-03-floating-points-101 %}). If you don't read it, or, for that matter, if you don't know how floats are represented according to IEEE 754 standard, then don't bother reading the rest of this article.

Before moving on to `float` types, let's look at `int`. How can we compute `INT_MIN` and `INT_MAX`? The first bit is the sign bit, so `INT_MAX` is just the same as setting every bit to 1, except for the most significant bit. Assuming we're on 2's complement representation, we also know that `INT_MIN` is `-INT_MAX-1`. Thus, we can calculate these 2 values in a portably like this:

```c
#define INT_MAX ((unsigned int) ~0 >> 1)
#define INT_MIN (-((unsigned int) ~0 >> 1)-1)
```

The logic behind this is that because we know that `int` is using a 2's complement notation, we somehow induce a bit pattern into our code that corresponds to `INT_MAX` and `INT_MIN`. Note that the "set everything to 1 except the first bit" rule is valid for any machine that uses 2's complement representation, no matter how many bits an `int` is.

Let's focus our attention on `float` and try to do exactly the same thing. Once we figure how to do it with `float`, it will be pretty easy to jump to `double`. Doubles just use more bits, and that's basically it. Remember, we need to figure out the bit patterns we want to induce in order to find `FLT_MAX` and `FLT_MIN`. 

We will see how to do this without assuming a 32-bit representation, but for now, recall that a 32-bit floating point number in standard IEEE 754 representation is composed by a sign bit, followed by 8 exponent bits, and then 23 mantissa bits. What are the bit patterns that correspond to the minimum and maximum values that can be represented? If you're familiar with the representation, you know that we can have 2 minimums, depending on whether we care if some precision is lost. With only 1 precision bit, floats can be as small as:

| S   | E   | E   | E   | E   | E   | E   | E   | E   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 1   |

Or, if we don't want to give up on any mantissa bits, it can be:

| S   | E   | E   | E   | E   | E   | E   | E   | E   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   |

Note that the only bit set is the least significant bit in the exponent. I think we all agree here: in the first approach, if we're willing to lose precision bits, then the smallest value we can ever represent is given by merely setting the least significant bit to 1 in the mantissa. In the second approach, if mantissa bits are to be left untouched, then the least significant bit that we can touch (the first exponent bit after the most significant mantissa bit) must be set to 1.

I'm sorry, I forgot that I should clear something before I spook you: when I say "smallest" value, I really want to say "smallest positive value". Sure, floats can be negative, but once we have the smallest positive value that we can represent, it's just a matter of setting the first bit to 1 to make it negative.

Anyway, I think it's pretty clear that these 2 bit configurations represent very small values (the smallest) that we can represent in a `float`, depending on whether we want to lose mantissa bits.

What about the maximum value? That's just the opposite. The maximum value is obtained by setting every bit to 1, except the sign bit and the least significant bit of the exponent. Why? Because the standard represents `NaN` (Not A Number) using the highly sloppy, obfuscated rule that every bit on the exponent is set to 1 and there's at least one bit set in the mantissa bits. So, if we set everything to 1 except the sign bit, we're representing `NaN`, the result of an invalid math operation, like division by 0, rather than `FLT_MAX`. Well, if we can't use every bit in the exponent, which one will we set to 0? The one that causes the least impact, of course: the least significant. So, `FLT_MAX` is represented by:

| S   | E   | E   | E   | E   | E   | E   | E   | E   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 0   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   | 1   |

We are now ready to infer the general rule for any `n` bit representation. Assuming that a float is represented by `N` bits, of which the least significant `M` bits are the mantissa bits, then we necessarily have `N-M-1` exponent bits, and the most significant bit for sign representation. For example, with 32 bits, we have `N = 32`, `M = 23`, and `32-23-1 = 8` exponent bits, and 1 bit for sign.

Generically, we can then infer that:

* The minimum value with only 1 precision bit is always represented by setting the least significant bit to 1
* The minimum value without precision loss is represented by `1 << M`
* The maximum value is represented by `~(1 << M) & (~0 >> 1)`

You might have gotten a little confused with the last one. The first part, `~(1 << M)`, is just the negation of the minimum value without precision loss. `1 << M` only sets the least significant bit of the exponent to 1, so `~(1 << M)` sets everyting to 1 except the least significant bit of the exponent - just what we want. However, it leaves the sign bit set to 1, and we need to set it off. That's what the bitwise AND with `(~0 >> 1)` is doing: it yields a 0 with `N-1` 1's after it.

With this in mind, we theoretically have the necessary knowledge to start coding this away. This was my first attempt:

```c
#include <stdio.h>

void compute_ranges(void);

int main() {
	compute_ranges();
	return 0;
}

unsigned int get_mantissa(void);

void compute_ranges(void) {
	float lowest;
	/* Tricking the compiler to let us mess with floats bits... */
	unsigned int *lowest_ptr = (unsigned int *) &lowest;
	*lowest_ptr = 1;
	unsigned int mantissa_bits = get_mantissa();
	float lowest_precise;
	unsigned int *ptr = (unsigned int *) &lowest_precise;
	*ptr = 1;
	*ptr <<= mantissa_bits;
	float max_val;
	ptr = (unsigned int *) &max_val;
	*ptr = 1;
	*ptr <<= mantissa_bits;
	*ptr = ~(*ptr);
	*ptr &= ((unsigned int) ~0)>>1;
	printf("FLOAT\n\tLowest possible number with precision loss (only 1 precision bit): %g\n", lowest);
	printf("\tWithout precision loss: %g TO %g\n", (double) lowest_precise, (double) max_val);
}

/* 
Very nice!!! 
Credit goes to http://stackoverflow.com/questions/502022/how-to-find-mantissa-length-on-a-particular-machine 
*/
unsigned int get_mantissa(void) {
	float a = 1.0;
	float b = 0.5;
	float c = a+b;
	unsigned int bits = 0;
	while (c != a) {
		bits++;
		b /= 2;
		c = a+b;
	}
	return bits;
}
```

We have a couple of points worth mentioning here. The first concern is that C doesn't allow us to do bitwise operations on `float`. It's undefined. You can't do it. Because of that, I was forced to cheat on gcc, and make it think that we're not doing bitwise operations on a `float`, but rather on `unsigned int`. Why unsigned? Because right shifts can give you trouble with sign extension when performed on `signed int`. With unsigned types, this never happens - there's no sign bit. Then we have this `get_mantissa()` function. It's beautiful. If you read my article [Floating Points 101]({% post_url 2013-06-03-floating-points-101 %}), you will understand why it works. It's a clever way to compute mantissa bits that I found on stackoverflow.

Anyway, it's not a very elegant solution. It's got casts everywhere, it's not portable, and it's not reusable at all. What about when we want to repeat this for `double`? We'd have to copy-paste the code and change `float` to `double`.

You know what? This solution stinks. It probably works in your machine; note the word *probably*. It's got a serious problem. It assumes that `sizeof(unsigned int) == sizeof(float)`.

When we cast a `float` address to an `unsigned int` address, if `sizeof(float) != sizeof(unsigned int)`, then this:

```c
*ptr = 1;
```

As well as any other similar code, does not have the desired effect. Let's see what's happening here. Assuming that `sizeof(float) != sizeof(unsigned int)`, then we know that either `sizeof(float) < sizeof(unsigned int)`, or `sizeof(float) > sizeof(unsigned int)`. For the first case, the statement above causes your program to write in a memory address that was not reserved, and will probably crash with an ugly segfault (or maybe not). This happens because you force the compiler to think that the size of the thing pointed by that memory address is bigger than it actually is. For example, if `sizeof(float) == 4` and `sizeof(unsigned int) == 8`, then the statement `*ptr = 1` would set the least significant bit to 1, but the least significant bit of what would be an `unsigned int` is 8 bytes away, not 4, and you only reserved space for a `float`, which takes 4 bytes. Well, now that I really think about it, the assignment *can* work, depending on whether your machine is little endian. If your machine is little endian, then the least significant bit of the first byte is set to 1, which is not problematic, and when you read the value of `lowest`, it will work. But still, `*ptr = 1;` would write 7 bytes with `0`, and thus would go 3 bytes ahead the place where it should have stopped.

Similarly, the same is true for the case `sizeof(float) > sizeof(unsigned int)`.

With this, notice that the issue of adapting this code to make the same for `double` raises a red flag: `sizeof(float)` is *definitely* lower than `sizeof(double)`, yet, the code blindly assumes that `sizeof(float) == sizeof(unsigned int)`, and if you change everything to `double`, it will assume that `sizeof(double) == sizeof(unsigned int)`. Because both can't be true at the same time, something has to go wrong. You can be lucky and both versions of the code with `float` and `double` work for you, but that's just a sheera mount of luck, you probably had the neighboring memory positions set to 0, AND they happened to be in your program addressable space, AND they happened to be free. That's a lot of conditions, but it can surely happen.

So what can we do? Imagination is the limit, and I've come up with a rather obfuscated solution, but you will understand it if you pay attention. Stay with me, this is getting really interesting! The first thing to notice is that we can represent a `float` as an array of `char`. So, something like this:

```c
char my_float[sizeof(float)];
```

Declares an array with exactly the required blocks to hold a `float`. Each position in the array is a "piece" of our `float`, a logical block unit. And guess what, because it's a `char` array, we are free to make any bit operations. We then have to create a pointer to `float`, and make it point to `my_float`, after casting `my_float` to `float *`. This is safe because `my_float` is guaranteed to be exactly the size that a pointer to `float` is expecting it to be. Genius.

What's the equivalent of `*ptr = 1;` in our float array representation?

```c
my_float[sizeof(float)-1] = 1;
```

If that's what came to your mind, don't worry, that was my first idea as well. Nice try! But it's wrong. Remember that we will be doing something like:

```c
float *f = (float *) my_float;
```

If the machine is little endian, then you just created a `float` where the only active bit is the least significant bit of the most significant byte (because the most significant byte is last in a little endian representation). But seriously, we WANT the least significant byte to be 1, no matter what. 

So here's what I did: I created an abstraction layer to deal with endianness problems. This abstraction introduces the concept of *base*, and it is used to calculate the real position in the array where we want to write. `base` is just an array index saying where the "real" first block of the float is. It's a way for us to abstract the fact that the first block of a `float` can be `my_float[0]` or `my_float[sizeof(float)-1]`, depending on the endianness. So, if the underlying architecture is big-endian, then `base` is 0, if it's little-endian, then `base` is `sizeof(float)-1`. Remember: `base` is basically an array index pointing to the most significant "block", aka byte, of our float representation.

Let's assume there's a function, `getpos()`, that implements this abstraction. It receives the base value, and a position `p` where we want to write. So, for example, the statement

```c
my_float[getpos(base,sizeof(float)-1)] = 1;
```

Will write the value `1` in the last block of the `float`, whatever block that is. `getpos()` must be able to figure out what the index `sizeof(float)-1` corresponds to in real life. So, let's start by writing `getpos()`. It will be a simple macro. Let's look at the previous example with more attention. When `p = sizeof(float)-1`, the real index is `sizeof(float)-1` as long as our machine is big-endian. If the machine is little-endian, then that means that when the user asks to write the "last" position of the float, `sizeof(float)-1`, the real position to be written is the first one (because the first block is the least significant byte in a little-endian machine). So, if `p = sizeof(float)-1`, then the real index is supposed to be `0`. With similar examples, it is easy to see that the general rule is that if the machine is big-endian, then no change to the index is required, if it's not, then the real index is given by `base-p`. Thus, `getpos()` is as simple as:

```c
#define getpos(base, p) ((base) != 0 ? ((base)-(p)) : (p))
```

Cool! Let's stop for a bit and organize our thoughts and see what we have so far. The current state is:

* We know what bit patterns we want to induce in a float type
* We have a nice, proper way of doing bit manipulation in float types
* We have a function that calculates how many mantissa bits are available
* We have a macro to write to an array position without having to worry about endianness issues

And this is what would be really, really nice to have:

* A macro to set a bit to `1` in position `x` in index `p`, where `x` can be something between `0` and `CHAR_BIT-1` - the number of bits in a char.
* A similar macro to unset a bit in position `x`
* Some way of reusing all this code to do the same job for doubles, since the only difference is that there are more exponent and mantissa bits

If that would be really, really nice to have, then let's do it. Once we develop a solution for the first 2 items, we will be capable of computing `float` ranges by ourselves. Starting out with `setbit()`, we're going to assume that it receives the array, `base`, `p`, and a position `x` that varies between `0` and `CHAR_BIT-1`, where `0` denotes the least significant bit of whatever is stored in position `p`. For example, assuming that `CHAR_BIT` is 8, then `setbit(arr, base, 0, 7)` sets the most significant bit of the most significant byte of our array-represented `float` to `1`. In other words, it makes it a negative `float`.

This way, we can easily define `setbit()`: it merely calls `getpos()` to get the correct array index and ORs whatever is in there with `1 << x`. You are allowed to think it out loud: if you grab a `1` and shift it left `x` positions, you'll get a `1` followed by `x` zeros. Because of the OR semantics, ORing that with whatever is stored in position `p` will make nothing but force the `x`-th bit (remember that `x` starts at `0`) to be `1`. Turning this into code yields:

```c
#define setbit(arr,b,p,x) ((arr)[getpos(b,p)] |= (unsigned char) 1 << (x))
```

The cast to `unsigned char` makes sure there are no problems with sign extension. What about `unsetbit`? It's the opposite: instead of ORing, we want to AND it, and instead of using a `1` followed by `x` zeros, we want exactly the contrary: everything set to `1` except the bit we want to turn off. Again, because of bitwise AND properties, ANDing whatever is in position `p` with a bunch of `1` where the only `0` is in position `x` will make nothing but turn off the `x`-th bit. So, `unsetbit()` is just:

```c
#define unsetbit(arr,b,p,x) ((arr)[getpos(b,p)] &= ~((unsigned char) 1 << (x)))
```

Like I promised, with these macros, we are now able to compute `float` or `double` ranges by ourselves. The generic procedure is:

* Set the rightmost bit of the last "block" in the array to 1 - this will give the minimum number representable with only 1 precision bit.
* Set the least significant bit of the exponent to 1 - this will give the minimum number representable without any precision loss.
* Compute the unary complement of the previous quantity and unset the sign bit (the leftmost bit of the first position in the array) - this yields the maximum float value representable.

Let's focus on one thing at a time. Step 1 is very, very reasy. Assuming `my_float` is initialized to 0, all we have to do is:

```c
my_float[getpos(base,sizeof(float)-1)] = 1;
```

Ok. That was fast. For step 2, we need to figure out the array position of our first exponent bit. Because bit counting starts at 0, the first (rightmost) exponent bit will be in position `sizeof(float)-M/CHAR_BIT-1`, where `M` is the number of mantissa bits. What about the index inside that position? It's `M % CHAR_BIT`. `M % CHAR_BIT` tells us how many mantissa bits were "left behind" before filling another array position, and because we start counting bits at 0, the first exponent bit's index inside position `sizeof(float)-M/CHAR_BIT-1` is just `M % CHAR_BIT`.

If you have difficulties coming up with the above formulas, don't worry, it's normal. My way of doing it is to kind of think about it and come up with an intuitive formula for what I need. Then I try it with some sample numbers to see if it works. It's important that the formula is tested with normal cases, but also with boundary conditions, like "what if I had 0 mantissa bits? What if it was 1? What if it was 31?". 

For example, my first thought for the position formula was `M/CHAR_BIT`. Then I realized that what I really wanted was `sizeof(float)-M/CHAR_BIT` because `M/CHAR_BIT` gives me the position counting from the end of the array. Finally, I calculated it with some test values and noticed that it was off by 1, hence the -1. Then, finally, I looked at the formula and understood why the -1 was needed (we start counting at 0), and it all made sense (it must make sense, you can't really come up with a formula that you can't explain but "it just works").

These formulas are directly translated to:

```c
setbit(my_float,base,sizeof(float)-M/CHAR_BIT-1,M%CHAR_BIT);
```

Step 3 is pretty damn easy. Like I mentioned earlier, all we have to do is loop through every element in the array and set it to its own unary complement. And last, but not least, we unset the sign bit, which is just bit number `CHAR_BIT-1` in position 0:

```c
int i;
for (i = 0; i < sizeof(float); i++)
   my_float[i] = ~my_float[i];
unsetbit(my_float,base,0,CHAR_BIT-1);
```

Putting this all together, we get:

```c
#include <stdio.h>
#include <float.h>

int is_bigendian();
unsigned int get_mantissa();
#define getpos(base, p) ((base) != 0 ? ((base)-(p)) : (p))
#define setbit(arr,b,p,x) ((arr)[getpos(b,p)] |= (unsigned char) 1 << (x))
#define unsetbit(arr,b,p,x) ((arr)[getpos(b,p)] &= ~((unsigned char) 1 << (x)))

void compute_ranges(void);

int main() {
	compute_ranges();
	return 0;
}

void compute_ranges(void) {
	int be = is_bigendian(), base = 0, i;
	unsigned int m = get_mantissa();
	unsigned char my_float[sizeof(float)] = { 0 };
	float *f = (float *) my_float;
	
	/* Define the base value */
	if (!be)
		base = sizeof(float)-1;
	
	my_float[getpos(base, sizeof(float)-1)] = 1;
	printf("FLOAT\n\tLowest possible number with precision loss (only 1 precision bit): %g\n", *f);
	
	/* Reset */
	my_float[getpos(base, sizeof(float)-1)] = 0;
	
	setbit(my_float,base,sizeof(float)-m/CHAR_BIT-1,m%CHAR_BIT);
	printf("\tWithout precision loss: %g", *f);
	
	/* Negate everything to get max. value... */
	for (i = 0; i < sizeof(float); i++)
		my_float[i] = ~my_float[i];
	/* ... and turn sign bit off */
	unsetbit(my_float,base,0,CHAR_BIT-1);
	
	printf(" TO %g\n", *f);
}

/* 
Very nice!!! 
Credit goes to http://stackoverflow.com/questions/502022/how-to-find-mantissa-length-on-a-particular-machine 
*/
unsigned int get_mantissa() {
	float a = 1.0;
	float b = 0.5;
	float c = a+b;
	unsigned int bits = 0;
	while (c != a) {
		bits++;
		b /= 2;
		c = a+b;
	}
	return bits;
}

int is_bigendian() {
	unsigned char i[sizeof(int)] = { 0 };
	i[sizeof(int)-1] = 1;
	int *p = (int *) i;
	return *p == 1;
}
```

This printed:

```
FLOAT
        Lowest possible number with precision loss (only 1 precision bit): 1.4013e-45
        Without precision loss: 1.17549e-38 TO 3.40282e+38
```

Which matches the ranges defined in `float.h` for my machine. I needed to include `limits.h` because that's where `CHAR_BIT` is defined. We could easily compute it ourselves as well, with something like:

```c
int getcharbit(void) {
  int bits;
  unsigned char c = ~0;
  for (bits = 0; c; bits++, c >>= 1);
  return bits;
}
```

This code literally sets every bit in a `char` to 1, and counts all the bits that are 1, removing them once they are counted, untill no more "1" bits exist.

Hurrah! We have code that computes `float` ranges by itself. What now? Now it is time for step 3: making it reusable for `double`. Notice that *everything* is exactly the same, we just have to replace `float` with `double`. We surely can't do that in runtime. What is the perfect tool that fits this "change this type to that" request? Macros, of course!

So, again, we can define a macro that expands into the code inside `compute_ranges()` function. It just needs to receive the type as argument. Here's how it looks like:

```c
#define compute_ranges(type) { \
	int be = is_bigendian(), base = 0, i; \
	unsigned int m = 0; \
	unsigned char my_type[sizeof(type)] = { 0 }; \
	type *f = (type *) my_type; \
	if (!be) \
		base = sizeof(type)-1; \
	type a = 1.0; \
	type b = 0.5; \
	type c = a+b; \
	while (c != a) { \
		m++; \
		b /= 2; \
		c = a+b; \
	} \
	my_type[getpos(base, sizeof(type)-1)] = 1; \
	printf( #type "\n\tLowest possible number with precision loss (only 1 precision bit): %g\n", *f); \
	my_type[getpos(base, sizeof(type)-1)] = 0; \
	setbit(my_type,base,sizeof(type)-m/CHAR_BIT-1,m%CHAR_BIT); \
	printf("\tWithout precision loss: %g", *f); \
	for (i = 0; i < sizeof(type); i++) \
		my_type[i] = ~my_type[i]; \
	unsetbit(my_type,base,0,CHAR_BIT-1); \
	printf(" TO %g\n", *f); }
```

As you can see, I was forced to bring the code for `get_mantissa()` inside our macro block, because that depends on the type as well. I can't just define `get_mantissa()` as a macro, because macros can't return values.

Now, choosing between `float` and `double` is as simple as changing `type` when we call `compute_ranges()`. Here's the final version, calling `compute_ranges()` for `float` and then for `double`:

```c
#include <stdio.h>
#include <limits.h>

#define getpos(base, p) ((base) != 0 ? ((base)-(p)) : (p))
#define setbit(arr,b,p,x) ((arr)[getpos(b,p)] |= (unsigned char) 1 << (x))
#define unsetbit(arr,b,p,x) ((arr)[getpos(b,p)] &= ~((unsigned char) 1 << (x)))
#define compute_ranges(type) { \
	int be = is_bigendian(), base = 0, i; \
	unsigned int m = 0; \
	unsigned char my_type[sizeof(type)] = { 0 }; \
	type *f = (type *) my_type; \
	if (!be) \
		base = sizeof(type)-1; \
	type a = 1.0; \
	type b = 0.5; \
	type c = a+b; \
	while (c != a) { \
		m++; \
		b /= 2; \
		c = a+b; \
	} \
	my_type[getpos(base, sizeof(type)-1)] = 1; \
	printf( #type "\n\tLowest possible number with precision loss (only 1 precision bit): %g\n", *f); \
	my_type[getpos(base, sizeof(type)-1)] = 0; \
	setbit(my_type,base,sizeof(type)-m/CHAR_BIT-1,m%CHAR_BIT); \
	printf("\tWithout precision loss: %g", *f); \
	for (i = 0; i < sizeof(type); i++) \
		my_type[i] = ~my_type[i]; \
	unsetbit(my_type,base,0,CHAR_BIT-1); \
	printf(" TO %g\n", *f); }

int is_bigendian();

int main() {
	compute_ranges(float);
	compute_ranges(double);
	return 0;
}

int is_bigendian() {
	unsigned char i[sizeof(int)] = { 0 };
	i[sizeof(int)-1] = 1;
	int *p = (int *) i;
	return *p == 1;
}
```

This printed:

```
float
        Lowest possible number with precision loss (only 1 precision bit): 1.4013e-45
        Without precision loss: 1.17549e-38 TO 3.40282e+38
double
        Lowest possible number with precision loss (only 1 precision bit): 4.94066e-324
        Without precision loss: 2.22507e-308 TO 1.79769e+308
```

Isn't that beautiful? Our code is nothing but macros! Macros can be really useful. Just for some fun, we can take a quick look at how the code looks like after the preprocessor finishes its job. `compute_ranges(float)` expands into:

```c
 { int be = is_bigendian(), base = 0, i; unsigned int m = 0; unsigned char my_type[sizeof(float)] = { 0 }; float *f = (float *) my_type; if (!be) base = sizeof(float)-1; float a = 1.0; float b = 0.5; float c = a+b; while (c != a) { m++; b /= 2; c = a+b; } my_type[((base) != 0 ? ((base)-(sizeof(float)-1)) : (sizeof(float)-1))] = 1; printf( "float" "\n\tLowest possible number with precision loss (only 1 precision bit): %g\n", *f); my_type[((base) != 0 ? ((base)-(sizeof(float)-1)) : (sizeof(float)-1))] = 0; ((my_type)[((base) != 0 ? ((base)-(sizeof(float)-m/8 -1)) : (sizeof(float)-m/8 -1))] |= (unsigned char) 1 << (m%8)); printf("\tWithout precision loss: %g", *f); for (i = 0; i < sizeof(float); i++) my_type[i] = ~my_type[i]; ((my_type)[((base) != 0 ? ((base)-(0)) : (0))] &= ~((unsigned char) 1 << (8 -1))); printf(" TO %g\n", *f); }
```

And our whole program, after macro expansion is applied, will look like:

```c
int is_bigendian();

int main() {
 { int be = is_bigendian(), base = 0, i; unsigned int m = 0; unsigned char my_type[sizeof(float)] = { 0 }; float *f = (float *) my_type; if (!be) base = sizeof(float)-1; float a = 1.0; float b = 0.5; float c = a+b; while (c != a) { m++; b /= 2; c = a+b; } my_type[((base) != 0 ? ((base)-(sizeof(float)-1)) : (sizeof(float)-1))] = 1; printf( "float" "\n\tLowest possible number with precision loss (only 1 precision bit): %g\n", *f); my_type[((base) != 0 ? ((base)-(sizeof(float)-1)) : (sizeof(float)-1))] = 0; ((my_type)[((base) != 0 ? ((base)-(sizeof(float)-m/8 -1)) : (sizeof(float)-m/8 -1))] |= (unsigned char) 1 << (m%8)); printf("\tWithout precision loss: %g", *f); for (i = 0; i < sizeof(float); i++) my_type[i] = ~my_type[i]; ((my_type)[((base) != 0 ? ((base)-(0)) : (0))] &= ~((unsigned char) 1 << (8 -1))); printf(" TO %g\n", *f); };
 { int be = is_bigendian(), base = 0, i; unsigned int m = 0; unsigned char my_type[sizeof(double)] = { 0 }; double *f = (double *) my_type; if (!be) base = sizeof(double)-1; double a = 1.0; double b = 0.5; double c = a+b; while (c != a) { m++; b /= 2; c = a+b; } my_type[((base) != 0 ? ((base)-(sizeof(double)-1)) : (sizeof(double)-1))] = 1; printf( "double" "\n\tLowest possible number with precision loss (only 1 precision bit): %g\n", *f); my_type[((base) != 0 ? ((base)-(sizeof(double)-1)) : (sizeof(double)-1))] = 0; ((my_type)[((base) != 0 ? ((base)-(sizeof(double)-m/8 -1)) : (sizeof(double)-m/8 -1))] |= (unsigned char) 1 << (m%8)); printf("\tWithout precision loss: %g", *f); for (i = 0; i < sizeof(double); i++) my_type[i] = ~my_type[i]; ((my_type)[((base) != 0 ? ((base)-(0)) : (0))] &= ~((unsigned char) 1 << (8 -1))); printf(" TO %g\n", *f); };
 return 0;
}

int is_bigendian() {
 unsigned char i[sizeof(int)] = { 0 };
 i[sizeof(int)-1] = 1;
 int *p = (int *) i;
 return *p == 1;
}
```

Quite some good, readable code! Muaahuahuahauhaua.

Very nice exercise. I hope you enjoyed it as much as I did!
