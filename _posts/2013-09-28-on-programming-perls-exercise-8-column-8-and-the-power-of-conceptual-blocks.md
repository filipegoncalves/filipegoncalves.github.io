---
layout: post
title: On Programming Pearls exercise 8, Column 8, and the power of conceptual blocks
---

Haaaah, conceptual blocks..!

-----

It happens to everyone: you are given a problem, and you don't come up with a solution very quickly. In fact, you think, and think, and keep thinking, and you come up with nothing. You realize you're stuck, and there's this ticking clock in the back of your head that makes you feel dumber the more it ticks. Time passes. You wonder if you've got some serious mental illness that stops you from solving this problem. You think you're stupid. Don't laugh, I know you just laughed at this, and you know why? Because it happened to you before, but it's not happening to you *right now*, but you're sure it's happening to somebody else in this planet, and honestly, you don't give a damn about that person and you're obviously in a superior position. So you laugh at our poor little fellow. But let's go back to our story mainline. So, you're stuck, you don't get anything done, you're obsessed with this problem because you *have* to find some solution.

And that keeps on going for days....

and days....

until you finally get it. And you shout "Eureka!", just like ancient Greek scholar Archimedes when he stepped into a bath and noticed that the water level rose. Ok, maybe you don't. I'm not implying you don't shout, I'm saying that you don't shout the specific word "Eureka". But if you have a strong passion for what you're doing and this is important for you, you will shout a bunch of happy words no matter what. And then you're the happiest person ever. You're the best, you solved it. And it WORKS. What happens next? Well, the cycle begins again, of course, with another problem. It's life.

A problem that is trivial for someone can be complex for another person. And a problem may be very easy to the latter person and extremely difficult for the former. Why is that? Why does the same problem can be hard for someone and easy for others? Is it possible to somehow classify problems into a hierarchical tree? 

Jon Bentley briefly discusses these issues in his amazing book Programming Pearls, in Column 1 and Column 2. Column 2 shows the importance of designing good algorithms and thinking wisely and calmly before rushing into code. An extremely good example is the vector rotation algorithm. The problem asks you to develop an algorithm to rotate a one-dimensional vector of `n` elements left by `i` positions. For instance, with `n = 8` and `i = 3`, the vector `abcdefgh` is rotated to `defghabc`. In this column, the best and simplest algorithm to do the task is reversing block `A`, then reversing block `B`, and then reversing the whole vector, where `A` is the block of letters going from `0` to `i-1`, and `B` is the block of letters going from `i` to `n-1`. The problem looked hard until a simple, easy to understand algorithm was discovered.

Jon concludes that a hard problem may have an easy solution, and he calls it the "Aha!" insight (that's why the column's title is "Aha! Algorithms"). It must have happened to you as well. The *aha!* insight is the moment you realize how much of a jerk you were being for not finding that solution. 

So, what was keeping you from finding such a great solution? This is a vague question with a vague answer. There are inumerous factors. Normally, it all comes down to one important concept: *conceptual blocks*.

Huhu. That's right. Conceptual blocks. Where did I get that from? Column 1 in Programming Pearls, 2nd edition, introduces this concept as part of the "Further Reading" section. It's the history of a programmer that had to write a program to sort a set of at most 10 million positive integers using only 1 MB of memory, and it couldn't take longer than 10 seconds (and in fact, it should be as fast as possible, because the user had to wait for this sorting operation to finish before resuming work in the system). The programmer immediately assumed that he couldn't fit 10 million integers in 1 MB of memory, so he asked Jon how to sort a disk file. Sorting disk files is a pain in the ass because you can't run away from a multi pass algorithm. When Jon asked him *why* he wanted to do that, things began to look a little less scary, and it took him about 15 minutes to come up with an algorithm whose only bottleneck are I/O operations and that stored every 10 million integer in memory with 1 MB (want to know how? Read the book!). His comments on this matter are very elucidative:

> [...]
> We finally solved his problem by breaking through his conceptual block and solving an easier problem. Conceptual Blockbusting by James L. Adams (the third edition was published by Perseus in 1986) studies this kind of leap and is generally a pleasant prod towards more creative thinking. Although it was not written with programmers in mind, many of its lessons are particularly appropriate for programming problems. Adams defines conceptual blocks as "mental walls that block the problem-solver from correctly perceiving a problem or conceiving its solution" [...]

Right? Did you see that? *mental walls that block the problem-solver from correctly perceiving a problem or conceiving its solution*. Hum hum. Sounds familiar. Been there, done that. Like I said, happens to everyone.

