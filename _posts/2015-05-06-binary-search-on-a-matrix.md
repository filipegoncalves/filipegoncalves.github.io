---
layout: post
title : Binary Search on a matrix
---

Efficient lookup in a sorted matrix

-----

The question was short and to the point, but it gave food for thought:

> Given an MxN matrix in which each row and each column is sorted in ascending order, write a method to find an element.

I picked this from *Cracking the Coding Interview*. If I were to choose one - and only one - piece of advice for other fellow CS students, I would definitely go with *Whatever you do, never let your coding skills atrophy*. Besides side projects and open source, I strongly believe that solving at least one interview type of question per day is of paramount importance to make sure that the fundamental coding skills and CS concepts stay fresh. It's a programmer's gym.

How would you find an element in a matrix where each row and each column is sorted?

We have a couple of options. The easiest, brute-force solution is to iterate through every element - not particularly exciting, and it doesn't leverage the fact that rows and columns are sorted. And it's \\(\mathcal{O}(MN)\\) - meh!

The published solution is a hybrid binary search, but it turns out that it is not optimal. Let's look at the example used in the solutions:

|     |     |      |     |
| :---: | :---: | :----: | :---: |
| 15  | 20  | 40   | 85  |
| 20  | 35  | 80   | 95  |
| 30  | 55  | 95   | 105 |
| 40  | 80  | 100  | 120 |


Given a cell \\((i, j)\\) in this matrix, what do we know about the nearby positions? Consider the position \\((2, 2)\\), which has the value 95. Rows are sorted, so anything to the right of 95 is greater than or equal to 95. Columns are sorted too, so anything below is also greater than or equal to 95. That is, the set of positions \\((2, i\\)), for \\(i > 2\\) hold values that are greater than or equal to 95. The same is true for positions \\((i, 2\\)), where \\(i > 2\\). But for each of these positions, remember that the corresponding rows and columns are sorted, which means that this property transitively propagates to the right and bottom of each of these rows and columns. So, 95 is the smallest value of the sub-matrix that has 95 on the upper left corner. This sub-matrix is highlighted in bold below:

|     |     |      |     |
| :---: | :---: | :----: | :---: |
| 15  | 20  | 40   | 85  |
| 20  | 35  | 80   | 95  |
| 30  | 55  | **95**   | **105** |
| 40  | 80  | **100**  | **120** |

Similarly, the reverse is also true: 95 is the largest value in the sub-matrix that has 95 on the lower right corner. 

|     |     |      |     |
| :---: | :---: | :----: | :---: |
| **15**  | **20**  | **40**   | 85  |
| **20**  | **35**  | **80**   | 95  |
| **30**  | **55**  | **95**   | 105 |
| 40  | 80  | 100  | 120 |

In general, any sub-matrix inside this matrix will have the minimum value living on the upper left borner, and the largest value living on the lower right corner. Why is this useful? Say we're trying to find a value `v`. By comparing `v` with 95, we can immediately rule out some sub-matrices. For example, if we're looking for 120, we know for sure that it is not in the sub-matrix that has 95 on the lower right corner, since 95 is the largest value of that sub-matrix. Because 120 > 95, we know that maybe it is in the sub-matrix that has 95 on the upper left corner.

But wait, it's not that easy. We can't simply recurse to this single sub-matrix. Just to make sure everyone is on the same page, let's see what can go wrong. We compared 120 with 95, and the conclusion is that 120 may be on this sub-matrix:

|      |     |
| :----: | :---: |
| 95   | 105 |
| 100  | 120 |

It looks good, but it's not good enough. What if 120 was not on this sub-matrix? For all we know, this is valid:

|     |     |      |     |
| :---: | :---: | :----: | :---: |
| 15  | 20  | 40   | 85  |
| 20  | 35  | 80   | 120 |
| 30  | 55  | 95   | 125 |
| 40  | 80  | 100  | 130 |

What happens with this matrix? When we compare 95 with 120, we recurse with this sub-matrix:

|      |     |
| :----: | :---: |
| 95   | 125 |
| 100  | 130 |

And 120 is not there - we should have recursed with this one instead:

|      |     |
| :----: | :---: |
| 40   | 85  |
| 80   | 120 |
| 95   | 125 |

Given a cell in a matrix, we know how the other elements of the lower right sub-matrix and the upper left sub-matrix relate to that cell. But what about the upper right and lower left sub-matrices?

The sad truth is that there is no relation. The elements on the upper right and lower left sub-matrices can be greater than, equal to, or lower than the value on that cell. This essentially means that in the worst case we may have to recurse on 3 sub-matrices out of 4. Going back to our earlier example: if we find 120 on that lower right sub-matrix, that is great, but if we don't find it, we have to recurse on these 2 sub-matrices:

|      |     |
| :---:  | :---: |
| 40   | 85  |
| 80   | 95  |
| 95   | 105 |

And:

|     |     |      |
| :---: | :---: | :----: |
| 30  | 55  | 95   |
| 40  | 80  | 100  |

With this in mind, we are ready to discuss two slightly different approaches.

## The official, suboptimal solution

