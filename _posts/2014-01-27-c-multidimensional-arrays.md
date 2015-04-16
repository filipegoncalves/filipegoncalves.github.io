---
layout: post
title: Some thoughts on C multi-dimensional arrays
---

How to pass multi-dimensional arrays around without explicit dimension size declaration

-----

Arrays are useful. They allow us to pack items into contiguous memory locations with the benefit of \\(\mathcal{O}(1)\\) random access.

Oh, wait a second, I can't resist it. Since I saw [this question on StackOverflow about RAM memory](http://stackoverflow.com/questions/20960406/how-is-ram-able-to-acess-any-place-in-memory-at-o1-speed), I can't take my mind off it. So, for nitpickers (I don't really think anyone would go that far), here's some crazy facts of life and thoughts on RAM memory layout, taken from the most voted answer to the question:

> Yes, in the physical world, memory access cannot be constant time.
>
> But it cannot even be logarithmic time. The O(log n) circuit you have in mind ultimately involves some sort of binary (or whatever) tree, and you cannot make a binary tree with constant-length wires in a 3D universe.
>
> Whatever the "bits per unit volume" capacity of your technology is, storing n bits requires a sphere with radius O(n^(1/3)). Since information can only travel at the speed of light, accessing a bit at the other end of the sphere requires time O(n^(1/3)).
>
> But even this is wrong. If you want to talk about actual limitations of our universe, our physics friends say the absolute maximum number of bits you can store in any sphere is proportional to the sphere's surface area, not its volume. So the actual radius of a minimal sphere containing n bits of information is O(sqrt(n)).

This is crazy. For our purposes, we can certainly assume it is \\(\mathcal{O}(1)\\). Don't worry, no one will ever come down on you because you think that RAM is \\(\mathcal{O}(1)\\), because, well, it is, we all know, for the practical purposes of real world, that this is the case. Come on, seriously? *you cannot make a binary tree with constant-length wires in a 3D universe*, what does that even mean? Heh...

Anyway, let's get back on track. C, and C-like languages, provide support for `N` dimensional arrays. Passing one dimensional arrays across function calls is straightforward, but passing arbitrary `N` dimensional arrays requires you to explicitly declare the size of all but the first dimension in the function's prototype. This means that a function receiving multi-dimensional arrays is generally bound to specific dimension sizes. You can't easily feed a 10x20x30 and a 20x30x10 array to the same function from different places in the code; one of these calls will always cause a size mismatch and won't compile.

Bzzzzt! Let's look at how we can deal with this. First, we have to understand why this limitation exists.

## Why you have to supply dimensions other than the first one

It is a well-known fact that arrays in C easily decay into a pointer to the first element. When used in an expression, an array name always decays into a pointer, unless the array name is:

* The operand of `sizeof`
* The operand of the reference operator `&`
* A string literal initializer

A function call is an expression, so an array name passed as an argument decays into a pointer to its first element. This is why you don't have to provide the size of the first dimension (and if you do, the compiler ignores it): there is no array bound checking in C, so it's not really relevant. The size of the first dimension is useless to the compiler.

Why are the other dimensions important? We have to look at what is happening when we index an array on position `i`. You see, when you declare an array of `T`, then each array position will have enough space to hold an element of type `T`. An array of integers can fit `sizeof(int)` bytes into each position. Since arrays are nothing but a bunch of contiguous bytes in memory, indexing a position `i` in an array of `T` means that we must:

* Get the base address of the array, let it be `base_addr`
* Multiply the index by `sizeof(T)`, let's say we have `final_index = i*sizeof(T)`
* Calculate `base_addr+final_index` - this is the starting address for element `i`, let it be `start_addr`
* Read / write `sizeof(T)` bytes starting at `start_addr`

This means that indexing requires the compiler to know the size of each element in the array.

Multi-dimensional arrays are flatly laid out in memory. Let's take an example for a 2D array. Consider the following declaration:

```c
int n[5][10];
```

Programmers sometimes like to think of 2D arrays like a matrix:

![2D Array as table]({{ site.baseurl }}{{ site.assets }}2darray.png)

As Peter Van Der Linden once wrote, *never allow a programmer like this to park your car*. The storage and a reference to an individual element are laid out linearly in memory. The array `n` is seen as an array of 5 elements, where each element is an array of 10 integers. Thus, this array uses `5*10*sizeof(int)` bytes of memory. With multidimensional arrays in C, the rightmost subscript varies fastest; the memory layout for `n` looks like this:

![N-Dim array layout]({{ site.baseurl }}{{ site.assets }}2darray2.png)

This is why you have to provide the size of every dimension but the first one. The exact location of `n[1]` depends on the size of these dimensions, because the array is layed out in memory linearly. Moving to `n[1]` in this example means going `10*sizeof(int)` bytes forward, because each element in `n` is an array of 10 integers.

Likewise, if we were to define a 3D array `v`:

```c
int v[5][10][15];
```

Then `v[1]` is `10*15*sizeof(int)` bytes away. Remember, each element of `v` is an array holding 10 elements, each element being an array of 15 integers. So, each element in `v` uses `10*15*sizeof(int)` bytes. See how we're not using the first dimension, but we need the other dimensions? We need it because they are used to compute the size of each chunk in the array, so that indexing works as expected.

## Making the jump - provide your own indexing

The compiler requires you to declare the size of every dimension but the first one because it is doing this maths for you. You can write `v[2][4][8]` without thinking about this, there is absolutely no need to know how the array is laid out in memory.

But then of course, the secret to pass arrays with different dimensions to a function is to provide your own indexing, that is, make the compiler's job. You might as well give up using a compiler at all if you're really into doing this kind of stuff. I present it here for educational purposes, but I wouldn't recommend anyone to do this in real world code. If you have to, you most likely have a design flaw.

Basically, the idea is to pass a pointer to the first element in the array, and do the necessary maths to convert from a multi-dimensional index to a flat index. This is cheating at its best. Let's get to the maths of it before writing any code.

From the previous examples, it is kind of easy to come up with a general rule. Our goal is to develop a formula that, given a set of indexing positions \\(i_j\\), where \\(i_j\\) corresponds to the `j`-th index (assuming it is within its dimension bounds), in an array of `N` dimensions, where \\(n_i\\) denotes the `i`-th dimension, produces the corresponding flat index. That's it. That's all we want. An example is often useful, so let's look at one. Imagine an array with 4 dimensions:

```c
int arr[10][4][3][2];
```

Here, `N` is 4, and \\(n_1 = 10\\), \\(n_2 = 4\\), \\(n_3 = 3\\), and \\(n_4 = 2\\). This indexing:

```c
arr[0][1][2][1];
```

Is equivalent to \\(i_1 = 0\\), \\(i_2 = 1\\), \\(i_3 = 2\\), and \\(i_4 = 1\\). In memory, the array is represented like this:

![4-D array layout]({{ site.baseurl }}{{ site.assets }}4darray_indexing.png)

In this example, `arr[0][1][2][1]` corresponds to the 11th position in a flat indexing scheme (starting at position 0). Note the thick vertical black lines denoting the different dimensions. The longest line separates the leftmost dimension, the second longest separates \\(n_2\\) inside \\(n_1\\), and the shortest lines separate \\(n_3\\) inside \\(n_2\\). We don't need to separate \\(n_4\\) inside \\(n_3\\) because, well, at this point we have single elements (integers), so they are implicitly isolated from each other. This picture absolutely shows how the compiler perceives the array and uses the dimensions to calculate the byte offset of an index. If we want to support different dimension size in the same function, we have to cheat on the compiler: we will no longer tell it that this is a multi-dimensional array; instead, we will rely on the fact that arrays are linearly layed out in memory, and provide our own indexing. So, inside our function, we will receive a pointer to the first element (pointer to `int` in this case), so that the compiler blurs away the virtual boundaries created by the vertical lines in the image. Instead, this is what we will have:

![4-D array as a 1-D array]({{ site.baseurl }}{{ site.assets }}4darray_indexing_flat.png)

It is still the same array with 4 dimensions, but because we passed a pointer to the first element, this is what the compiler sees. With this, we can write a function that calculates array indexing during runtime. The advantage is that it is not tied to a set of specific, hardcoded dimensions (but of course, `N` cannot vary).

So how do we go from the multi-dimensional index `arr[0][1][2][1]` to the equivalent `arr[11]`? This is 4th grade maths, all we need is multiplication and sum. The secret is to interpret the size of each array element in a different way depending on the dimension we're looking at. We start with an empty value (0) as a result, and we go through each index one at a time. The first dimension allows things like `arr[0]`, `arr[1]`, `arr[2]`, ..., `arr[9]`. Each of these positions is an array of 4 arrays of 3 arrays of 2 integers. So, `arr[0]` holds \\(4\times3\times2 = 24\\) integers. Each `arr[i]` is a set of 24 integers, which means that multiplying \\(i_1\\) by 24 should give us the flat index of the first position in the next sub-array. For example, if we were to index `arr[4][1][2][1]`, then \\(4\times24 = 96\\) is the flat index for `arr[4]`. The first position in `arr[4]`, which is equivalent to `arr[4][0][0][0]`, is the 96-th position on the flat layout. Similarly, we deal with the first dimension in `arr[0][1][2][1]` by calculating \\(0\times24 = 0\\), meaning that 0 is the first position in `arr[0]` in a flat version. And now we can forget about the first dimension and work on the other ones. The process is the same; we can recursively apply this rule for the next dimension, and add it all up together to get the final result.

In the case of `arr[0][1][2][1]`, we start with \\(0\times24 = 0\\). Then, we get to the next dimension: \\(i_2 = 1\\). Given that `arr[i][j]` is an array of 3 arrays of 2 integers, then `arr[i][j]` can be seen as holding blocks of 6 integers; \\(0\times24+1\times6 = 6\\) - this is our current result. Moving on to the next dimension, we have \\(i_3 = 2\\), each `arr[i][j][k]` is supposed to be an array of 2 integers, because we are indexing position 2, we are \\(2\times2 = 4\\) positions away from the beginning of this dimension. \\(0\times24+1\times6+2\times2 = 10\\). Then we have the last dimension indexing: this is when we don't have to multiply because each block is already a unit of known size (namely, `sizeof(int)` bytes), and the compiler knows this, so we just have to add this offset to our current result: \\(0\times24+1\times6+2\times2+1 = 11\\).

The general formula is:

$$
i_N + \sum\limits_{j=1}^{N-1} {(i_j * \prod\limits_{k=j+1}^N n_k)}
$$

Basically, at each step we go to the beginning of the next dimension, until we are out of dimensions, in which case we add the final offset \\(i_N\\) to go to the exact position. Beautiful and intuitive.

One of the most fundamental principles in programming is that of abstraction: we don't want to think about this every time we want to work with multi-dimensional arrays in functions that provide this flexibility. We can quickly code a macro to perform this conversion for us. For the previous example, this would do it:

```c
#define array_index(ptr, n2, n3, n4, i1, i2, i3, i4) \
        (ptr)[(i1)*(n2)*(n3)*(n4)+(i2)*(n3)*(n4)+(i3)*(n4)+(i4)]
```

Note that we don't need \\(n_1\\), the first dimension. We can use it like this:

```c
/* Access position arr[0][1][2][1] on a Yx4x3x2 array, where Y > 0 */
array_index(arr, 4, 3, 2, 0, 1, 2, 1) = 59;
```

or:

```c
int a  = array_index(arr, 4, 3, 2, 0, 1, 2, 1);
```

Or, for that matter, any other place where an array could be used. Here's a small testing program:

```c
#include <stdio.h>
#define array_index(ptr, n2, n3, n4, i1, i2, i3, i4) \
        (ptr)[(i1)*(n2)*(n3)*(n4)+(i2)*(n3)*(n4)+(i3)*(n4)+(i4)]

void f(int *arr, int dim1, int dim2, int dim3, int dim4) {
  array_index(arr, dim2, dim3, dim4, 0, 1, 2, 1) = 59;
  array_index(arr, dim2, dim3, dim4, 8, 3, 2, 0) = 14;
  printf("%d %d\n", array_index(arr, dim2, dim3, dim4, 0, 1, 2, 1),
	 array_index(arr, dim2, dim3, dim4, 8, 3, 2, 0));
}

int main() {
  int arr[10][4][3][2];
  f(&arr[0][0][0][0], 10, 4, 3, 2);
  printf("%d %d\n", arr[0][1][2][1], arr[8][3][2][0]);
  return 0;
}
[/code]

As expected, it prints:

[code language="cpp"]
59 14
59 14
```

Of course, you should be aware of the extra overhead of using this approach. Every time we use the `array_index()` macro, it recalculates `n2*n3*n4` and `n3*n4`. As usual, flexibility comes at the cost of efficiency. You can't expect to have a function dealing with arrays having different sizes on each dimension being as good as the version with the hardcoded dimensions. Bear in mind, though, that there really is nothing you can do to make it much more efficient. After all, if we were to declare `f()` with the correct dimensions:

```c
void f(int arr[][4][3][2]) {
/* ... */
}
```

Then all the compiler could do is replacing the equivalent of `n2*n3*n4` and `n3*n4` with the hardcoded values 24 and 6. That's a substantial improvement, but then again, you lose flexibility. Sorry, life is all about tradeoffs!

So, the bottom line is: if you really want to do this, you have to:

* Declare your function such that it receives a pointer to the first element of the array, and the sizes of each dimension but the first one (unless, of course, you want to do bounds checking). In an `N`-dimensional array, the pointer is obtained with `&array[0][0][0]....[0]`, with the `[0]` repeated `N` times,  or with `array[0][0][0]....[0]`, with `[0]` repeated N-1 times (so that the result of the expression is array of size \\(n_N\\), which decays into a pointer to its first element).
* Provide your own indexing. This is the fun part. Use the general formula provided above to code a macro that translates multi-dimensional indexing into a flat index, so that you can abstract away these sneaky little details and think of it as an `N`-dimensional array again. The formula becomes quite simple for 2D arrays (which is the most common case), so you might even drop the macro abstraction if that's the case.

And now, for the interview question: How would you go the other way around? That is, given a flat index `j` in an N-dimensional array with dimensions \\(n_1\\), \\(n_2\\), \\(n_3\\), ..., \\(n_N\\), can you retrieve each dimension index back? Code up a function to do this:

```c
void get_index_back(char *result, const char *array_name, int dim[], int ndim, const char *index) {
    /* TODO: Implement me */
}
```

`result` should be a null-terminated character sequence with the equivalent array indexing. `array_name` is the name of the array, `dim` is an array with the size of each dimension, `ndim` is the number of dimensions, and `index` is the name of the flat index variable.

As an example, this code:

```c
int dim[2] = {3, 4};
char name[1024];
get_index_back(name, "x", dim, 2, "j");
```

Should result in:

```
name = "x[j/3][j%4]"
```

Which means that any flat index `j` in a 3x4 array named `x` is equivalent to `x[j/3][j%4]` in the unflattened version. Can you generalize it for an array with `N` dimensions?

I'll show my solution in the next post!
