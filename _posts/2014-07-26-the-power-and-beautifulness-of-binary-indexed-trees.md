---
layout: post
title: The power and beautifulness of Binary Indexed Trees
---

Why you must learn Binary Indexed Trees

-----

Until a couple of days ago, I didn't know what a Binary Indexed Tree was. And, oh, mind you, this knowledge totally filled a gap in my life. I mean, seriously, it did. I just don't know how I made it all this time without it, because it's a very useful data structure, easy to implement, extremely efficient, and has a relatively low, if non-existent, theoretical background.

It all started with - you guessed it - a hackerrank challenge. I'll get into that probably on my next post, but for now, I'm happy with a post about Binary Indexed Trees.

A Binary Indexed Tree is a data structure particularly useful for efficient cumulative frequency counting over a set of predefined numbers. Assume, without loss of generality, that you have a set of integers `1` to `N`. You're working on a problem where, for whatever reason, it is important to keep track of each integer's frequency. For example, the frequency of `1` is `23`; the frequency of `2` is `10`, etc, you get the point.

You are to code a function `f(x)` that returns the cumulative frequency for `x`, or, in other words, \\(f(x) = \sum_{i=1}^{x} F_i\\), where \\(F_i\\) is the frequency of \\(i\\).

The naive approach involves storing the frequencies of each number in an array. Then, to retrieve the cumulative frequency for `x`, one would initialize a counter to 0, and loop from `1` to `x`, adding into the counter the frequency of every number along the way.

But of course, that's easy, boring, and unnecessarily slow. This is where Binary Indexed Trees join the party. This problem can be solved in \\(O(log(n))\\) time using a Binary Indexed Tree. This article is divided into small subsections. The first one explains the rationale behind Binary Indexed Trees, and it is essentially a copy of [this outstanding answer](http://cs.stackexchange.com/a/10541) from CS stackexchange. I include this answer here for future reference, and to keep this post self-contained.

The second section goes a little bit further and adds in some information that would make that answer perfect: it explains *why* this works. It's not that difficult to see why, but I think it's a nice piece of complementary information.

The third and last section shows a sample implementation in C.

## Rationale

