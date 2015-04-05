---
layout: post
title: Determining your machine's endianness
---

A cute and quick hack to see endianness in action

-----

Today I want to go through a small topic on machine endianness.

How can one know if a machine is little endian or big endian? It may not be very obvious at first, but it can be quickly checked with a few lines of C code.

Now, before getting into that, I have a confession to make: in the past, I always kept forgetting the differences between little endian and big endian. I knew, for sure, that one of them interpreted the first byte as being the most significant, and the other interpreted it as being the least significant. Which one is which? I always got confused. But eventually I found a way to never forget about it anymore.

The trick goes like this: endianness always talks about the first byte. Big endian means that the first byte is the biggest, and little endian means that the first byte is the smallest. Ok, I know that bytes are all the same size, but by "biggest" I mean that it's the one with higher value, so big endian means that the first byte is the most significant byte, and little endian is the opposite. This simple mnemonic is very easy to remember and I never forgot which one was which after learning it this way.

So how can we determine machine endianness with C code? We can declare an `int` variable and assign it the value `1`. Then we just have to check on which of the bytes the `1` is stored. If the first byte is `00000001`, it means we're on a little endian machine. Otherwise, it's big endian. However, I'm not going to declare an `int` like normal people. We need to control each "block" of the integer. And what's the smallest block of bits we can process in a C program? It's a `char` block, of course, since it is guaranteed that `sizeof(char) == 1`. Because `sizeof` operator returns a value whose reference is `sizeof(char)`, that is, if `sizeof(X)` returns `Y`, it means `X` uses the same space as `Y` chars, we can do this:

```c
int is_bigendian() {
	unsigned char i[sizeof(int)] = { 0 };
	i[sizeof(int)-1] = 1;
	int *p = (int *) i;
	return *p == 1;
}
```

What is going on in here? Well, we declare an array with exactly the required number of blocks to hold an `int`. For example, if `char` is 8 bits and `int` is 32 bits, then `sizeof(int) == 4`. If `char` is 16 bits, then `sizeof(int) == 2`. This works on any computer with any architecture: we are simply representing our `int` as a set of unitary blocks, each block having the number of bits used by a `char`, the basic storage unit in C.

Remember that arrays are sequentially allocated in memory, that's why you can do pointer arithmetic to navigate through the array elements. This means that our `int` blocks are consecutive in memory. Thus, we set the last block of our manipulated `int` representation to 1. And now, the fun begins. We create a pointer to an `int`, and we make it point to the first position of the array. Note that the cast of `i` to `int *` is safe, since the array is assured to be exactly the same size of an `int`.

By this time, we're pretty much done. We just have to ask C to interpret the `int` pointed to by `p`. This is the crucial instruction that is determining machine endianness. Everything in our array is 0, except for the last block, which is 1. So, if `*p` is 1, it means that our machine interpreted the last block of the `int` as the least significant byte, which implies that the first byte is the most significant, meaning it's big endian. Otherwise, it's little endian. If for some reason you're not comfortable with this uncommon array usage, there is an equivalent altenative:

```c
int is_bigendian() {
	unsigned int i = 1;
	char *p = (char *) &i;
	return *p != 1;
}
```

This is virtually the same thing, we just switch roles and now we actually declare an `int` to be 1, and we use a `char` pointer to look at the first block of the `int`.

Now, coming back to the original version: what is `*p` equal to when we're on a little endian machine? Well, assuming that an `int` is represented by `n` bits and a `char` uses `m` bits, then `*p` will be \\(2^{n-m}\\). For example, if an `int` is 32 bits and a `char` is 8 bits, `*p` is \\(2^{32-8} = 2^{24} = 16777216\\). Indeed, if you add a `print()` instruction before returning from `is_bigendian()`, you can easily check that for this example it will print `16777216`, which is the same as `1 << 24`.

Why \\(2^{n-m}\\)? Well, if we're on a little endian architecture, it means we have an `int` with the most significant block consisting of \\(m-1\\) 0's and a 1 in the end of that block. It's a question of applying powers of 2 in a binary representation: the first bit is \\(2^{n-1}\\), the second is \\(2^{n-2}\\), and so forth. We have the \\(m^{th}\\) bit set to 1 in that block (and that's the only bit in our `int`), so the final result is \\(2^{n-m}\\).
