---
layout: post
title: Floating points 101
---

A brief introduction to IEEE 754 standard for float representation

-----

Most people use `float` pretty much the same way as they use `int`. After all, what's so special about a `float`? It's no big deal, right?

Lots of things can go severely wrong with floating point arithmetic. Lots of people know this, but they tend to ignore it until the shit hits the fan. Bugs with floating point numbers can be really, really, really mysterious and extremely hard to track down. In this post, I will cover a very common standard, IEEE754m, the one on which most machines rely on to represent floating point numbers as a sequence of bits.

The real question here is: how can a computer represent fractional number using 0's and 1's?

IEEE 754 defines the representation that is nowadays used in most computers, using 32 bits to represent a float. The basic idea is: some bits represent what we call the *base*, others represent an *exponent*, and one bit represents the *sign*. Remember scientific notation from highschool? It's sort of the same thing. But this time, instead of using a base 10 exponent, we will be using a base 2 exponent. So for example, if you want to represent 0.5, in base-2 scientific notation it will be \\(1 \times 2^{-1}\\). We can represent every floating point number as a quantity `A` multiplied by the `i`-th power of 2: \\(A2^{i}\\). IEEE 754 uses the 23 least significant bits to represent `A`. `A` is technically referred to as the *mantissa*. The next 8 bits - that is, the exponent bits - represent `i` in a normalized form. Finally, the most significant bit represents the sign: `0` for a positive number, `1` for negative. Pictorially, we can see it like this: (`S` is the most significant bit, the sign bit, `E` represents the exponent bits, and `M` the mantissa):

```
S E E E E E E E E M M M M M M M M M M M M M M M M M M M M M M M
```

Remember that we're working in binary. However, if we were to pause for a bit and go back to good old base-10, we could see that one of the problems of representing floats as a combination of mantissa and exponent is that the same number can have different representations, and as such, different bit patterns can represent the same number. For example, \\(24.68\\) has inifinitely many representations, such as:

* \\(24.68 \times 10^0\\)
* \\(2.468 \times 10^1\\)
* \\(0.2468 \times 10^2\\)
* \\(246.8 \times 10^{-1}\\)

And so on...

This is bad because determining whether 2 bit patterns represent the same number is not a simple bit-by-bit comparison anymore. Argh!

Anyways, equality is not the main issue here. In fact, we will see that strict equality tests are problematic in floats, but let's focus on something else now. The problem is that huge waste we would fall into. If there are numerous ways to represent a single float, then we can represent much, much less numbers than we actually thought.

IEEE 754 defines a canonical representation known as the *1.m representation*. It fixes the problem by having the mantissa bits define the decimal part of the number that comes after an implicit 1 to the left of the decimal point. The exponent is then adjusted accordingly. This is a little tricky to understand at first, so let's see an example. Suppose we have:

| S   | E   | E   | E   | E   | E   | E   | E   | E   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 1   | 1   | 1   | 1   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   |

The mantissa is `11110000000000000000000`, so, according to what I mentioned earlier about the 1.m representation, this is the decimal part of the number and there's an implicit 1 on the left, meaning we end up with `1.11110000000000000000000`. The leading `1.` is **NOT** represented in the bit pattern anywhere, like I said, it is assumed that there is always a `1.` prefix and that the mantissa bits only describe the decimal part.

Now, because the exponent is 1, we just have to move the decimal point one place to the right (multiplying by \\(2^1\\) moves the decimal place to the right, just like in base 10; \\(2^{-1}\\) would move it one place to the left), getting our final result:

$$
11.1110000000000000000000
$$

Converting to base-10, this is a number with integer part equal to 3 and decimal part equal to \\(\frac{1}{2} + \frac{1}{4} + \frac{1}{8} = 0.875\\), so it looks like `00000000111110000000000000000000` represents \\(3.875\\). We will see this is not quite right though - keep reading.

You might be wondering if the 1.m representation really works for every number. For one, how on earth do we represent 0?

As it turns out, there is a valid 1.m representation for every number except 0.

If the number being represented is not 0, then there must be at least a 1-bit somewhere. From most significant to least significant, we look for the first bit that is not a 0. The mantissa bits will be all the bits to the right of that bit. Again, the first 1-bit is never explicitly stored; it is implicitly there because of the 1.m representation. Then, we adjust the exponent to place the decimal point in the right place.

Ok, but what about 0? The standard defines a special case for 0: every bit set to 0. That's it. So, `00000000000000000000000000000000` represents 0. That's exciting.

However, now we can't represent 1, because by seeing all zero's we don't know if the intention was to represent a 0 (and thus ignore 1.m representation) or if there's an implicit `1.`. What gives?! Oops!

