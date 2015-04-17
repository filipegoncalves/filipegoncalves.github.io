---
layout: post
title: C multi-dimensional arrays part 2 - converting flattened to unflattened index
---

Solving the question from my previous post

-----

Recently, I have been discussing C multi-dimensional arrays. I showed [how to pass them around without explicit dimension declaration]({% post_url 2014-01-27-c-multidimensional-arrays %}), and by the end of that article, I left an interesting challenge for the reader to solve:

Given a flat index `j` in an `N`-dimensional array with dimensions \\(n\_1\\), \\(n\_2\\), \\(n\_3\\), ..., \\(n\_N\\), can you retrieve each dimension index back? Write a function to do this:

```c
void get_index_back(char *result, const char *array_name, int dim[], int ndim, const char *index) {
    /* TODO: Implement me */
}
```

`result` should be a null-terminated characters sequence with the equivalent array indexing. `array_name` is the name of the array, `dim` is an array with the size of each dimension, `ndim` is the number of dimensions, and `index` is the name of the flat index variable.

As an example, this code:

```c
int dim[2] = {3, 4};
char name[1024];
get_index_back(name, "x", dim, 2, "j");
```

Should result in:

```c
name = "x[j/3][j%4]"
```

Which means that any flat index `j` in a \\(3\times4\\) array `x` is equivalent to `x[j/3][j%4]` in the unflattened version. Can you generalize it for an array with `N` dimensions?

As usual in any computer science problem, we can't deal with everything at once. Abstraction is the key to success, we have to look at a problem, understand it, decompose it, design a solution, and only after that can we code it. A good method that I'm very comfortable with in this kind of problems is to work with examples and then generalize. The art of arriving at a correct solution is, of course, to know how to generalize. It may be very hard sometimes. And guess what - if you do it the wrong way, *POOF*, your solution is wrong.

Consider the following picture, which shows the memory layout for a 3D array `x[2][3][5]`:

![3D Array Memory Layout]({{ site.baseurl }}{{ site.assets }}3darray.png)

If you have troubles understanding this, see [my other article]({% post_url 2014-01-27-c-multidimensional-arrays %}) where I explain this in detail.

The picture shows two example flattened indexes, `14` and `18`, which could be legitimately used to index this array when we treat it as a 1D array - this is just what I showed in the other article. We saw how to convert `x[0][2][4]` to `14`. Now, the question is how to go the other way around: convert `14` to `x[0][2][4]`.

Again, this is 4th grade maths, but the amount of CS students that gets really confused with this is both astonishing and worrying.

By looking at the picture, and combining that with the fact that C multi-dimensional arrays are just arrays of arrays (the notion of "dimension" is a compile time thing), it can be seen that our example is described by an array with 2 positions. Each of those holds something. What is that? It's an array with 3 positions, and each of those 3 is holding - guess what - an array of 5 things. I don't even care what type it is. Why would I? `int`, `char`, `float`, it's all the same at this level. Just look at it as an array with 2 positions, each holding *something*. Now, this is when it starts to get interesting: we have to know how many things each of these 2 positions holds, so that we can see into which of these 2 positions the offset `14` falls into. Right?

For example, our picture shows that `x[0]` and `x[1]` each have 15 "things", which means that `x[0]` spans the closed flattened interval `[0..14]` and `x[1]` spans `[15..29]`. This tells us that offset `14` is in `x[0]`. Why? 

Really, why? Don't look at the picture, let's generalize it.

The reason is simple - if each `x[i]` has 15 "things", then an offset `j` is in `x[j/15]` (we assume integer division here). Think of `x` as a set of buckets. Each `x[i]` is a bucket with 15 elements. Where is element 14? Bucket 0, that is, `x[0]`, will contain elements 0 to 14, bucket 1 contains 15 to 19, bucket 2 will have 30 to 44, etc. Tell me any number, and I can tell you which bucket it belongs to by calculating `j/15`.

Ok, I think that's clear enough. So, 14 maps to bucket 0, 18 maps to bucket 1. Sounds fair and correct. What about the next dimension? Ok, we know that 14 is in bucket 0, but now, *inside bucket 0*, there are 3 buckets, each of them has 5 things (remember, bucket 0 has 15 elements, which is \\(3\times5\\) - 3 buckets of 5 elements). Well, it's the same thing again, right? If each bucket has 5 elements, then we grab a flattened index `j` and calculate `j/5`. `14/5 = 2`, so it means that offset `14` is in bucket 2 of bucket 0 - correct.

Hold on for a second, we have a problem here. What happens with offset 18? `18/5 = 3`, but 18 is supposed to be in bucket 0 of bucket 1 (i.e., 18 belongs to `x[1][0]`). Oops! What happened here?

The problem is that we are never resetting bucket count. By calculating `18/5`, we are assuming that the whole array is a set of buckets of 5 elements, that's why 18 maps to 3. After all, if our array were really a set of buckets of 5 elements, then buckets 0, 1 and 2 would hold elements `[0..14]`, and bucket 3 would hold `[15..19]`, where 18 fits.

No! Wait! We're almost there. Remember, we have buckets of 5 elements, but each of these buckets is also inside another, higher level bucket (the first dimension, `x[0]` and `x[1]`), that groups these buckets of 5 into 3-tuples. So, what we really want to do is to reset bucket count every 3 buckets when looking at 18. The operator that does this is the integer division remainder (modular arithmetic). We don't want `18/5`, we want `(18/5)%3`, because we need to reset bucket count every 3 buckets. It's like you looked at the picture, and said: "Ok, 18? Let's see, I'm going to split it into bucket of 5 elements. Here's bucket 0, bucket 1, bucket 2 ... bucket 0, bucket 1, bucket 2...", and you keep doing that until you hit the bucket with 18. We have to reset counting because of the higher, previous dimension.

