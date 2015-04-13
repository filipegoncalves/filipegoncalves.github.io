---
layout: post
title: Some thoughts on Programming Pearls exercises 9 and 10, column 8
---

Further thoughts on the maximum subarray problem

-----

As you probably had the chance to notice, I am reading Programming Pearls, 2nd Edition, by Jon Bentley. It's an amazing book that every software developer, or everyone aspiring to be one, should definitely read.

Today, I am writing a concise, brief article about exercises 9 and 10 in Column 8. An article to the point, no more, no less.

This is, of course, related to my previous article, and you should read it to know what I'm talking about. And, for that matter, you should grab your copy of Programming Pearls and read Column 8 before reading on (and, of course, after reading Column 8 and this article, go and read the rest of the book).

My previous article was about exercise 8, where we are asked to turn an \\(\mathcal{O}(n log(n))\\) divide-and-conquer algorithm to compute the maximum subarray sum into an \\(\mathcal{O}(n)\\) algorithm. It was a great challenge to which I found an elegant solution. Anyway, enough chit-chat. I want to discuss the following 2 exercises. Exercise 9 from Programming Pearls Column 8 challenges the reader with some variants of the same problem:

> We defined the maximum subvector of an array of negative numbers to be zero, the sum of the empty subvector. Suppose that we had instead defined the maximum subvector to be the value of the largest element; how would you change the various programs?

If you read Column 8, or if you read wikipedia's page about the maximum subarray sum problem, you probably know by this time that there is a known \\(\mathcal{O}(n)\\) algorithm to solve the problem. Here's some pseudo-code:

```
maxsofar = 0
maxendinghere = 0
for i = [0, n)
    /* invariant: maxendinghere and maxsofar
       are accurate for x[0..i-1] */
    maxendinghere = max(maxendinghere + x[i], 0)
    maxsofar = max(maxsofar, maxendinghere)
```

Let's start out by looking at the proposed solution. The solution to exercise 9 has this to say:

> Replace the assignment maxsofar=0 with maxsofar=-infinity. If the use of -infinity bothers you, maxsofar=x[0] does just as well; why?

It is interesting to note that even the best coders writing the best books about algorithms can make mistakes. And I'm guessing the book was revised by the author and someone independent, and still, this error made it all the way up to the bookshop shelves. The problem with this solution is that it doesn't change the overall algorithm behavior. It has no effect. Nothing. Zero.

Well I'm sorry, that's not true. Sorry, nitpickers, you're right. The algorithm changes for the case n = 0. When n = 0, that is, when we have an empty vector, the loop is not executed, and `maxsofar` will hold the initialization value \\(-\infty\\) and, thus, the *only* difference is that we are defining an empty vector to have a maximum subarray sum of \\(-\infty\\), which doesn't make much sense. What is \\(-\infty\\)? That's not even a number, it's a concept. A sum can't be "negative infinity", it must be a well defined number. So yeah, instead of having a negligible effect on the algorithm, it is legit to say that it has a *negative* effect on the algorithm, making it invalid for `n = 0` and leaving it the same for every other case.

Column 4 describes program verification techniques, and it is easy to understand why this solution completely misses the point and fails to solve the problem. If \\(n > 0\\), the loop is executed, and we know that the assignment operation `maxendinghere = max(maxendinghere + x[i], 0)` is executed at least once. After executing this, we are guaranteed to know that `maxendinghere` is greater than or equal to zero. The next instruction, `maxsofar = max(maxsofar, maxendinghere)`, propagates this property down to `maxsofar`, so now we know that both `maxendinghere` and `maxsofar` are greater than or equal to 0, even if it's a negative numbers array. It doesn't matter. That makes the proposed solution incorrect, and that's basically it.

You may be thinking that we're on the right track by initializing `maxsofar` to \\(-\infty\\) and that Jon just forgot to say that `maxendinghere` should be compared to `x[i]` and not 0, like this:

```
maxendinghere = max(maxendinghere + x[i], x[i])
```

While this ensures that, after the loop, `maxsofar` will hold the largest value in an array of negative numbers, it will break all the cases where the array is not solely comprised of negative numbers and the maximum subvector sum starts in a position `j` that is greater than 0. You see, the former assignment `maxendinghere = max(maxendinghere + x[i], 0)` makes sure that the sum is reset to 0 whenever a negative sum is reached. Resetting it to 0 is the same as trying out a new, alternative sum beginning at position `i`, that is, when `max(maxendinghere + x[i], 0) == 0`, or, in other words, `maxendinghere + x[i] < 0`, then we start computing a NEW, fresh sum beginning at position `i`. That's exactly what's happening. So, if we change that to `max(maxendinghere + x[i], x[i])`, instead of starting a new sum at position `i` with initial value 0 (the neutral element for a sum), it kind of starts a biased sum, because it will initialize it to `x[i]` instead of 0, and if `x[i]` is negative, the sum will not "look" as good as it is in reality, and we end up with wrong results.

