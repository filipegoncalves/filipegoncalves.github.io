---
layout: post
title: Shlemiel The Painter
---

Shlemiel is dumb, and so is `strlen()`

-----

Yesterday, I finished reading Joel Spolsky's book: *Joel on Software, And on Diverse and Occasionally Related Matters That Will Prove of Interest to Software Developers, Designers, and Managers, and to Those Who, Whether by Good Fortune or Ill Luck, Work with Them in Some Capacity*.

This book is a spectacular compilation of [Joel's articles](http://www.joelonsoftware.com). It is full of knowledge and stories related to his experience and career since his job at Microsoft. Even though the articles are all online, I highly recommend buying the book, because it's a lot better for your eyes to read, it's tangible, and, as Joel says in the preface:

*The book you hold in your hands is, I hope, a heck of a lot more cohesive than the website, where by cohesive I mean "can be read in the bathtub without fear of electrocution".*

There are some things you will never learn from his articles if you don't buy the book, because you're not going to want to read the same 346 pages in a computer screen with the information scattered all over the place.

So, today, I want to talk about something I read in this fantastic book. I'm going to write some thoughts on the Maths side of the [Shelmiel The Painter](http://discuss.fogcreek.com/techInterview/?cmd=show&ixPost=153) joke. I highly recommend reading [Joel's article on the subject](http://www.joelonsoftware.com/articles/fog0000000319.html) before proceeding. The joke goes like this:

> Shlemiel gets a job as a street painter, painting the dotted lines down the middle of the road. On the first day he takes a can of paint out to the road and finishes 300 yards of the road. "That's pretty good!" says his boss, "you're a fast worker!" and pays him a kopeck.
>
> The next day Shlemiel only gets 150 yards done. "Well, that's not nearly as good as yesterday, but you're still a fast worker. 150 yards is respectable," and pays him a kopeck.
>
> The next day Shlemiel paints 30 yards of the road. "Only 30!" shouts his boss. "That's unacceptable! On the first day you did ten times that much work! What's going on?"
>
>"I can't help it," says Shlemiel. "Every day I get farther and farther away from the paint can!"


Shlemiel is dumb. He always paints until he's out of ink, then he goes all the way back to the paint can, and then he walks everything again up to where he stopped. Just for the fun of it, today I want to show how we can build a function \\(g(j)\\) that outputs the total painted yards by the end of day \\(j\\), since Shlemiel started his job. Of course we need some extra information. Let's assume that Shlemiel walks at a constant rate \\(w\\) yards/sec., paints at a constant rate \\(p\\) yards/sec., after picking up more ink, he can paint \\(f\\) yards, and he works \\(d\\) secs. every day. To simplify, we will assume that the action of picking up more ink from the paint can is the same as going to the paint can and coming back. We will also assume that Shlemiel starts every day from the place where the paint can is, because he must begin his work by refilling. So, putting it all together, we have this:

* Walking rate: \\(w\\) yards/sec.
* Painting rate: \\(p\\) yards/sec.
* Refill distance: \\(f\\) yards
* Working day duration: \\(d\\) secs.

Goal: Find \\(g(j)\\) s.t. \\(g(j)\\) outputs the total yards painted after \\(j\\) days (\\(j \ge 1\\)).

Let's see what happens in day 1. Shlemiel only works for \\(d\\) seconds, so, we have

$$
\frac {f}{p} + \frac {f}{w} + \frac {f}{w} + \frac {f}{p} + \frac {f+f}{w} + \frac {f+f}{w} + \frac {f}{p} + \frac {f+f+f}{w} + \frac {f+f+f}{w} + \dots = d
$$

What's this? This is the time he's spending. First, he takes \\(\frac{f}{p}\\) time to paint \\(f\\) yards.

Then he goes out of ink, and because he already painted \\(f\\) yards, he walks for \\(\frac {f}{w}\\) seconds to go back to the paint can; he refills, and then he walks those \\(f\\) yards again, spending another \\(\frac {f}{w}\\) seconds. Afterwards, he paints \\(f\\) yards again.

So far, he painted \\(f+f\\) yards, so now he needs to refill: he goes back, walks \\(\frac {f+f}{w}\\) yards, refills, and goes back again, walks \\(f+f\\) yards, thus spending another \\(\frac {f+f}{w}\\) seconds. And this keeps happening over and over until eventually \\(d\\) seconds have passed and h goes back home.

What's the pattern here? I think this is easy for everyone to see that, after all, our elapsed time is:

$$
\sum_{i = 1}^{n}{\frac{f}{p} + \frac{2if}{w}}
$$

Because Shlemiel works \\(d\\) seconds per day, then this must hold:

$$
\begin{equation}
\sum_{i = 1}^{n}{\frac{f}{p} + \frac{2if}{w}} = d
\end{equation}
$$

Remember that this is for Day 1. So, after day 1, how many yards did he paint? Notice what we're summing: each term in the sum corresponds to painting \\(f\\) yards, going back to the paint can, refill, and coming back to the same point. So, if we have \\(n\\) terms in our sum, it means we painted \\(n*f\\) yards.

What's the value of \\(n\\)? Well, \\(n\\) is the largest integer (\\(n \in \mathbb{N}\\)) that satisfies the constraint above. So, you solve the equation and you take the largest closest integer to the result, and your answer is \\(f*n\\).

This was for day 1. What about day \\(j\\)? Well, it's basically the same thing, except that Shelmiel gets to walk \\(g(j-1)\\) more yards on every journey, which is the amount of yards he painted in the previous day. The expression is, therefore, quite similar:

$$
\frac{g(j-1)}{w} + \sum_{i=1}^{n}{\frac{f}{p} + \frac{2*(i*f + g(j-1))}{w}}
$$

Note that the term out of the sum is because he starts his day where the paint can is, so he has to walk all the way down to the finishing point of the previous day.

Conveniently, sums are linear, so we can rewrite this as:

$$
\frac{g(j-1)}{w} + \frac{f*(n+1)}{p} + \frac{2*g(j-1)*(n+1)}{w} + \frac{2f}{w}\sum_{i=1}^{n}{i}
$$

When we force the equality to \\(d\\), we get a quadratic equation:

$$
\begin{split}
(f*p)n^2 + (2*p*g(j-1)&+f*(p+w))n+ \\
&(3*p*g(j-1) + w*(f-dp)) = 0
\end{split}
$$

Applying the quadratic formula, you can solve for n:

$$
\begin{split}
n &= \lfloor \frac{-(2pg(j-1) + f(p+w))}{2fp} \\
&+ \frac{\sqrt{(2pg(j-1) + f(p+w))^2 - 4fp(3pg(j-1) + w(f - dp))}}{2fp} \rfloor
\end{split}
$$

Note that I applied the floor function (these fancy little bars around the expression for \\(n\\)), because it makes no sense for \\(n\\) to be a fractional number. Also, we only want the positive solution of the equation: all of our quantities \\(p\\), \\(f\\), \\(w\\), \\(d\\) and \\(g(j-1)\\) are positive, so we're not interested in the case where we subtract the square root instead of adding it.

As you can imagine, I didn't even try to simplify this thing. I didn't simplify it because at this point we're pretty much done. We have defined a recursive expression that is able to tell us how many yards will be painted by the end of day \\(j\\).

Because \\(n\\) depends on \\(g(j-1)\\), we should represent this quantity as a function as well. I'll call it \\(y\\). \\(y(j)\\) represents how many yards Shlemiel painted on day \\(j\\). We can define it as follows:

$$
\begin{split}
y(j) &= f * \lfloor \frac{-(2 p g(j-1) + f(p+w))}{2fp} \\
&+ \frac{\sqrt{(2 p g(j-1) + f(p+w))^2 - 4 f p (3 p g(j-1) + w(f - dp))}}{2fp} \rfloor
\end{split}
$$

This is nothing more than our original \\(n\\) expression multiplied by \\(f\\), because as we saw, when there are \\(n\\) elements in a sum, we paint \\(f\\) yards.

We're almost done with our final expression. All we have to do now is to add the number of yards painted in the previous day. So, \\(g(j)\\) is therefore given by:

$$
g(j) = 0 \text{    if } j = 0
$$

$$
g(j) = g(j-1) + y(j) \text{    if } j \ge 1
$$

Don't get confused about \\(y(j)\\), it's just a way for me not to rewrite all of that huge expression in this little box for \\(g(j)\\). It's supposed to simplify.


## Implementation and experimental results

For me, the best way to implement this is using [dynamic programming](http://en.wikipedia.org/wiki/Dynamic_programming). If you don't know what I'm talking about, dynamic programming in this case is just a fancy name for "implement it in such a way that recursive calls never solve the same problem again". In other words, you would allocate a big array where you stored each value for \\(g\\) that you have computed in the past. To calculate \\(g(j)\\), go and see if it's in the table. If it's not, recursively calculate it, and then store the result in the table. This prevents the algorithm from calculating the same value recursively 4 times in one single iteration. This is important, it saves a lot of space and time.

I tested this with some realistic values:

* \\(w = 1.822\\)
* \\(p = 0.109\\)
* \\(f = 5.468\\)
* \\(d = 28800\\)

Here's what I got:

|   Day   | Painted in this day | Total painted |
| :-----: | :-----------------: | :-----------: |
| 1       | 486.652             | 486.652       |
| 2       | 213.252             | 699.904       |
| 3       | 164.04              | 863.944       |
| 4       | 136.7               | 1000.644      |
| 5       | 120.296             | 1120.94       |
| 6       | 109.36              | 1230.3        |
| 7       | 98.424              | 1328.724      |
| 8       | 87.488              | 1416.212      |
| 9       | 82.02               | 1498.232      |
| 10      | 82.02               | 1580.252      |
| 11      | 76.552              | 1656.804      |


Notice that after 11 days, there was a huge loss of productivity, he's got a big BOOM in the first day, where he paints almost 500 yards, and after 11 days he's doing \\(\frac{3}{20}\\) of what he did in the 1st day. So, indeed, it all goes back to the joke baseline. He seems to be such a great worker in the first day, but then in the second he starts losing power, and then ... oh my ... it gets worse and worse.

Now that we deeply analyzed Shlemiel the Painter's joke, it's time for reflecting on the Computer Science side. By this time, you should have a pretty damn bad feeling about Shlemiel the Painter's Algorithms: they can ilude you within the first steps, but when you start using them a lot, things will be terribly slow. Really slow. You do NOT want a Shlemiel the Painter's Algorithm running somewhere in your code.

As you can read in Joel's article, typical string concatenations in C are a Shelmiel the Painter's algorithm (or in Java - when you use the overloaded operator `+` to make things like `"hello" + ", world"`, java will copy everything into a new string, and then if you use `+` again, it will copy everything again to a new string - this can happen in other languages, so you can't run away saying "I use language X or Y, so I don't have to worry about it", because you *do* have to worry about it, and you'll not be using the same language for the rest of your life anyway, so that's a doomed argument anyway).

In his article, Joel shows a sample piece of code where he concatenates a set of strings. Analyzing the complexity of this code should be pretty straightforward by now. Imagine you have a set of \\(m\\) strings, each one with at most \\(n\\) characters. What's the complexity of concatenating them all? Try to solve it by yourself, it's a nice exercise.

Let's see a few examples before determining a general pattern. When we concatenate string 2 to string 1, we have to run through all of the \\(n\\) characters of `s1` and the \\(n\\) characters of `s2`:

$$
n + n = 2n
$$

When we concatenate string 3, we have to traverse the previous string, which has now \\(2n\\) characters, plus the \\(n\\) of `s3`, so:

$$
2n + n = 3n
$$

It can be seen that when we concatenate string \\(i\\), we have to run through this number of characters:

$$
i*n
$$

We are going to concatenate \\(m\\) strings (in fact, we concatenate \\(m-1\\), but it's not significant, we're making an asymptotic analysis), so, we will be running through

$$
\sum_{i=1}^{m}{i*n}
$$

characters, which is basically

$$
n \frac{m(m+1)}{2}
$$

meaning, this is \\(\mathcal{O}(m^2)\\) (note that \\(n\\) is constant, that's why I took it away). It's bad. We can do it in linear time (again, read Joel's article).

I decided to make this article as a response to the challenge Joel left on his book when he talks about Shelmiel the Painter's Algorithms. By the end of page 7, you can read:

> For extra credit, what are the real numbers?


This is my answer!
