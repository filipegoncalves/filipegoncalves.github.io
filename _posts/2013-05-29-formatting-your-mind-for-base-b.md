---
layout: post
title: Formatting your mind for base b
---

*There are 10 types of people in this world, those who understand binary and those who don't*

-----

I want to refresh everybody's memory and talk a little bit about base-10 to base-b conversion.

Most people will be familiar with conversion to binary, but sometimes it starts to get really messy when fractional numbers come into play.

For the record: I will not be covering how to convert base-10 integers to base-2, or any other base. I'm going to focus in fractional numbers.

How would you implement a function that receives a fractional number in the interval `[0..1[` in base-10 and converts it to a char array representing it in base-b?

It may sound complicated at first, but it's not. Remember this algorithm?

* Multiply by 2
* If there's an integer part, it's a 1-bit
* Otherwise, it's a 0-bit
* Subtract the integer part
* If 0, stop. Otherwise, go back to 1

This is the basic way to convert a base-10 fractional number in `[0..1[` into its base-2 variant.

Let's see a few examples.

### Example 1

In this example, let's take the value \\(0.125\\).

In each row, we multiply the previous row's fractional part by 2. The first row corresponds to \\(0.125 \times 2 = 0.25\\).

| Fractional part | Integer part |
| --------------- | ------------ |
| 0.25            | 0            |
| 0.5             | 0            |
| 0               | 1            |

The algorithm stops when the fractional part is \\(0\\). The fractional representation of \\(0.125\\) is then obtained by concatenating the integer parts (2nd column). So, \\(0.125\\) in binary is \\(0.001\\).

### Example 2

Let's try a different value: \\(0.2\\)

| Fractional part | Integer part |
| --------------- | ------------ |
| 0.4             | 0            |
| 0.8             | 0            |
| 0.6             | 1            |
| 0.2             | 1            |
| 0.4             | 0            |

Wooohha! We got a loop! Cool! This is an infinite fraction in binary. \\(0.2\\) in binary is \\(0.00110011...\\)

This  drives us into fun fact number 1 about base-b:

**Some fractions can be finite in one base and infinite in another**

Pretty cool, huh?

## The jump to base-b

If instead of base-2 we wanted to convert to base-b, all we have to do is multiply by `b` in each step instead of multiplying by 2. Of course, if you're working on base-b, then digits on that base go from `0` to `b-1`.

But the question comes: Why does this work? Where does this algorithm come from?

It's simple maths. When using base `b`, any fractional part `N` of a number is represented as:

$$
N = a_1b^{-1} + a_2b^{-2} + a_3b^{-3} + a_4b^{-4} + \dots + a_Xb^{-X}
$$

The representation of `N` in base `b` is the sequence of coefficients \\(a_i\\). In the first example, because \\(N = 0.125 = \frac{1}{8} = 2^{-3}\\), we get \\(001\\) for the fractional part: \\(0*2^{-1} + 0*2^{-2} + 1*2^{-3}\\).

So, you want to find out the values of \\(a_1\\), \\(a_2\\), \\(a_3\\), etc. 

What happens if we multiply the base `b` representation of `N` by `b`? We get:

$$
bN = a_1 + a_2b^{-1} + a_3b^{-2} + a_4b^{-3} + \dots
$$

This is beautiful, genial and extremely simple. By doing this, we get an integer \\(a_1\\), which is some number between \\(0\\) and \\(b-1\\), and the rest of the coefficients make up the fractional part.

In other words, the integer part of \\(b \times N\\) is the next coefficient. And we just keep doing this until there is no fractional part anymore, or up to the point where we can see a loop.

## The code

Here's *teh codez*:

```c
char get_numberChar(int base, int n) {
  int a = n + '0';
  if (a >= '0' && a <= '9')
    return a;
  return 'a'+n-10;
}

void to_baseb(int b, float number, char converted[], int lim, int index) {
  int ipart;
  if (index == lim-1 || number == 0) {
    converted[index] = '\0';
    return;
  }
  number *= b;
  ipart = (int) number;
  number -= ipart;
  converted[index] = get_numberChar(b, ipart);
  to_baseb(b, number, converted, lim, index+1);
}

void base_b_converter(int b, float number, char converted[], int lim) {
  if (b > 26 || b <= 0) {
    printf("Invalid base!\n");
    return;
  }
  if (number == 0) {
    converted[0] = '0';
    converted[1] = '\0';
    return;
  }
  to_baseb(b, number, converted, lim, 0);
}
```

And that's how it's done.

Example usage:

```c
int main() {
  char c[100];
  float f = 0.465897654154515;
  base_b_converter(20, f, c, 10);
  printf("%s\n", c);
  return 0;
}
```

This will print `9673c9j09`, which, according to my calculator, is \\(0.4658976555\\).

Unfortunately, we cannot explore this down to the level of pure maths, because of the computer precision problems. Things like \\(\frac{1}{3} = 0.33333333\dots\\) in base-10, which is \\(0.0101010101\dots\\) in base-2, do not yield a loop. The problem is that the more we multiply by 2 and the more integer parts we remove, the smaller the number gets, and even though theoretically it never reaches 0, in a finite bits representation inside a computer, it eventually reaches 0, because it is so small that the computer can't represent it anymore, so we don't get a loop. Still, it's a good exercise. I tried this with \\(0.47\\) as well. \\(0.47\\) is also an infinite fraction in base-2; it is represented as:

$$
0.0111100001010001111010111(0111100001010001111010111)
$$

But, even with `lim = 100` in my implementation, the best I can get is this:

$$
0111100001010001111010111
$$

Which, for the record, is pretty good. According to my calculator, this is \\(0.4699999988\\).

By the way, for the sake of simplicity, the code assumes that `b` is less than \\(36\\), so that each character in the alphabet directly maps to the range `[10..35]`.

And remember, there's no reason to be scared of numbers in other bases.