The solution proposed in the book works as follows: perform a binary search on the diagonal to find the target value, or the largest integer \\(N\\) in the diagonal such that \\(N < v\\) if \\(v\\) is not found (side note for the astute reader: binary search is a very popular topic in Computer Science, but people often struggle when asked to implement it, *especially* when some changes are required - for example, how would you implement a modified binary search that returns the closest match if an exact match is not possible? How would you implement binary search to return the first occurrence of a value if the array contains duplicates? Compare your solution with [the code on my repository](https://github.com/filipegoncalves/interview-questions/blob/master/DS_Algos/bin_search/binary_search.c)). Let \\((i, j)\\) be the position returned by the diagonal binary search.

If \\(v\\) is equal to `matrix[i][j]`, we found it, and we return \\((i, j)\\). If \\(v\\) is less than `matrix[i][j]`, recursively find \\(v\\) on the upper left sub-matrix. Otherwise, compare \\(v\\) with the next diagonal element, `matrix[i+1][j+1]` (if it exists). Again, if \\(v\\) is equal to that, return; otherwise, if \\(v\\) is greater than that, recurse to the lower right sub-matrix. In either case, if \\(v\\) was found, that's awesome, but if it wasn't, or if \\(matrix[i][j] < v < matrix[i+1][j+1]\\) (in which case we didn't recurse neither to the upper left sub-matrix nor to the lower right sub-matrix), recursive on the upper right sub-matrix, and if *that* doesn't work, recurse on the lower left sub-matrix.

Whatever happens, we know one thing: even though we partition the matrix in 4 sub-matrices, we *never* recurse on all of them. In the worst case, we recurse on 3 out of 4 sub-matrices. 

What's the running time of this approach? Assuming the 4 sub-matrices are relatively balanced, each step reduces the problem by 75%. There are \\(M \times N\\) elements in the original matrix, and the base case is a one-element matrix. Simply put, we are continuously dividing the problem size by \\(\frac{4}{3}\\) (or, equivalently, multiplying it by \\(\frac{3}{4}\\)). How many times do we need to divide \\(M \times N\\) by \\(\frac{4}{3}\\) to get a 1x1 matrix?

$$
\frac{M \times N}{(\frac{4}{3})^i} = 1
$$

Solving for \\(i\\), we get \\(i = log_{\frac{4}{3}}(M \times N)\\). Yes, a weird base-\\(\frac{4}{3}\\) logarithm, but it's still a logarithm. Each recursive call involves \\(\mathcal{O}(log(min(M, N)))\\) work to perform the binary search on the diagonal (this is a pessimist approximation. Note that as soon as we start recursing, we get smaller matrices, so \\(min(M, N)\\) is perhaps a bit of a brute force estimation for the diagonal size), so the total running time is \\(\mathcal{O}(log_{\frac{4}{3}}(M \times N) \times log(min(M, N)))\\). By the way, the logarithm base is not really important in big-O analysis, so we'll just use that in our favor and reduce this to \\(\mathcal{O}(log(M \times N) \times log(min(M, N)))\\).

[This code in my github repository](https://github.com/filipegoncalves/interview-questions/blob/master/sorting_searching/2d_binsearch/solution.c) implements this algorithm.

Can we do better? Let's find out.

## The next step

One would think that the algorithm outlined above is optimal. Logarithms are usually good in Computer Science, and the best search algorithms are all \\(\mathcal{O}(log(N))\\), so this must be it. We can't optimize any further.

Or can we?

The problem with the proposed solution is that the algorithm can degrade into linear search very quickly on some very strange inputs. It is always a good exercise to think about odd input and boundary conditions. What happens with 1xN matrices? What about Nx1 matrices? This kind of matrices doesn't have a diagonal, and as such, our "diagonal" is only 1 element. So, each diagonal binary search goes through one element only, so the algorithm ends up scanning element by element - we went all the way from logarithmic to linear time. Oops! This is the worst scenario ever, where sub-matrices are not balanced *at all*.

There's one way to fix this.

What is it we are really trying to do here? We are basically trying to make a 2D binary search. How does 1D binary search work? It looks at the middle element, and decides on which side to recurse based on that element.

What's the equivalent of that in a matrix? The equivalent is to look at the central element. Forget about doing binary search in the diagonal. Why would we do that? Instead, look for the "center of mass" of the matrix. In a square matrix, the center of mass is, well, the center of the square. In a 1xN matrix, is the middle element. Generically, in an \\(M \times N\\) matrix, the center of mass is the cell \\(M/2, N/2\\). This is a *true* 2D binary search, because the center of mass is guaranteed to always split the matrix into 4 equal sub-matrices. The algorithm will never degrade into linear search, and it has the desirable and beautiful property of becoming logically equivalent to traditional binary search for 1xN matrices (aka regular arrays). This is very important.

Furthermore, by comparing against the center of mass, each recursive call does \\(\mathcal{O}(1)\\) work, thus reducing the algorithm's *guaranteed* running time to \\(\mathcal{O}(log(M \times N))\\). No more linear scans in the worst case, no more bad partitioning.

So, to sum it up: the algorithm is exactly the same, but instead of using the closest diagonal match, just compare with the center of mass of the matrix.

[This code](https://github.com/filipegoncalves/interview-questions/blob/master/sorting_searching/2d_binsearch/solution2.c) implements the algorithm outlined above.

## Does it really matter?

Yes, it does. Quite a lot. I ran a few performance tests to compare both solutions. The tests used various matrix dimensions, ranging all the way from 5,000x5,000 up to 10,000x10,000. Each test was repeated 3 times and each execution made a total of 1,000 find queries. In average, the \\(mathcal{O}(log(M \times N))\\) approach ran 75% faster than the \\(mathcal{O}(log(M \times N) \times log(min(M, N)))\\) algorithm. The improvement is considerably higher in rectangular matrices.

## So, this is an interview question?

Apparently, it is. It surely is a tricky question, and I think it would be extremely hard to write all that code in an interview - but I believe that at least designing the algorithm would be achievable. Plus, a candidate's performance is measured and compared against how well other candidates did, so if it's a lot of code that no one can finish in the time span of an interview, that doesn't make you a bad candidate. Remember, it's not about writing code, it's about the thought process.

All in all, I *do* think it's an unfortunate question to get in an interview. Practice makes perfection!