Today, I want to discuss an exercise that blocked me for a while. It was exercise 8 from Column 8. It's about the [maximum subarray problem](http://en.wikipedia.org/wiki/Maximum_subarray_problem) (DON'T read about Kadane's algorithm if you want to keep your head clean for this article - just read the problem description): given an array of `n` integers, find the maximum sum in any contiguous subarray. With `n = 10` and a sample array like this:

```c
int x[] = { 31, -41, 59, 26, -53, 58, 97, -93, -23, 84 };
```

The expected result is 187 (sum from `x[2]` up to `x[6]`).

Now, I know that there's an easy to implement linear algorithm to solve this problem. But stick with me for a minute, will you? There are important lessons to be learned from here. Let's forget, for a few moments, that someone already found a linear algorithm. In Column 8, Jon presents several algorithms that solve this, one of them is `O(n log(n))`. It's a divide-and-conquer algorithm: for an array of size `n`, split it in two arrays `x[0..n/2]` and `y[n/2+1..n-1]`, and recursively compute the maximum subarray sum for `x` and `y`. The maximum subarray sum for the original array is either the maximum subarray sum in `x`, the maximum subarray sum in `y`, or it crosses the border between `x` and `y` (crosses position `n/2`). In that case, the recursive calls couldn't find it because we handed them isolated pockets of the original array, so we must find the maximum subarray that ends in position `n/2`, the maximum subarray that starts in position `n/2+1`, and sum it up. If this is greater than the sums for `x` and `y`, we return it, otherwise we return the greatest sum value between `x` and `y`. The recursion stops as soon as an array's lower bound equals it's upper bound. The maximum subarray sum for an empty array or an array with every element less than 0 is 0.

Finding the maximum subarray that crosses the border between `x` and `y` is the equivalent of a merge step in mergesort. Because we have to test every possible subarray that ends in position `n/2` (and the same for the right side, where we test every possible subarray that begins in position `n/2+1`), each recursive call uses \\(\mathcal{O}(n)\\) time. There are \\(\mathcal{O}(log(n))\\) recursive calls, so the overall algorithm runs in \\(\mathcal{O}(n log(n))\\) time. To help you further understand the algorithm before diving into the exercise, here's a short quote from section 8.3 explaining it pictorially:

![Max Subarray Example]({{ site.baseurl }}{{ site.assets }}max_subarray.png)

The pseudo-code for this algorithm is described below, taken from page 80:

```c
float maxsum3(l, u)
	if (l > u) /* zero elements */
		return 0
	if (l == u) /* one element */
		return max(0, x[l])
	m = (l + u) / 2
	/* find max crossing to left */
	lmax = sum = 0
	for (i = m; i >= l; i--)
		sum += x[i]
		lmax = max(lmax, sum)
	/* find max crossing to right */
	rmax = sum = 0
	for (i = (m, u]
		sum += x[i]
		rmax = max(rmax, sum)
	
	return max(lmax+rmax, maxsum3(l, m), maxsum3(m+l, u))
```

After this brief introduction, here's the exercise I'm going to talk about:

> 8.
>
> Modify Algorithm 3 (the divide-and-conquer algorithm) to run in linear worst-case time.


Bam. There. Stuck. No ideas on how to improve it. This is actually worse than it looks: because I read the entire column before looking at the exercises, I already knew the extremely simple, not recursive, linear solution. That means I was completely misguided as to where I should turn into to make this recursive approach \\(\mathcal{O}(n)\\). For that matter, I immediately realized that I was being a victim of a HUGE mental wall, a conceptual block that would haunt me for days.

I tried to think hard about it, but I wouldn't forget the simple linear solution. Damn, why did I read that before seeing this exercise? This kind of conceptual block is very frustrating. This is why being brain damaged about something is so bad. You're so blind and brainwashed that you can't think reasonably about alternative perspectives on the same problem. And it freaks the hell out of me! Before moving on, I'm going to give you a chance to do better than I did (that's why I asked you not to read about Kadane's algorithm in wikipedia). How would you improve it so that it is linear?

After struggling for a while, I began by observing that the merge step must be \\(\mathcal{O}(1)\\). Every recursive call together, as a whole, will iterate through every element in the array (ideally, just once). So we have to drop that fateful idea of calculating the maximum subarray that crosses the border. For some reason that I can't seem to understand, I just thought that a good approach would be to make recursive calls return not only the maximum subarray sum, but also the begin and end positions of the maximum subarray. So, for this array (the same example as before):

```c
int x[] = { 31, -41, 59, 26, -53, 58, 97, -93, -23, 84 };
```

Not only I'd know that the maximum subarray sum is 187, I would also know that it's bounded by positions 2 and 6. *Hmmmmmm...*, you mumble, *Why is that useful?*. I have to be honest with you here: *I have no freaking idea*.