`14/5` just happened to work because 14 is in the first dimension (`x[0]`), so we never really have to reset bucket count for the second dimension. The formula fails for every other offset that is in `x[1]`. Thus, the correct expression, even if unnecessary, would be `(14/5)%3`. And the same happens for `14/15` and `18/15` (the formula for the first dimension), even though it is not needed at all for the first dimension - we assume well behaved code that will not indicate an out of bounds offset. But if we want to generalize, we could say that by this point we have:

```
unflattened_expression = x[(j/15)%2][(j/5)%3]
```

We're almost done: we just have to keep repeating this. The 3rd and last dimension will not need integer division - the last dimension will always look at the rest of the array as buckets of 1 element, and dividing by 1 is useless. What we do need is the remainder: once again, in this specific case, we want to reset element count every 5 elements, because the last dimension has 5 elements. Thus, we need `j%5`. Here's the final conversion result:

```
unflattened_expression = x[(j/15)%2][(j/5)%3][j%5]
```

Give it any `j` in interval `[0..44]`, and that's the same as indexing `x[(j/15)%2][(j/5)%3][j%5]` in the original, multi-dimensional array.

The next step? Generalize it for arbitrary dimension sizes! Where did that 15 in `x[(j/15)%2]` come from? What about `(j/5)%3`? This shouldn't look like black magic if we have made it this far. 15 is the number of elements in each of the highest level (first dimension) buckets, which is \\(3\times5\\) (because each bucket has 3 arrays of 5 elements): it's the product of every dimension except the first one. The remainder operator is applied to the current dimension, that is, if we're dealing with dimension `i`, then our results will be modulo \\(n_i\\).

Just stay with me on this one: `x[(j/15)%2][(j/5)%3][j%5]` comes from `x[(j/(3*5))%2][(j/5)%3][j%5]`. The numbers 2, 3 and 5, which are the second operand of the remainder operator, correspond to the size of each dimension \\(n_i\\) (that makes perfect sense - as we saw earlier, we have to reset bucket counting because we want to stay within each dimension size), and the divisor `k` in `j/k` is the product of every dimension \\(n_i\\) such that `i` is greater than the current dimension - after all, in dimension `i`, we look at our array as a set of buckets with `p` elements, where `p` is the product of the next dimensions.

With all of this in mind, assuming you're not lost / sleeping / apathetic / not thinking, we can almost intuitively derive the general formula:

$$
index(i, j) = mod(\frac{j}{\prod_{k=i+1}^N{n_k}}, n_i)
$$

Where `i` denotes the current dimension (for `x[2][3][5]`, `i` takes the values 1, 2, and 3, and \\(n_i\\) takes the values 2, 3, 5 - if you've read the other article, you know this notation), `j` is the flattened index variable, and \\(mod(x,y)\\) means `x%y`, that is, the remainder of the integer division `x/y`.

As far as the implementation is concerned, we will not want to compute the product every time we need it, so we might want to calculate all the products we will need and store them in an array. Because at each step `i` we need the product of every dimension greater than `i`, the best approach is to use a cumulative array of multipliers, where `mul[i]` is the product of every dimension greater than `i`. And of course, be smart when building it - it is more convenient to fill it from right to left.

Here's my implementation:

```c
void get_index_back(char *result, const char *array_name, int dim[], int ndim, const char *index) {
    int mul[ndim];
    mul[ndim-1] = 1;
    for (int j = ndim-2; j >= 0; j--)
        mul[j] = mul[j+1]*dim[j+1];
    strcpy(result, array_name);
    result += strlen(array_name);
    for (int j = 0; j < ndim; j++)
        result += sprintf(result, "[(%s/%d)%%%d]", index, mul[j], dim[j]);  
}
```

This is C99 code, it uses variable length arrays (`mul`), it uses mixed code and declarations, and it has declarations inside statements (the `for` loops variables), to minimize their scope. This is just a matter of style, I am getting used to more recent standars (and learning C++). To compile (in gcc), you must use `-std=c99`.

Note that this function has no way of preventing buffer overflow - it is assumed that `result` is big enough to hold everything. This is reasonable to assume, but in real world code, we would have to worry about it and use `strncpy()`, `snprintf()`, and stop the loop earlier if there was no space left (and pass the size of `result` as an argument).

Another limitation of this code is that it can very easily overflow when multiplying dimensions. Be careful if you plan to use this with very large dimensions.

See what I did here - analyze the code and enumerate some of its limitations? That's important. As soon as you finish coding something, always do this. It's a good exercise, and it shows to others that you care about it and that you are knowledgeable enough to be aware of your solution's limits.

It is funny and glorifying to test this with \\(3\times4\times8\times7\times6\times9\\) arrays, or something bigger, and see how quickly and easily things get messy. Here's a small test for a \\(3\times4\times8\times7\times6\times9\\) array:

```c
int main(void) {
    int dims[] = { 3, 4, 8, 7, 6, 9 };
    char name[256]; /* Should be enough */
    get_index_back(name, "x", dims, 6, "i");
    printf("%s\n", name);
    return 0;
}
```

This prints:

```
x[(i/12096)%3][(i/3024)%4][(i/378)%8][(i/54)%7][(i/9)%6][(i/1)%9]
```

Isn't that cool?