(taken from [here](http://cs.stackexchange.com/a/10541))

Intuitively, you can think of a binary indexed tree as a compressed representation of a binary tree that is itself an optimization of a standard array representation. This answer goes into one possible derivation.

Let's suppose, for example, that you want to store cumulative frequencies for a total of 7 different elements. You could start off by writing out seven buckets into which the numbers will be distributed:

```
[   ] [   ] [   ] [   ] [   ] [   ] [   ]
  1     2     3     4     5     6     7
```

Now, let's suppose that the cumulative frequencies look something like this:

```
[ 5 ] [ 6 ] [14 ] [25 ] [77 ] [105] [105]
  1     2     3     4     5     6     7
```

Using this version of the array, you can increment the cumulative frequency of any element by increasing the value of the number stored at that spot, then incrementing the frequencies of everything that come afterwards. For example, to increase the cumulative frequency of 3 by 7, we could add 7 to each element in the array at or after position 3, as shown here:

```
[ 5 ] [ 6 ] [21 ] [32 ] [84 ] [112] [112]
  1     2     3     4     5     6     7
```

The problem with this is that it takes \\(\mathcal{O}(n)\\) time to do this, which is pretty slow if `n` is large.

One way that we can think about improving this operation would be to change what we store in the buckets. Rather than storing the cumulative frequency up to the given point, you can instead think of just storing the amount that the current frequency has increased relative to the previous bucket. For example, in our case, we would rewrite the above buckets as follows:

Before:

```
[ 5 ] [ 6 ] [21 ] [32 ] [84 ] [112] [112]
  1     2     3     4     5     6     7
```

After:

```
[ +5] [ +1] [+15] [+11] [+52] [+28] [ +0]
```

Now, we can increment the frequency within a bucket in time \\(\mathcal{O}(1)\\) by just adding the appropriate amount to that bucket. However, the total cost of doing a lookup now becomes \\(\mathcal{O}(n)\\), since we have to recompute the total in the bucket by summing up the values in all smaller buckets.

The first major insight we need to get from here to a binary indexed tree is the following: rather than continuously recomputing the sum of the array elements that precede a particular element, what if we were to precompute the total sum of all the elements before specific points in the sequence? If we could do that, then we could figure out the cumulative sum at a point by just summing up the right combination of these precomputed sums.

One way to do this is to change the representation from being an array of buckets to being a binary tree of nodes. Each node will be annotated with a value that represents the cumulative sum of all the nodes to the left of that given node. For example, suppose we construct the following binary tree from these nodes:

```
             4
          /     \
         2       6
        / \     / \
       1   3   5   7
```

Now, we can augment each node by storing the cumulative sum of all the values including that node and its left subtree. For example, given our values, we would store the following:

Before:

```
[ +5] [ +1] [+15] [+11] [+52] [+28] [ +0]
  1     2     3     4     5     6     7
```

After:

```
                 4
               [+32]
              /     \
           2           6
         [ +6]       [+80]
         /   \       /   \
        1     3     5     7
      [ +5] [+15] [+52] [ +0]
```

Given this tree structure, it's easy to determine the cumulative sum up to a point. The idea is the following: we maintain a counter, initially 0, then do a normal binary search up until we find the node in question. As we do so, we also the following: any time that we move right, we also add in the current value to the counter.

For example, suppose we want to look up the sum for 3. To do so, we do the following:

* Start at the root (4). Counter is 0
* Go left to node (2). Counter is 0
* Go right to node (3). Counter is 0 + 6 = 6
* Find node (3). Counter is 6 + 15 = 21.

You could imagine also running this process in reverse: starting at a given node, initialize the counter to that node's value, then walk up the tree to the root. Any time you follow a right child link upward, add in the value at the node you arrive at. For example, to find the frequency for 3, we could do the following:

* Start at node (3). Counter is 15
* Go upward to node (2). Counter is 15 + 6 = 21
* Go upward to node (1). Counter is 21

To increment the frequency of a node (and, implicitly, the frequencies of all nodes that come after it), we need to update the set of nodes in the tree that include that node in its left subtree. To do this, we do the following: increment the frequency for that node, then start walking up to the root of the tree. Any time you follow a link that takes you up as a left child, increment the frequency of the node you encounter by adding in the current value.

For example, to increment the frequency of node 1 by five, we would do the following:

```
                 4
               [+32]
              /     \
           2           6
         [ +6]       [+80]
         /   \       /   \
      > 1     3     5     7
      [ +5] [+15] [+52] [ +0]
```

Starting at node 1, increment its frequency by 5 to get

```
                 4
               [+32]
              /     \
           2           6
         [ +6]       [+80]
         /   \       /   \
      > 1     3     5     7
      [+10] [+15] [+52] [ +0]
```

Now, go to its parent:

```
                 4
               [+32]
              /     \
         > 2           6
         [ +6]       [+80]
         /   \       /   \
        1     3     5     7
      [+10] [+15] [+52] [ +0]
```

We followed a left child link upward, so we increment this node's frequency as well:

```
                 4
               [+32]
              /     \
         > 2           6
         [+11]       [+80]
         /   \       /   \
        1     3     5     7
      [+10] [+15] [+52] [ +0]
```

We now go to its parent:

```
               > 4
               [+32]
              /     \
           2           6
         [+11]       [+80]
         /   \       /   \
        1     3     5     7
      [+10] [+15] [+52] [ +0]
```

That was a left child link, so we increment this node as well:

```
                 4
               [+37]
              /     \
           2           6
         [+11]       [+80]
         /   \       /   \
        1     3     5     7
      [+10] [+15] [+52] [ +0]
```

And now we're done!

The final step is to convert from this to a binary indexed tree, and this is where we get to do some fun things with binary numbers. Let's rewrite each bucket index in this tree in binary:

```
                100
               [+37]
              /     \
          010         110
         [+11]       [+80]
         /   \       /   \
       001   011   101   111
      [+10] [+15] [+52] [ +0]
```

Here, we can make a very, very cool observation. Take any of these binary numbers and find the very last 1 that was set in the number, then drop that bit off, along with all the bits that come after it. You are now left with the following:

```
              (empty)
               [+37]
              /     \
           0           1
         [+11]       [+80]
         /   \       /   \
        00   01     10   11
      [+10] [+15] [+52] [ +0]
```

Here is a really, really cool observation: if you treat 0 to mean *left* and 1 to mean *right*, the remaining bits on each number spell out exactly how to start at the root and then walk down to that number. For example, node 5 has binary pattern 101. The last 1 is the final bit, so we drop that to get 10. Indeed, if you start at the root, go right (1), then go left (0), you end up at node 5!

The reason that this is significant is that our lookup and update operations depend on the access path from the node back up to the root and whether we're following left or right child links. For example, during a lookup, we just care about the left links we follow. During an update, we just care about the right links we follow. This binary indexed tree does all of this super efficiently by just using the bits in the index.

The key trick is the following property of this perfect binary tree:

> Given node `n`, the next node on the access path back up to the root in which we go right is given by taking the binary representation of `n` and removing the last 1.

For example, take a look at the access path for node 7, which is 111. The nodes on the access path to the root that we take that involve following a right pointer upward is

* Node 7: 111
* Node 6: 110
* Node 4: 100

All of these are right links. If we take the access path for node 3, which is 011, and look at the nodes where we go right, we get

* Node 3: 011
* Node 2: 010
* (Node 4: 100, which follows a left link)

This means that we can very, very efficiently compute the cumulative sum up to a node as follows:

```
    Write out node n in binary.
    Set the counter to 0.
    Repeat the following while n ≠ 0:
        Add in the value at node n.
        Remove the rightmost 1 bit from n.
```

Similarly, let's think about how we would do an update step. To do this, we would want to follow the access path back up to the root, updating all nodes where we followed a left link upward. We can do this by essentially doing the above algorithm, but switching all 1's to 0's and 0's to 1's.

The final step in the binary indexed tree is to note that because of this bitwise trickery, we don't even need to have the tree stored explicitly anymore. We can just store all the nodes in an array of length `n`, then use the bitwise twiddling techniques to navigate the tree implicitly. In fact, that's exactly what the bitwise indexed tree does - it stores the nodes in an array, then uses these bitwise tricks to efficiently simulate walking upward in this tree.

## Why it works
Staring at the data structure for a while should convince you that this works only as long as every number from `1` to `N `is in the tree. In other words, you can't make a Binary Indexed Tree for gaped sequences (or, well, you can, just keep the frequency for the nonexistent values set to 0 - note that I said the frequency, not the accumulated frequency, which is what the tree stores anyway). Sidenote: by "staring" at a data structure I mean literally staring at its representation in paper. It's a good exercise. Draw the tree, look at it. Things in paper make sense.
It works only when every value is in the tree because the bit twiddling hacks that allow the algorithm to walk along the path from a node back up to the root assume that every number in the way exists in the tree. 

For example, let's look at the update function. Unsurprisingly, the process of updating a node `n` starts by increasing that node's frequency. We then have to go back up until the root, updating every node `n'` in the way such that `n` is in the left subtree of `n'`. So, we need to find a good way to retrieve the next node on the path.

Given that the tree is perfectly balanced, a node `N` will have exactly \\(2^{i}-1\\) children on each side, where `i` is the position of the first 1-bit in `N`, starting from 0, and counting from least significant to most significant. For example, for \\(N=11000\\), we have \\(i=3\\). For \\(N=111\\), we have \\(i = 0\\). This can easily be checked by recalling that the left children of `N` consist of every number lower than `N` (similarly for the right side). The "minus 1" is there because we can't count `N` itself as being a child of `N`. There are two interesting points we can take from here.

**Fact number 1**: if node `N` has \\(2^{i}-1\\) children on its left side, then the parent of node `N`, `N'`, will have \\(2*(2^{i}-1)+1\\) children on its left side (`N`'s left and right children, plus `N` itself), which is the same as \\(2^{i+1}-1\\) (and similarly for the right side). That is, the first 1-bit on the parent of a node is one position after the first 1-bit of its children, going from least significant to most significant (and, implicitly, this means that the immediate children of any node have the first 1-bit in the same position).

**Fact number 2**: every leaf node corresponds to an odd number. Notice, if you will, that the parent of a leaf node, no matter whether that leaf node is a left or right child, must have \\(i = 1\\), so that \\(2^{i}-1\\) is 1. Knowing from fact 1 that the first 1-bit on the parent is one position after the first 1-bit on the children, together with `i` being 1 for the parent, makes it clear enough that `i` is 0 for every leaf node, effectively making every leaf correspond to an odd node.

During an update, we want to go back up in the tree, finding every node `N` such that the node being updated is in the left subtree of `N`. Let `n` correspond to the node being updated. The next node (on a bottom-up perspective) that has `n` in its left subtree is obviously going to be greater than `n`. Given that the first 1-bit moves one position left for each level we go up on the tree, all we have to do is find the node above `n` where the first 1-bit position corresponds to a 0 in `n`. This is crystal clear if we look at an example. I mean, as long as `i` is a position where `n` also holds a 1, then `n` will always be bigger than that node. This means that, given node `n`, the next node `n'` such that `n` is in `n'` left subtree is given by `n+(n&-n)` (assuming a two-complement representation). This is bitwise jargon for "Take the rightmost consecutive set of 1 bits, clear them all, and set the first bit after that into 1" - exactly what we want. `n&-n` results in a number where the only bit set to 1 is the least significant bit of `n`. Adding that to `n` has the described effect. We just have to keep doing this until we're out of nodes, that is, until we hit a node greater than the biggest possible value in the set.

What about lookups? Lookups are similar, though a little bit different. In a lookup for the cumulative frequency of node `n`, we start at the desired node, and we have to keep going up, stopping at every node that has `n` in its right subtree, and adding in that node's value (which comprises the cumulative frequency of itself plus every node smaller than `n`).

The next node `n'` up in the tree such that `n` is in its right subtree is the first node that is smaller than `n`. Because each node's parent has the first 1-bit position incremented on each level, this means that we're looking for a node where the first 1-bit position is the same as the next 1-bit position on node `n` (otherwise, choosing a node where the first 1-bit position corresponds to a 0-bit on `n` could lead to a node greater than `n`). Also, the chosen node cannot have any bit set after `n`'s last bit, since that would make it bigger. So, this means that we're looking for a node which looks exactly like `n` in binary, except that the first 1-bit position is the second 1-bit position in `n`. Given `n`, the node that satisfies these conditions can be obtained with `n&n-1`. This is the common bitwise hack to clear the first 1-bit from `n` (again, going from most significant to least significant). We just have to keep doing this until there are no more nodes left, accumulating all the cumulative frequencies along the way.

## The implementation

With all these details sorted out, the implementation is really straightforward. Anyone who read the rest of this article should clearly understand this highly concise, yet somewhat hard to grasp, piece of code:

```c
#define N 1000000

static int bit[N+1];

void update(unsigned int n, int x)
{
	for (; n <= N; bit[n] += x, n += n&-n);
}

int lookup(unsigned int n)
{
	int res = 0;
	for (; n; res += bit[n], n &= n-1);
	return res;
}
```

In the above methods, `n` is the node number, and `x` (in `update()`) is the increment for `n`. A sample usage would be to call `update()` on various nodes (for example, "Node 12 updated with +5"; "Node 4 updated with +8"; "Node 12 updated with +10"), and then call `lookup()` to get the cumulative frequency for a given node.

This is one of the easiest and shortest data structures to implement. I find it absolutely amazing how it solves such a problem so efficiently and with so few lines of code. BEAUTIFUL. Enjoy!