This is really bad. And sad. It bothers me a lot. It was a tremendous mental wall that I couldn't overcome. God damn it! I don't know why, but I just kept thinking that I would need to return the positions that bound the maximum sum because it would be useful, but I didn't know exactly what for.

This is actually bizarre, maybe you don't believe it, but I was stuck for about 2 days. And it really pissed me off. Next thing I know, I was always thinking about this problem. I went to bed thinking about it, I was driving and thinking about it. I was watching a TV program about the reconstruction of the World Trade Center site *and I was thinking about the problem*, secretly, with a background thread in my head (have you heard about WTC reconstruction? It's the tallest building in lower Manhattan at the moment.). I was obsessed. Well, not totally obsessed though: at least I wasn't thinking about it when I was working out. But it never came to that point anyway. For me, it's impossible to think about anything when I'm working out. I just think about doing exercise.

You know, sometimes the best thing you can do when you're stuck with a problem is to simply *let it go* for a while. Just have some rest. Take a nap. Go ride your bike. Play a video game. *Do something else*. Let your head stop thinking about it.

This is very hard for me. Seriously, it is. It's a hell of a problem. The only place where I can stop thinking about something that is bothering me is when I go to the gym. In the gym, I basically kill -9 every process that is stuck in an algorithmic problem. End of it.

Anyway, let's get back to business. My initial approach of trying to return the maximum sum positions was erratic, but at least I was on the right track: I couldn't make it \\(\mathcal{O}(n)\\) without returning additional information. The key to success, and I really don't know why I didn't come up with this earlier, is to think *why* the algorithm is \\(\mathcal{O}(n log(n))\\). It's \\(\mathcal{O}(n log(n))\\) because every recursive call is doing \\(\mathcal{O}(n)\\) work by figuring out the maximum subarray that crosses the middle position. Well, then let's stop doing that. But wait, we need to somehow, somewhere, have that information. It's crucial to get the right answer. 

Stay with me, this is getting really interesting.

What information do we need to compute the maximum subarray sum that crosses the middle position? Let's call the middle position `m`. Well, we need to know the maximum subarray that *ends* in `m` - let's call it `x'` - and we need to know the maximum subarray that *begins* in `m+1` - let's call it `y'`. If we know that, then we immediately know in \\(\mathcal{O}(1)\\) time that the maximum subarray sum that crosses position `m` is given by `x'+y'`. Maybe it's better to see it as a diagram:

![Max Subarray Explanation]({{ site.baseurl }}{{ site.assets }}max_subarray2.png)

So yeah, basically, if recursion can tell us `x'` and `y'`, we calculate `x'+y'` in constant time, and we get an overall \\(\mathcal{O}(n)\\) algorithm. But hold on for a second, recursion can't just do that magically. Base cases are easy, but how will we get the new values for `x'` and `y'` to pass to the upper recursive call? In other words, given `x'` and `y'`, how can we use that information to calculate the maximum sum ending or starting at some position in our current array?

The problem goes a little deeper than that: how do we know whether we should return the maximum sum ending in position `u` (the upper position, as you can see in the pseudo-code), or the maximum sum starting in position `l`? You see, if we're a recursive call for the right side of a bigger array, our `l` position is the upper caller's `m+1` position, so we want to pass it the maximum subarray sum beginning in position `l`, but on the other hand, if we're a recursive call for the left side of a bigger array, our `u` position is the upper caller's `m` position, so we want to pass it the maximum subarray sum ending in position `u`. What do we do? This is getting extremely complex.

Don't even think about creating special conditions for each side of the recursion. A smart, simple approach will work both ways. We need to define a structure to hold several information that will be passed to the upper caller. Let's call this structure the `solution` structure. For any recursive call, this structure will hold the maximum sum *beginning* in position `l` (from now on, refered to as `maxl`), and the maximum sum *ending* in position `u` (from now on, refered to as `maxu`). The structure will hold both of these values; it's up to the upper caller to decide which one to use. Besides, the structure will also have to hold the actual maximum sum found in the whole array, this will be called `maxsum`. `maxsum` can be the same as either `maxu`, `maxl`, or any other value for the case that the maximum sum does not belong to a subarray that begins at position `l` or ends at position `u`.

There's just a small little detail left behind. We're going to need to store the total sum of the whole array as well. This is necessary for the upper caller to figure out his values for `maxl` and `maxu`. In the upper caller, the proper way to do it is: my `maxl` value is the same as the left half subarray's `maxl`, unless that whole subarray PLUS the right subarray `maxu` is a better candidate. In other words: Either we choose to extend the left subarray's `maxu`, or we don't. We want to choose the maximum value for `maxu`, so we need to know if we can extend it. Well, `maxu` is already the best value for the left subarray, so if there's any chance to extend that to a better value, the only way to do it is by extending it up to the end of that subarray and adding in a few elements from the right subarray (which wasn't "seen" by the left subarray recursive call). "Adding in a few elements from the right" is optimal if we choose the right subarray `maxl` value, so that's what we'll be adding. And now, we compare this extension to the unextended, inherited `maxl` value, and we choose the best. It works the other way around for the right side.

The base cases for the recursion are easy to code, they occur only when an empty array or a one-element array is reached. In the former case, every value is set to 0 by definition, in the latter, every value is set to 0 except for the total sum value, which is the same as the array's element value.

The implementation avoids the use of dynamic memory allocation, so a pointer to the structure is passed by the upper caller to the recursive leafs.

```c
#include <stdio.h>
#define max(a,b) ((a) > (b) ? (a) : (b))

