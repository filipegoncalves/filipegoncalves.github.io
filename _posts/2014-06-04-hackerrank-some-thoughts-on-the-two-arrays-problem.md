---
layout: post
title: HackerRank - some thoughts on the Two Arrays problem
---

Some ideas on hackerrank's Two Arrays problem

-----

[HackerRank](http://www.hackerrank.com) has some interesting Computer Science problems to practice problem solving capabilities. Mastering algorithms is a key skill for a good software developer. Today, I'd like to discuss the solution for this simple question, taken from HackerRank's *Arrays and Sorting* category:

> You are given two integer arrays, `A` and `B`, each containing `N` integers. The size of the array is less than or equal to 1000. You are free to permute the order of the elements in the arrays.
> 
> Is there an arrangement of the arrays such that \\( A_i+B_i \ge K \\) for all `i`, where \\( A_i \\) denotes the `i`-th element in the array `A`?
> 
> Input Format
> The first line contains an integer, `T`, the number of test-cases. `T` test cases follow. Each test case has the following format:
> 
> The first line contains two integers, `N` and `K`. The second line contains `N` space separated integers, denoting `A`. The third line describes `B` in the same format.
> 
> Output Format
> For each test case, if such an arrangement exists output `YES`, otherwise `NO`.
> 
> 
> Constraints
>
> \\( 1 \le T <= 10 \\)
>
> \\( 1 \le N <= 1000 \\)
>
> \\( 1 \le K \le 10^9 \\)
>
> \\( 0 \le A_i, B_i \le 10^9 \\)
> 
> 
> Sample Input
> 
> 2
>
> 3 10
>
> 2 1 3
>
> 7 8 9
>
> 4 5
>
> 1 2 2 1
>
> 3 3 3 4
> 
> 
> Sample Output
> 
> YES
>
> NO
> 
> Explanation
> 
> The first input has 3 elements in `A` and `B`, we see that that one of the arrangements, `3 2 1` and `7 8 9`, has each pair of elements (3+7, 2+8 and 9+1) summing up to 10 and hence the answer is `YES`.
> 
> The second input has `B` with three 3s. So, we need at least three numbers in `A` that are greater than 1. As this is not the case, the answer is `NO`.
 
For the record, this is rated *Easy*, the lowest difficulty level in HackerRank.

There are certainly other possible approaches, but a reasonably good one, and the one I chose, is to sort both arrays, but one of them in reverse. For example, we can sort `A` in ascending order and sort `B` in descending order. By doing so, for each `i`, then `A[i]` is the `i`-th smallest element on `A`, and `B[i]` is the `i`-th largest element on `B`. It's as if we were globally maximizing each `A[i]+B[i]` (or, in other words: we are minimizing the standard deviation over all values of `A[i]+B[i]`). It should be intuitive to see that if we can find at least a value for `i` such that \\( A_i+B_i < K \\) after sorting, then there is no possible arrangement (otherwise, there is, since we just found one).

My implementation, in Python:

```python
def can_rearrange(a, b, k):
    a.sort()
    b.sort(reverse=True)
    for ai, bi in zip(a, b):
        if ai+bi < k:
            return False
    return True


for _ in xrange(int(raw_input())):
    n, k = (int(x) for x in raw_input().split(' '))
    a = [int(ai) for ai in raw_input().split(' ')]
    b = [int(bi) for bi in raw_input().split(' ')]
    if can_rearrange(a, b, k):
        print 'YES'
    else:
        print 'NO'
```

The algorithm running time is bounded by the sorting operations, hence, it's \\( \mathcal{O}(n log(n)) \\).

Now, even though that code is correct and it passed every testcase, I found it interesting to dig deeper on this, and actually prove that the algorithm always produces the correct output.

Basically, proving that this algorithm works boils down to proving that if there's at least one value for `i` where \\( A_i+B_i < K \\) for our sorted arrays, then there is no other arrangement where \\( A_i+B_i \ge  K \\) for every `i`. Let's formalize this idea: let \\( P(A,B) \\) denote the set of all tuples \\( (x, y) \\), where `x` is a permutation of `A` and `y` is a permutation of `B`. That is, \\( P(A,B) \\) produces a set where each element is a 2-tuple denoting a particular permutation of `A` and `B`. Also, let \\( P_s(A,B) \\) be the specific element of \\( P(A,B) \\) where `A` is sorted in ascending order, and `B` is sorted in descending order.

To show that the algorithm is correct, we want to prove that the following is true:

$$
\exists A_i, B_i \in P_s(A,B) : A_i + B_i < k \implies \forall (a, b) \in P(A, B) \exists a_i, b_i : a_i + b_i < k
$$

That reads: If there is at least a pair of elements \\( A_i \\) on \\( A \\) and \\( B_i \\) on \\( B \\) in \\( P_s(A,B) \\) whose sum is lower than \\( k \\), then for every other arrangement \\( (a,b) \\), where \\( a \\) is a permutation of \\( A \\) and \\( b \\) is a permutation of \\( B \\), there also exists at least a pair of elements in \\( a \\) and \\( b \\) with sum lower than \\( k \\). To put it another way: if it doesn't work with the sorted arrangement, it won't work with *any* arrangement.

The first baby step towards building a proof is to recall implications properties; in particular, we know that if \\( A \\) implies \\( B \\), then \\( \neg B \\) implies \\( \neg A \\):

$$
A \implies B \Leftrightarrow \neg B \implies \neg A
$$

That's one of the first properties learned in the first weeks of any Logic course, but if the reader is not familiar with it, a good way to think about it is with an example: imagine \\( A \\) is "It's sunny", and \\( B \\) is "I go to the beach". Then, \\( A \implies B \\) means "If it's sunny, then I go to the beach". The property above tells us that this is the same as saying "If I don't go to the beach, then it's not sunny". Notice, however, that if you do go to the beach, then you don't know whether it's sunny or not; one is free to go to the beach anytime he wants: the implication only requires going to the beach if it's sunny. Formally, it means that \\( A \implies B \\) does **not** mean that \\( B \implies A \\). All you know is that if you didn't go to the beach, then for sure it's not sunny.

That being said, we are ready to roll our proof. Proving that the former implication is true is the same as proving that when the right hand side is false, then the left hand side is false too. So, instead, we are going to prove that:

$$
\begin{split}
\neg (\forall (a, b) \in P(A, B) &\exists a_i, b_i : a_i + b_i < k) \implies \\
&\neg (\exists A_i, B_i \in P_s(A,B) : A_i + B_i < k)
\end{split}
$$

Now, I'm not here to teach you basic logic manipulation, so you're going to have to believe me on this one and assume that the above is equivalent to:

$$
\begin{split}
\exists (a,b) \in P(A,B) : &\forall a_i, b_i \in (a,b) \ a_i+b_i \ge k \implies \\
&\forall A_i, B_i \in P_s(A,B) \ A_i + B_i \ge k
\end{split}
$$

Basically, the negation of "for all X, Y is true" is "There exists at least one X where Y is false", and the negation of "There exists at least one X where Y is true" is "For all X, Y is false".

Now, how are we going to prove that? There may be a simpler way, but I believe proof by contradiction is a good choice. We will assume that the implication is false, and we will find a contradiction, and therefore the implication must be true. 

In this specific case, this means that we will assume there exists an arrangement where \\( a_i+b_i \ge k \\) but there's at least a pair \\( A_i \\), \\( B_i \\) in \\( P_s(A,B) \\) where \\( A_i+B_i < k \\).

Let \\( j \\) be the first position in \\( P_s(A,B) \\), from left to right, such that \\( A_j+B_j < k \\). Since we assumed there's an arrangement where this never happens, then \\( A_j \\) must have been matched with some other element from \\( B \\) that is greater than \\( B_j \\) in that arrangement. Because \\( B \\) is sorted in descending order, this means that \\( A_j \\) was matched with some \\( B \\) that is currently to the left of \\( B_j \\). Let \\( B_k \\) be that element, i.e., the element matched with \\( A_j \\) in the valid arrangement from our assumption. Moving \\( B_k \\) to match with \\( A_j \\) is the same as swapping \\( B_j \\) with \\( B_k \\). After that, we now know that \\( A_j+B_j \ge k \\), but - and this is important - \\( A_k+B_k < k \\), because \\( A_k < A_j \\), so if the new \\( B_k \\) (aka old \\( B_j \\)) didn't work with \\( A_j \\), then it won't work with \\( A_k \\). So we're back to the same problem: we have to swap \\( B_k \\) with some other element from \\( B \\) to its left, and this will keep happening over and over again, recursively, until we hit \\( A_0 \\) (we're always going towards the left). Again, when we hit \\( A_0 \\), we're left with an element from \\( B \\) that didn't work with other elements of \\( A \\) greater than \\( A_0 \\), so it won't work with \\( A_0 \\) as well. 

As it turns out, saying that there is an arrangement that makes \\( A_i+B_i \ge k \\) valid but \\( P_s(A,B) \\) invalid is the same as saying that given a sorted array \\( A \\) and a sorted array \\( B \\) in descending order, there exists \\( a_i \\) and \\( a_j \\) in \\( A \\) and \\( b_k \\) in \\( B \\) such that:

$$
a_i \ge a_j \wedge a_i + b_k < k \wedge a_j + b_k \ge k
$$

By this time, we can spot the contradiction. A bit of manipulation on \\( a_j \\) and \\( a_i \\) shows that this is the same as:

$$
a_i \ge a_j \wedge a_i < k-b_k \wedge a_j \ge k-b_k
$$

This effectively places \\( k-b_k \\) in two distinct places on the real line (one to the left of \\( a_j \\), another on the right of \\( a_i \\)). So, we clearly have a contradiction. Therefore, our earlier assumption is wrong, which means that this is true:

$$
\begin{split}
\exists (a,b) \in P(A,B) : &\forall a_i, b_i \in (a,b) \ a_i+b_i \ge k \implies \\
&\forall A_i, B_i \in P_s(A,B) \ A_i + B_i \ge k
\end{split}
$$

Which in turn means that this is true:

$$
\begin{split}
\neg (\forall (a, b) \in P(A, B) &\exists a_i, b_i : a_i + b_i < k) \implies \\
&\neg (\exists A_i, B_i \in P_s(A,B) : A_i + B_i < k)
\end{split}
$$

And, equivalently, our former, original implication, is true:

$$
\exists A_i, B_i \in P_s(A,B) : A_i + B_i < k \implies \forall (a, b) \in P(A, B) \exists a_i, b_i : a_i + b_i < k
$$

Thus making the algorithm correct.

Exercises like these are for a programmer what weight lifting is for athletes: they are necessary to build muscle that is needed out there on the field. This is not theoretical maths bullshit - these are important skills to keep in mind. Now, time to work on the next challenge.