The problem is that we don't know if the whole vector holds negative numbers only, so we can't decide whether we want `max(maxendinghere + x[i], 0)` or `max(maxendinghere + x[i], x[i])`. How do we solve this?

An obvious solution is to create a special case for negative numbers arrays, by iterating through the whole array once, in the beginning, keeping track of the largest element, and if everything we read was negative, we return the stored maximum, otherwise, we execute the original algorithm. This runs in \\(\mathcal{O}(n)\\), so it sounds about right.

But hold on, we can do better. In reality, we don't need to iterate twice. Once is enough! How? Well, we can do *both* at the same time (that is, keeping track of the largest element and seeing if everything is negative). That's not very hard, is it?

We just need a couple of additional variables: a counter to hold the number of negative elements seen so far - let's call it `neg` - and a variable to hold the largest negative element ever seen, `maxneg`. During the main loop, we just have to increment `neg` every time a negative element is seen, and update `maxneg` accordingly. By the end of the loop, we can determine if it's a negative numbers array in \\(\mathcal{O}(1)\\) time by comparing `neg` to `n` (the array length); if they're equal, we return `maxneg`, otherwise, we return `maxsofar`:

```c
int maxsum(int x[], int n) {
	int neg, maxendinghere, maxsofar, maxneg;
	int i;
	if (n <= 0)
		return 0;
	maxendinghere = maxsofar = neg = 0;
	maxneg = x[0];
	for (i = 0; i < n; i++) {
		if (x[i] < 0) {
			neg++;
			maxneg = (maxneg > 0 ? x[i] : max(maxneg, x[i]));
		}
		maxendinghere = max(maxendinghere+x[i], 0);
		maxsofar = max(maxsofar, maxendinghere);
	}
	return neg == n ? maxneg : maxsofar;
}
```

This may look weird at first:

```c
maxneg = (maxneg > 0 ? x[i] : max(maxneg, x[i]));
```

The test `maxneg > 0` makes sure that the method works properly even if `x[0]` is positive (because `maxneg` was initialized to `x[0]`). And that's pretty much all there is to it. The algorithm is still \\(\mathcal{O}(n)\\) and uses \\(\mathcal{O}(1)\\) memory. Simple and efficient.

What about exercise 10? Let's see it:

> Suppose that we wished to find the subvector with the sum closest to zero rather than that with maximum sum. What is the most efficient algorithm you can design for this task? What algorithm design techniques are applicable? What if we wished to find the subvector with the sum closest to a given real number t?

Wooohoo, this sounds harder. And it is harder. Think about it before reading further, it's an excellent exercise. A word of advice: strive for an \\(\mathcal{O}(n log(n))\\) solution.

What did you come up with?

An obvious solution comes to mind: explicitly testing every possible subarray to find the one for which the absolute value of the sum is minimal if we're looking for the closest sum to 0, or find the one for which `abs(sum-t)` is minimal. There are \\(\mathcal{O}(n^2)\\) subvectors, and computing the sum of a subvector is bounded by \\(\mathcal{O}(n)\\) - you need to iterate through that subvector to sum up all the elements inside. We'll be computing the internal sum for every subvector, so the final algorithm is \\(\mathcal{O}(n^3)\\).

Now, if you're happy with that piece of crap, then shame on you. I don't think any professional programmer who knows what he's doing and who is passionately dedicated will settle for that. So... hey, you there, lazy programmer! Use your head to think about a better solution, I'm not falling for that "it's good enough" crap. Use your head, we're rationally superior to other animals because of our intelligence, we have to use that competitive advantage.

The above algorithm can be turned into \\(\mathcal{O}(n^2)\\) by computing the sum in \\(\mathcal{O}(1)\\) time. If you read Column 8, this should be fresh in your memory, we just need a cumulative array, `cum`, such that `cum[i] = x[0] + x[1] + ... + x[i]`. Building this array takes \\(\mathcal{O}(n)\\) time. But wait a second, if we have a cumulative array, we don't need to test every possible subarray explicitly. First of all, notice that, for any values `l` and `u` such that `l >= 0` and `u <= n-1` (with `n` being the array length), the subvector `x[l..u]` (including positions `l` and `u`) sums to `cum[u] - cum[l-1]`. So, what we're really looking for, is for a pair of values `l` and `u` that minimize `abs(cum[u]-cum[l-1])`. The optimal case is when `cum[u]-cum[l-1] = 0`, or, equivalently, `cum[u] = cum[l-1]`. So, what we're really, really looking for, is a pair of values in `cum` that are very very close to each other. The closer, the better. How do we do that? We can sort `cum` in \\(\mathcal{O}(n log(n))\\). After sorting, elements close to each other will appear sequentially, so we can determine the best match in \\(\mathcal{O}(n)\\) time. The algorithm's run time is bounded by the sorting procedure, which runs in \\(\mathcal{O}(n log(n))\\). The code:

```c
#include <stdlib.h>
#define min(a,b) ((a) > (b) ? (b) : (a))
#define abs(x) ((x) < 0 ? -(x) : (x))

int compare(const void *n1, const void *n2) {
	return *(const int *) n1 - *(const int *) n2;
}

/* This runs in O(n log(n)) because of the sorting */
int closest_zero(int x[], int n) {
	int *cum = malloc(sizeof(int)*n);
	int res;
	int i;
	cum[0] = x[0];
	for (i = 1; i < n; i++)
		cum[i] = cum[i-1] + x[i];
	qsort((void *) cum, (size_t) n, sizeof(int), compare);
	res = abs(cum[0]);
	for (i = 1; i < n; i++)
		res = min(res, abs(cum[i] - cum[i-1]));
	return res;
}
```

What about finding the closest sum to `t`?

> Same thing, except that we choose the pair for which cum[u]-cum[l-1]-t is minimal

, you say. Huhu. Yeah. That's right. In theory, that's really fancy to say, but in practice, it is not trivial how to do it efficiently. Notice, if you will, that sorting `cum` was just a quick way to find similar elements, but now we're not trying to find similar elements. We're trying to find elements that *when subtracted* are close to t. They can be really, really far from each other. We're going to have to find some other approach here. What do you suggest?

Did the \\(\mathcal{O}(n^2)\\) solution jumped into your head again? Likewise. Did you find an \\(\mathcal{O}(n)\\) solution? Email me IMMEDIATELY! And the rest of the world too: there probably is no linear solution, but you can always try. Small hint: my solution runs in \\(\mathcal{O}(n log(n))\\).

So, any thoughts?

What if we had a quick way to find a given element in `cum`? Imagine we're scanning `cum`. We're currently in position `i`. Well, we want the best match that makes `cum[i]-cum[j]` very close to `t`, with `j < i`. The optimal case is when `cum[i]-cum[j] = t`, which is the same as `cum[i] - t = cum[j]`. What can we do? Since we're in position `i`, we can start by figuring out what would be the optimal value to find. In other words, we can say "I'm currently `X` units away from `t`, is there anyone worth `X`, or close to `X` who can help me reach `t`?". This is the same as saying that we want to find an element `j` such that `cum[j]` is equal to `cum[i] - t`. Or if there's no such element, we are happy with the closest element to `cum[i] - t`, we have to live with the fact that there is no one that really puts us in `t`, so we use someone who get us as close as possible. How do we find a given element in an array quickly? We can't use sequential search, that's \\(\mathcal{O}(n)\\), and we'll be doing `n` of these, so the resulting solution would be \\(\mathcal{O}(n^2)\\). But we can sort it in \\(\mathcal{O}(n log(n))\\) time and then find an element in \\(\mathcal{O}(log(n))\\) with binary search. But for this problem, we can't use a normal binary search, we have to develop a modified version that returns the closest match to the element being searched.

So, putting it all together, we know that the algorithm will:

* Build the cumulative array, `cum`
* Sort it
* For each value `cum[i]`, use a modified binary search to return the closest value to `cum[i] - t` in `cum`
* Store the best pair seen so far (the best pair is the one for which `abs(cum[i]-cum[j]-t)` is minimal)

Sorting needs \\(\mathcal{O}(n log(n))\\) time, and after that we'll be doing `n` binary searches, so the final algorithm is \\(\mathcal{O}(n log(n))\\). There's just a slight little detail that needs to be cleared: remember that step 3 is looking for the value `j` such that `cum[i]-cum[j] = t`, or close to it, but `j` must be lower than `i`. However, after sorting `cum`, we don't know where each element was originally, so we don't know if the value returned by the binary search was originally in a position below or above `cum[i]`.

What if we knew? Imagine, without loss of generality, that the binary search returned a value `v` that was originally in a position `j > i`. Then, by the definition of `cum`, `cum[i]-cum[j]` is just the negative of the sum of the subvector `x[i+1..j]`, since `cum[i]-cum[j] = -(cum[j]-cum[i]) = -(sum_of(x[i+1..j]))`. If that is the case, then the associated value is just the symmetric of `cum[i]-cum[j]`, so we will take the symmetric before comparing it to the best match found so far. You may be wondering at this time if that is really necessary. For any value `t` different from 0, it is, because `n` and `-n` are not equidistant from `t` (except when `t = 0;` in that case, we could drop that verification).