struct solution {
	int maxl;
	int maxu;
	int total;
	int maxsum;
};

int main() {
	int maxsum(int [], int);
	int x[] = {31, -41, 59, 26, -53, 58, 97, -93, -23, 84};
	printf("%d\n", maxsum(x, (int) sizeof(x)/sizeof(x[0])));
	return 0;
}

void maxsum_aux(int [], int, int, struct solution *);
int maxsum(int x[], int n) {
	struct solution s;
	maxsum_aux(x, 0, n-1, &s);
	return s.maxsum;
}

void maxsum_aux(int x[], int l, int u, struct solution *s) {
	struct solution left;
	struct solution right;
	int m;
	if (l >= u) {
		s->maxsum = s->maxu = s->maxl = (l > u ? 0 : max(x[l], 0));
		s->total = (l > u ? 0 : x[l]);
		return;
	}
	m = (l+u)/2;
	maxsum_aux(x, l, m, &left);
	maxsum_aux(x, m+1, u, &right);
	s->maxl = max(left.total+right.maxl, left.maxl);
	s->maxu = max(left.maxu+right.total, right.maxu);
	s->total = left.total + right.total;
	s->maxsum = max(max(left.maxsum, right.maxsum), left.maxu+right.maxl);
}
```

Phew! That was hard to come out.

You can now go and read the wikipedia page about the linear, simple, non-recursive solution. It's way better. That's what stopped me from reaching this solution. I've had serious difficulties overcoming this mental wall, but eventually I did it. And damn it, *I'm buying that book about conceptual blockbusting*. It can't magically blow away every conceptual block, but at least we get the chance to know why, how and when they show up, and possible shortcuts to avoid them.

It is important and recomforting to know that conceptual blocks can strike anyone. Even the best coders can get nailed and blocked in a subtle problem. This is described in section 8.6:

> The history of the problem sheds light on the algorithm design techniques. The problem arose in a pattern-matching problem faced by Ulf Grenander at Brown University; the original problem was in the two-dimensional form described in Problem 13. In that version, the maximum-sum subarray was the maximum likelihood estimator of a certain kind of pattern in a digitized picture. Because the two-dimensional problem required too much time to solve, Grenander simplified it to one dimension to gain insight into its structure.

Note that the key idea here was to turn a 2-dimensional problem into a single dimension problem. As you can see, algorithm design techniques can span through multiple levels of abstraction, and it is important to be open minded about every single detail of the design. An efficient algorithm may come up only when you conceive and structure the problem in a different way, or look at it from a different perspective. The *aha!* insight depends a lot on that. And then, of course, something that is obvious to a person may not be obvious to you:

> Grenander observed that the cubic time of Algorithm 1 was prohibitively slow, and derived Algorithm 2. In 1977 he described the problem to Michael Shamos, who overnight designed Algorithm 3. When Shamos showed me the problem shortly thereafter, we thought that it was probably the best possible; researchers had just shown that several similar problems require time proportional to n log(n). A few days later Shamos described the problem and its history at a Carnegie Mellon seminar attended by statistician Jay Kadane, who sketched Algorithm 4 within a minute.

Did you read that carefully? Especially the final sentence: *[...] who sketched Algorithm 4 within a minute*. Impressing! Notice, if you will, that even great programmers like Jon and others were mentally attached to the `n log(n)` algorithm, and pretty much convinced themselves that it was the best solution, and it turns out it wasn't. It only took a minute for someone with a fresh, cleared, undamaged perspective on this problem to find a better solution. And now, it is utterly trivial for everyone to understand how the best algorithm works, but the truth is that no one came up with it for quite a while. Guess what, when Kardane found this solution, I am totally sure that Shamos and Jon felt a little bit stupid and gave themselves a little mental slap.

> Yeah! Of course! Obviously. That's a very good solution, why didn't I think about that before?

I'm sure everyone felt this once upon a time, and sometimes there are mental blocks that we never overcome. We just have to live with it and accept the fact that there is always a limit.