This is where the shifted exponent rule comes in. Basically, the exponent is shifted by 127, that is, to get the real exponent, one needs to subtract 127 from the original exponent.

So, in the previous example, the real exponent is \\(1-127 = -126\\). So instead we will move the decimal point 126 positions to the left, yielding \\(2^{-126} + 2^{-127} + 2^{-128} + 2^{-129} + 2^{-130}\\), which is something like \\(2.2775203 \times 10^{-38}\\).

With this in mind, one would assume that the 0-representation convention together with the shifted exponent rule implies that the smallest possible exponent is \\(-126\\). But that is not the case. It is possible to add more bits to the exponent as long as the regular 8 bits reserved for the exponent are all 0. When the 8 exponent bits are 0 but the mantissa has at least one bit that is not 0, the 1.m representation is thrown away - completely forget about it - and the exponent extends itself to use every bit up to (and including) the first 1-bit in the mantissa. For each additional digit, the exponent is shifted one more unit.

Again, an example: 

| S   | E   | E   | E   | E   | E   | E   | E   | E   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   |

Because each of the original 8 exponnt bits is 0 and the mantissa is not 0, then the exponent extends up to the first 1-bit in the mantissa. It used 15 more bits than it was supposed, so our new exponent is -126-15 = -141, so this is representing \\(2^{-141}\\), which is roughly \\(3.6 \times 10^{-43}\\). However, this loses precision, because the exponent "stole" bits from the mantissa.

Interesting fact: what happens when the only 1-bit in the mantissa is the rightmost bit? Well, the same rule applies, so basically we get \\(2^{-126-23} = 2^{-149} = 1.40129846 \times 10^{-45}\\), the smallest value representable under IEEE 754. This is the extreme case where we only get 1 precision bit - the rightmost bit!

There are a few more conventions regarding the rules to represent `NaN` (not a number) and infinity. \\(\pm\infty\\) is represented by having every bit in the exponent being a 1 and every bit in the mantissa being a 0. The sign bit determines whether it's positive or negative infinity. When every bit in the exponent is 1 AND there is at least one mantissa bit set to 1, then it's `NaN`.

`NaN`, short for *not a number* is the result of an invalid operation, like dividing by 0. `NaN` usually propagates among operations. That is, if some intermediate operation results in a `NaN`, further operations depending on that one will result in a `NaN`.

And that's basically all there is to it. Let's make a simple summary of the steps that go into decoding a float represented in IEEE 754 into a base-10 decimal:

* Read the mantissa bits. Add the implicit "1." to the left of the mantissa bits.
* Read the exponent, and subtract 127 to obtain the real exponent.
* Shift the decimal place to the right or to the left according to the exponent value and sign.
* Convert the integer part to base-10.
* Convert the decimal part
* Enjoy every moment of it :)

What about encoding a base-10 floating point into this representation?

Let's go through an example and pick the number \\(11.47\\). The first step is to convert it into a binary number. \\(11 = 8 + 2 + 1\\), so the integer part is \\(1011\\).

Now, for the decimal part, we can see that \\(0.47\\) is infinite in base 2 (you may want to see [Formatting your mind for base-b]({% post_url 2013-05-29-formatting-your-mind-for-base-b %}) to learn how to convert base-10 fractions to base-2). In binary, \\(0.47\\) is

$$
0111100001010001111010111(0111100001010001111010111)\dots
$$