Ok, that's cool. So what? Well, the cumulative array will have to be a little more complex to allow us to know each element's original position. Therefore, each element in `cum` will be a structure that holds the value for that position, and the original index where it was in the array before sorting. Here's the structure definition:

```c
struct cumelement {
	int index;
	int value;
};
```

An array of `cumelement` is created, and each position `i` is initialized to a `cumelement` with `index = i` and `value = x[0] + x[1] + ... + x[i]`. The rest of the algorithm remains the same; the sorting procedure will use `cum[i].value` as a comparison value.

And now, the implementation:

```c
#include <stdio.h>
#include <stdlib.h>
#define min(a,b) ((a) > (b) ? (b) : (a))
#define abs(x) ((x) < 0 ? -(x) : (x))
#define closest(t, x, y) (abs((x)-(t)) < abs((y)-(t)) ? (x) : (y))
#define max(a,b) ((a) > (b) ? (a) : (b))

/* This runs in O(n log(n)), although it tends to be more like O(2n log(n)) because we sort and then we make n binary searches */
int closest_to(int x[], int n, int t) {
	struct cumelement *search(struct cumelement *, int, int);
	int bestvalue = 0, needed, i;
	struct cumelement *cum = (struct cumelement *) malloc(sizeof(struct cumelement)*n);
	struct cumelement *bestmatch;
	cum[0].value = x[0];
	cum[0].index = 0;
	for (i = 1; i < n; i++) {
		cum[i].value = cum[i-1].value + x[i];
		cum[i].index = i;
	}
	qsort((void *) cum, (size_t) n, sizeof(struct cumelement), compare_elements);
	for (i = 0; i < n; i++) {
		needed = cum[i].value - t;
		if (needed == 0)
			return cum[i].value;
		bestmatch = search(cum, n, needed);
		bestvalue = closest(t, bestvalue, (bestmatch->index > cum[i].index ? bestmatch->value - cum[i].value : cum[i].value - bestmatch->value));
	}
	return bestvalue;
}

/* Modified binary search. Returns pointer to array element closest to v */
struct cumelement *search(struct cumelement *x, int n, int v) {
	int l = -1, u = n;
	int m;
	while (l+1 != u) {
		/* invariant: x[l] < v <= x[u] */
		m = (l+u)/2;
		if (x[m].value < v)
			l = m;
		else
			u = m;
	}
	/* assert: l+1 == u && x[l] < v <= x[u] */
	if (u >= n)
		return x+n-1;
	else if (l == -1)
		return x;
	else
		return x+l+(closest(v, x[l].value, x[l+1].value) == x[l+1].value);
}
```

Understanding the modified binary search can be tricky. Program verification techniques were used to make sure it never fails. We start with a lower index, `l`, initialized to `-1`, assuming that `array[-1]` is \\(-\infty\\), and an upper index, `u`, initialized to `n`, assuming that `array[u]` is \\(+\infty\\). Special careful must be taken to avoid accessing `array[-1]` and `array[n]`, but these initialization values make sure that the loop invariant is always true.

A loop invariant is a condition that is true when a loop iteration starts, and is true when the current iteration finishes. It is the programmer's job to make sure that the loop invariant is kept across iterations. If the programmer did his job correctly, when the loop stops, the invariant will remain true. In the above binary search code, we always keep the property that `x[l] < v <= x[u]`. Thus, we know that `l` and `u` are such that `x[l] < v <= x[u]`, and we also know that `l+1 == u`; in other words, the range for `v` was trimmed down to 2 values. In a regular binary search, we would check against `x[u]`, since `v >= x[u]` (and note that `u` could be `n`, so that explicit test should be made before accessing `x[u]`), but in this modified version, we will check against `x[l]` and `x[u]`, because we want to choose which one of them is closest to `v`. Again, careful must be taken because `l` can remain `-1`, or `u` can be `n`, so we test for these boundary conditions before accessing the array.

It has been a tough journey since we started. Algorithms design techniques and analysis are one of the most powerful tools a programmer can have, but they are also one of the most hard skills to develop. As dedicated and passionate programmers, it is our job to struggle with problems like this and use our head to practice and gain some *aha!* insights in algorithm design.

You can read more about program verification techniques in Column 4. Column 9 addresses very important issues related to code tuning, and keeps on using program verification. A very similar binary search algorithm is developed, with the same loop invariant, to search for the first occurrence of some value in a sorted array with duplicates.

And remember, how lazy or clever programmer you are, your head is your best tool to design efficient algorithms. It is worth spending some time to find a solution that is \\(\mathcal{O}(n)\\), or even the less good \\(\mathcal{O}(n log(n))\\), instead of implementing something \\(\mathcal{O}(n^3)\\) right away and have to wait 15 days instead of a few minutes when the problem size starts to grow...

Think about it!
