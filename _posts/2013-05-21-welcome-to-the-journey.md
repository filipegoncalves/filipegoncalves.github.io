---
layout: post
title: Welcome to the journey
---

A welcome message to my readers

-----

Programming is an art. Code is poetry. Just think about what happens when you write a computer program. You write it, some compiler parses it and transforms it in a bunch of CPU assembly instructions, which in turn will give birth to a set of binary instructions known as opcodes. Your program executes. Maybe it works as expected, maybe it doesn't. Nevertheless, you have just created something new. You *produced* something.

Hold on....

Yeah! You just CREATED something that did not exist a few minutes ago. That is the closest feeling you will ever have in your entire life to being God. The art of computer programming (haha, get it? I mean, the book reference...) allows you to create whatever you want, when you want, and how you want. You can do it in a creative, clean way, or you can obfuscate it. You can do it in a dumb way, or you can do it the smart way. It's your call.

The computer is always there for you, it doesn't need to eat, drink and sleep. It doesn't demand a high salary, it doesn't care about weekends. It's just there, waiting for more work to come.

Ok. That was cool. That was one of those ZEN moments where I go all nuts about computers, bits, bytes, programming, cpus, memories, whatever. Why on earth am I writing this? Why does this website exist?

Although the computer world is full of beautiful and interesting stuff for you to explore, it can be a very tiring activity, challenging and tricking your brain over and over again. I cannot hide the fact that human beings often have this tendency to write a piece of code in a dumb way. In fact, I can see some kind of pattern that could even be described by some sort of fluxogram, or state machine. It goes like this: you write the code without really caring too much or without understanding what you're doing, expecting it to work. And then KABOOM. It doesn't work. Or it works, but when you try it in your laptop, it doesn't. Or it works well sometimes, but other times something really weird happens and you have to keep trying it over and over again until it finally works. Or it's too slow. Or it eats all of your memory. A lot can go wrong. And what do you do? It depends. There are 3 possible solutions that come to your mind:

* It's good enough
* Let's try to fix it
* Let's start over

For obvious reasons, I will not even comment the first one. Number 3 might be a good choice if your first attempt consisted of writing some quick, throwaway code that would never work.

And then you have number 2. Number 2 sounds good. Fixing errors is always good, right? The problem is, "trying to fix it" can also be done in a smart way, or in a stupid way.

Until this exact moment, throughout my short programmer life, I've seen the most incredible, dumb ways to debug programs. Imagination has no limits, and you will just not believe your freaking eyes when you see how someone actually "got it to work". One of the worst ways to do it is by trial and error programming. *TRIAL AND ERROR IS NOT THE SOLUTION*.

Let's go through a small example:

Someone comes at me and complains about a warning when compiling a C program. It was *something about [...] makes integer from pointer without a cast*.

In this example, trial and error basically means that this programmer starts adding `*` and `&` everywhere in the program in a desperate way to stop gcc from spitting out this warning.

If that's the spirit, it would be better to simply ignore the warning, because at least that would give other experienced programmers a chance of *understanding* what the code was trying to accomplish in the first place. Otherwise, we end up with code that compiles and gives no more warnings, but no one really understands why it is coded that way, and what on earth the programmer was thinking. And then you ask him.

*It's the only way that it compiles*

There you go - when you hear an answer like this, you're talking to a trial and error programmer. Beware!

Where was I? Oh, right, why this blog exists...

I've been taught that software correctness is the most important aspect of software development. And I don't think it is very hard to believe me when I tell you that trial and error programming will not lead to correct software.

You want software to be correct. No, you do not want it to be efficient. First of all you want it to *work*. When someone comes at you because their code is not producing expected output and asks you how your program works to solve problem `X` or `Y`, and you start describing your first attempt, and then suddenly he tells you that your program is really slow and that they can do it 1000 times faster, it's time for you to say *Yeah. But mine works. Yours doesn't.*

Well, in fact, if it doesn't work, I could do it even faster by not computing anything at all, right? So, there's not much to be proud of if the thing is really efficient but does not produce the expected result.

To produce correct software means that you must know what you're doing. And if you want to know what you're doing, you have to learn. This blog will be my little corner to dump my thoughts and share with you what I'm learning.

Computers world is full of endless cool things to learn and try. There is always something else that you can make.

With this website, my goal is to share my knowledge with fellow programmers that are as interested and dedicated to this as I am, and who enjoy doing awesome programs in the most clean, understandable way and like to know what they're doing.

Typical readers should be familiar with some basic low-level computer knowledge. I will not cover basic topics, I will simply assume you're literate enough to read my posts. I will try to cover lots of different problematic code cases and introduce you to weird, uncommon behaviour that may occur and you would never have a single clue on why it happens if you didn't know what is going on under the hood.

Let's start with this welcome post by introducing a small and apparently innocent little piece of code:

```c
int abs(int n) {
  if (n < 0)
    return -n;
  return n;
}

int main1(void) {
  /* .... */
  x = abs(i);
  /* From now on, x >= 0... */
  ...
}

int main2(void) {
  /* .... */
  x = abs(i);
  if (x >= 0) {
    /* do some work ... */
  }
  else {
    /* This should never happen */
    perror("Negative absolute value");
  }
}
```

Now, the question is: which one is better? `main1()` or `main2()`?

I know that most people will tell me that `main2()` looks better, because it makes sure that `x` is really positive. The thing is, although most people can recognize this, they would actually never code is that way because, hey, `abs(x)` is always positive. And similar assumptions will be made on the whole program. It turns out that most of these "never happens" cases do, in fact, happen. You must be trained and understand that you cannot assume that something never happens. So what's wrong with this particular case?

Did you find it? No? Here's a hint: bugs usually show up in boundary conditions.

Assuming a 2's complement representation, what happens if `n` is `INT_MIN`? Typically, `INT_MIN` is `-(INT_MAX+1)` (although technically you cannot assume that - signed integer overflow is undefined behavior). So, when you compute `-n`, in a 2's complement notation, you would need another extra bit because `-INT_MIN` would be `INT_MAX+1`. But you can't represent `INT_MAX+1`!

An example is often useful: imagine we're using 8 bits ints in a 2's complement notation. `INT_MAX` would be `01111111`, and `INT_MIN` would be `10000000`. With 8 bits, `-INT_MIN` will be `10000000` again (you'd need a 9th bit).

And voila. `abs()` returns a negative value and `main2()` is prepared to handle it, `main1()` would probably crash the whole application in a mysterious, ugly way, with some random garbage-fatal-unknown-error message displayed to the poor user.

And if the programmer doesn't know what is going on down there on the CPU, he will *never* really understand why `abs()` is returning a negative value. He will never make it to the next level of producing portable, robust code, unless he dives deeply into this low-level CPU stuff. Otherwise, he will just hide behind crappy ignorant excuses like saying that `abs()` is buggy or he will eventually end up ignoring the error because it doesn't happen *that* often. This is not an academic example, I will give you many more examples like this in future posts.

As a final note, how would you solve it? It depends on what you'd use the absolute value for. You could do it like I showed you in `main2()`, or you could create a special case for `INT_MIN` in your code (however, you must be careful with these "special" cases - lots of times there are more special cases than you think). Or you could figure out some way of not needing the absolute value of a number in your computations. Alternatives are endless, and obviously there is not *a* way to fix it.

It takes a lot of effort to make good code. It's a huge learning process, and you have to live with it. Remember, it's your call.

Welcome to the journey!