I deliberately chose this contrived example; converting base-10 fractions into base-2 that are already base-2 fractions takes away all the fun of it (imagine if the example was 11.5 - that's damn easy!). Also, because our integer part is 11, and after the first 1-bit we have 3 bits, we will have to move these 3 bits into the mantissa bits because of the 1.m representation, thus losing 3 bits for the decimal part.

So, the mantissa for this example is:

$$
01101111000010100011110
$$

And now comes the exponent. We need to move the decimal point 3 places to the right, meaning our exponent will be \\(3+127 = 130\\), which is `10000010` in binary. The number is positive, so the sign bit will be 0. So, 11.47 is represented as:

| S   | E   | E   | E   | E   | E   | E   | E   | E   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   | M   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | 1   | 0   | 0   | 0   | 0   | 0   | 1   | 0   | 0   | 1   | 1   | 0   | 1   | 1   | 1   | 1   | 0   | 0   | 0   | 0   | 1   | 0   | 1   | 0   | 0   | 0   | 1   | 1   | 1   | 1   | 0   |

When I tested this example, I noticed that the last bit of the mantissa was set to 1. Instead of ending with `11110`, it ended with `11111`. I suspect this is because of rounding that was made when the 3 decimal precision bits were lost. The interesting fact is that if it was \\(1.47\\) instead of \\(11.47\\), which doesn't steal bits from the mantissa, the last bit would indeed be a 0. In fact, for \\(1.47\\), the last mantissa bit is not the same bit as for \\(11.47\\) - it's the third bit counting from the end. So, yeah, it must be some rounding that is being computed. Because we have a finite set of bits, the computer must round floats accordingly.

What about doubles? Doubles work exactly the same way, except you get to have 52 bits for mantissa and 11 bits for the exponent. Oh, and the exponent shift amount is 1023.

### Ranges for floats using this representation

In a nutshell, these are the most important informations to keep in mind about float ranges:

* Largest representable number: \\(3.402823466\times10^{38}\\)
* Smallest representable number without losing precision: \\(1.175494351\times10^{-38}\\)
* Smallest representable number with only 1 precision bit<: \\(1.401298464\times10^{-45}\\)
* \\(\epsilon: 1.1929093\times10^{-7}\\)

\\(\epsilon\\) (epsilon) is the smallest value such that \\(1 + x > 1\\).

### Impacts in day-to-day programming

This is a delicate topic. It depends a lot on what type of application we're talking about However, there are a few general guidelines that one must always keep in mind when working with floats. There are lots of pitfalls and weird bugs that can show up.

For example, strict equality tests with floats are often doomed.

Why is it so hard to tell if two floats are the same? The problem with strict equality is that it is based in a bit-by-bit comparison. Because of rounding in intermediate calculations, it might happen that 2 numbers that were supposed to be the same were rounded in different ways and thus will compare different.

For example, math libraries providing functions like `sin()`, `cos()`, `tan()`, etc, are implemented using polynomial approximations, and you might be asking too much if you expect \\(cos(\frac{\pi}{2})\\) to be strictly equal to `0.0`, or \\(sin(\pi)\\) to be exactly `-1.0`. The *operations* themselves are inaccurate. And how do you represent \\(\pi\\)? Can you even represent it? You can't, you have to make an approximation. So when you call `sin()` or `cos()`, you're not really calling it with \\(\pi\\), you're calling it with something very close to \\(\pi\\). And then, like I said, `sin()` is a polynomial approximation, so the bottom line is that you end up with an approximate value of the sinus of something that is almost \\(\pi\\).

How can we fix this? Well, we really can't, but you as a programmer can decide what is "close enough" to accept as "equal". For example, you can define a constant and test for equality like this:

```c
#define EPSILON 0.0000001
#define is_equal(a,b) (fabs((a)-(b)) <= EPSILON)
```

This has been a widely used technique, but it's got its drawbacks as well. What if the numbers are totally different but have a really small exponent?

Take, for example, \\(3.14896\times10^{-15}\\) and \\(1.22246\times10^{-15}\\). Their difference is \\(1.9265\times10^{-15}\\), which is way smaller than `EPSILON`, so that macro would indicate that the numbers are equal. What if we had \\(3.14896666666666\times10^{-15}\\) and \\(3.14896666666667\times10^{-15}\\)? This is a completely different case, and they are so close that we can assume they're the same. But the macro can't distinguish between those 2 cases.

This is very bad news. Notice that any pair of numbers \\(n_1\\) and \\(n_2\\), with exponents \\(a_1\\) and \\(a_2\\) (that is, \\(n_1 = c\times10^{a_1}\\) and \\(n_2 = x\times10^{a_2}\\), where `c` and `x` are some constant) such that \\(max(a_1,a_2) < \epsilon\\) (\\(10^{-7}\\) in this case) are all considered equal by the code shown above. This is fine if your program never reaches cases like this; it's your job to analyze it and find the best method.

How can we solve this? The problem with the epsilon method is that it is assuming that the exponents are close to 0. It's not very intuitive to understand this at first. Let me try a different approach: if the exponents were always 0, or close to 0, every number would be \\(c\times10^{0} = c\\), so, for some `c` and `c'`, if \\(c-c' \le \epsilon\\) then in fact we could consider them to be the same. But the exponent will most likely not be 0, and different numbers with different exponents representing different levels of magnitude will turn out to be equal when we didn't want them to be.

Just to make sure you understand: imagine our examples with the exponent set to 0. The first example would become \\(3.14896\\) and \\(1.22246\\). The difference between those 2 numbers is \\(1.9265\\) (the same result, but no exponent!), which is greater than `EPSILON`, so the numbers are not the same. The 2nd example would become \\(3.14896666666666\\) and \\(3.14896666666667\\). The difference is now \\(1\times10^{-14}\\), smaller than `EPSILON`, so this pair is correctly identified as being equal. With this, I hope you understand why the method assumes that the exponents are close to 0. It only works properly in two cases:

* The exponents are always 0, or
* The exponents never go below \\(\epsilon\\)

To solve these naughty cases, we will have to write our own comparison function for floats. It will work like this: first, it attempts a strict equality test. If that doesn't work, it will compare the exponents. If the exponents are different, it immediately returns `false` (this covers all the cases in which 2 numbers, both with different exponents less than `EPSILON`, were considered equal). If they are the same exponents, then we can simply ignore the exponents, by setting it to 0, and applying our original `EPSILON` technique to these new numbers.

Convince yourself that this method works. The problem of \\(3.14896\times10^{-15}\\) and \\(1.22246\times10^{-15}\\) being "equal" in the former method is merely because of the exponent - again, this all goes back to the "assume the exponent is 0" issue. But if the exponents are the same, then you're really only interested in how many significant digits are the same in the two numbers. If you just want to do that, and you already know the exponent is the same, then you can remove the exponent, i.e., set it to 0. After you do that, you can use the original epsilon method to determine if the numbers are equal, because as we saw, it works well for cases where the exponent is 0.

The implementation of this scheme boils down to the use of the `ieee754.h` header file. This file exports a `union` that gives us access to the exponent and other components of a float.

```c
#include <ieee754.h>
#include <math.h>
#define EPSILON 0.0000001
#define is_equal(a,b) (fabs((a)-(b)) <= EPSILON)
int float_equal(float number1, float number2) {
  union ieee754_float *n1_ptr, *n2_ptr;
  unsigned int n1_exp, n2_exp;
  if (number1 == number2) {
    /* They are exactly the same */
    return 1;
  }
  n1_ptr = (union ieee754_float *) &number1;
  n2_ptr = (union ieee754_float *) &number2;
  if (n1_ptr->ieee.negative != n2_ptr->ieee.negative) {
    /* Different signs, don't even look at exponent... */
    return 0;
  }

  n1_exp = n1_ptr->ieee.exponent;
  n2_exp = n2_ptr->ieee.exponent;

  if (n1_exp != n2_exp) {
    /* Different number of decimal places */
    return 0;
  }

  /* If we get here, we can safely set the exponent to 0
   * This is typically 127
   */
  n1_ptr->ieee.exponent = IEEE754_FLOAT_BIAS;
  n2_ptr->ieee.exponent = IEEE754_FLOAT_BIAS;
  return is_equal(number1, number2);
}
[/code]

I ran a couple of tests (remember to compile this with -lm because of math.h):

[code language="cpp"]
#include <stdio.h>

int main() {
  /* Using pure epsilon technique */
  printf("%d\n", is_equal(2.124,
                          2.125));
  printf("%d\n", is_equal(0.00000001,
                          0.000000001));
  printf("%d\n", is_equal(0.000000001,
                          0.000000002));
  printf("%d\n", is_equal(0.199999999,
                          0.199999998));

  printf("--\n");

  /* Using our new, better technique */
  printf("%d\n", float_equal(2.124,
                             2.125));
  printf("%d\n", float_equal(0.00000001,
                             0.000000001));
  printf("%d\n", float_equal(0.000000001,
                             0.000000002));
  printf("%d\n", float_equal(0.199999999,
                             0.199999998));
  return 0;
}
```

This printed:

```
0
1
1
1
--
0
0
0
1
```

Very nice!!! But this is not the definite solution to every problem. It can misbehave when testing against 0. Most of the times, we won't have a perfect `0.0` value, instead, we will have something that is very close to zero. For example, \\(sin(\pi)\\) doesn't give us 0 - in my machine I get \\(1.22465\times10^{-16}\\). The test that checks if the exponents are different succeeds and it will say that \\(sin(\pi)\\) is not equal to 0.

### Accumulated errors

As we have seen, floats get "dirtier" everytime you make an operation with them. Math library functions and arithmetic operations will always return an approximate result, so the more computations we make on the same float, the more accumulatederror we will have. This can have dramatic, terrific impacts. Pick my previous example for \\(sin(\pi)\\). We got \\(1.22465\times10^{-16}\\). If our next step was to divide this by \\(10^{-16}\\), we would get \\(1.22465\\)! This is **way** far from the 0 we were expecting and illustrates how things can go terribly wrong in your code in the most mysterious, ugly way.

### Overflow

Dealing with overflow errors is one of those things that is easy in floats. When an operation with floats overflows, we get \\(\pm\infty\\), depending on which "side" overflowed. Infinity quantities work as expected, i.e., any number is less than \\(\infty\\), and greater than \\(-\infty\\).

### Consider only using `int`

If you're building some program that uses lots of fractions and you want very precise computations, consider representing fractions as a pair of `int`. This is often a good way to avoid loss of significance and accumulated error problems.

For interested readers, I suggest having a look at *What Every Computer Scientist Should Know About Floating Point Arithmetic* (google it).
