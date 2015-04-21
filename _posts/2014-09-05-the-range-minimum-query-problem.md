---
layout: post
title: The Range Minimum Query (RMQ) problem
---

Impacts and usefulness of the Range Minimum Query problem

-----

The Range Minimum Query problem, which has many possible applications, is an interesting and heavily studied topic that aims to answer the simple question of finding the index of the minimum value in a sub-array efficiently. More precisely, given an array \\(a[0..N-1]\\) of \\(N\\) values, and a closed range \\([i,j]\\), with \\(0 <= i <= j < N\\), we want to be able to quickly get the index of the minimum value in the sub-array \\(a[i..j]\\).

There are 2 solutions that come to mind on a first approach: either we previously stored the index of the minimum value for every possible pair \\([i,j]\\), making it possible to answer any query in \\(\mathcal{O}(1)\\) time, or we don't store anything, and each time a query is made, we iterate through the specified sub-array to find its minimum.

The first solution uses \\(\mathcal{O}(N^2)\\) memory, and the preprocessing step can be as bad as \\(\mathcal{O}(N^3)\\) (although it can be made \\(\mathcal{O}(N^2)\\) with the help of dynamic programming - but still, this is not really satisfactory). The second solution uses \\(\mathcal{O}(1)\\) memory, but each query takes \\(\mathcal{O}(N)\\) time to run.

Can we do better? We surely can.

The range minimum query problem can be approached using several different methods. One of them breaks the array into \\(\sqrt(N)\\) partitions and stores the index of the minimum value of each of these partitions in an auxiliary array with \\(\sqrt(N)\\) elements. Finding the minimum of a sub-array \\([i,j]\\) is now a matter of finding the minimum of the minimums reported by the auxiliary array for every partition that lies inside the interval \\([i,j]\\) plus the minimum of every value inside \\([i, j]\\) that belong to a partition that goes outside of \\([i,j]\\). Such an approach would allow us to answer any query in at most \\(\mathcal{O}(3\sqrt(N))\\) time, and building the auxiliary array takes \\(\mathcal{O}(N)\\) time (basically, for each of the \\(\sqrt(N)\\) partitions, we naively find and store its minimum) and \\(\mathcal{O}(\sqrt(N))\\) memory. That's reasonably acceptable and clever, and it is probably good enough for most real-world applications.

However, today, I would like to discuss another approach, known as the Sparse Table approach. The sparse table approach builds an auxiliary array in \\(\mathcal{O}(N log(N))\\) time, and uses \\(\mathcal{O}(N log(N))\\) memory, but enables us to answer any query in \\(\mathcal{O}(1)\\) time, which is wonderful if we plan to make lots of queries. The more queries we do, the more the preprocessing time spent in the beginning pays off.

So, what exactly is the Sparse Table method, and how exactly does it work?

Like everything else in Computer Science, the Sparse Table method relies on the scalability power of logarithms. We all know that logarithmic time algorithms are desirable and scale well. The method consists of building a 2D array, \\(mins[0..N-1][0..log(N)-1]\\), where \\(mins[i][j]\\) stores the index of the minimum value in the sub-array starting at position \\(i\\) with length \\(2^j\\). So, say we have an array, \\(A\\), with 18 integers:

![RMQ Auxiliary Array]({{ site.baseurl }}{{ site.assets }}rmq_aux_array.png)

\\(mins[2][2]\\) stores the index of the minimum value in the sub-array that starts at \\(A[2]\\) and has length 4. \\(mins[2][3]\\) stores the index of the minimum in the sub-array that starts at \\(A[2]\\) and has length 8. You get the point now. Building such an array takes \\(\mathcal{O}(N log(N))\\) time and memory. We will get to it shortly, but for now, I want to show why having `mins` is useful and important.

The rationale behind `mins` is that if we use it cleverly enough, we can answer any RMQ query in \\(\mathcal{O}(1)\\) time. When we're given a sub-array \\([i,j]\\), we can actually know immediately the minimum in the largest interval whose length is a power of 2 that fits inside that sub-array by reading \\(mins[i][\lfloor log(j-i+1) \rfloor]\\). This is basically the same as saying "Give me the minimum on the sub-array starting at position i and ending at the last position of the largest interval whose length is a power of 2 that ends on or before j". In fact, if the length of the sub-array, \\(j-i+1\\), is exactly a power of 2, then the answer is precisely \\(mins[i][log(j-i+1)]\\). But what should we do otherwise? Notice, if you will, that if the sub-array length is <em>not</em> a power of 2, then we simply can't return \\(mins[i][log(j-i+1)]\\), since the minimum can possibly sit between \\(A[i+2^{\lfloor log(j-i+1) \rfloor}]\\) and \\(A[j]\\). In other words, we're ignoring the elements between the end of the largest interval that is a power of 2 and the end of the sub-array.

The key point to make the final leap is to realize that for any pair \\([i,j]\\), there is at least a value \\(k\\), where \\(i <= k <= j\\) such that \\(j\\) is the endpoint of a perfect interval starting at \\(k\\) with length that is a power of 2. It is easy to think of this visually. After all, if the last perfect interval starting at \\(i\\) ended \\(m\\) positions before \\(j\\), then if we were to start from \\(i+m\\), \\(j\\) would be the endpoint of a perfect interval with the exact same length as before - does that make sense? 

Take this as an example:

![RMQ Example]({{ site.baseurl }}{{ site.assets }}rmq_ex2.png)

With \\(i = 3\\) and \\(j = 8\\), we have an interval of length 6. The largest interval that is a power of 2 that fits inside is an interval of length 4. Using `mins`, we can get the minimum of the sub-array of length 4 starting at position 3 (this sub-array corresponds to the positions in grey). But obviously, this is not enough, because the minimum could (although in this case it doesn't) lie in either \\(A[7]\\) or \\(A[8]\\). But - and this is the "Aha!" step - if we consider an interval of length 4 starting on \\(A[5]\\), then we can also get its minimum using `mins`, and then we just return the minimum between these 2 values. See what we did there? We basically shifted the starting position in such a way that 8 is the endpoint of a perfect interval whose length is a power of 2. Looking at the picture, it is easy to see that for any \\([i,j]\\), we will use \\(mins[i]\\) and \\(mins[i+(j-i+1)-2^{\lfloor log(j-i+1) \rfloor}]\\), which is really just \\(mins[j+1-2^{\lfloor log(j-i+1) \rfloor}]\\). Again, we basically shift \\(i\\) for the necessary amount of positions that were "left behind", so as to get to a position where \\(j\\) is now the end of a perfect interval.

With all this in mind, and assuming that `mins` was previously built, we can formally say that \\(RMQ(i,j)\\) is given by:

$$
\begin{split}
RMQ(i, j) = min(&mins[i][\lfloor log(j-i+1) \rfloor], \\
&mins[j+1-2^{\lfloor log(j-i+1) \rfloor}][\lfloor log(j-i+1) \rfloor])
\end{split}
$$

Again, assuming that we have `mins`, this is all we need to answer any query in \\(\mathcal{O}(1)\\) time.

## Building `mins`

So, our problem is now reduced to building the auxiliary array `mins`. If we are careful, it can be built in \\(\mathcal{O}(N log(N))\\) time - that's neat. We will build it iteratively, using some kind of dynamic programming. Since we're doubling intervals size as \\(j\\) grows in \\(mins[i][j]\\), we can compute \\(mins[i][j]\\) by looking at \\(mins[i][j-1]\\) and \\(mins[i+2^{j-1}][j-1]\\). The intuition is that the minimum value in the sub-array of length \\(2^j\\) starting at \\(i\\) is the smallest value between the minimum in the sub-array of length \\(2^{j-1}\\) starting at \\(i\\) and the minimum in the sub-array of length \\(2^{j-1}\\) starting at \\(i+2^{j-1}\\). We are, essentially, breaking the interval of size \\(2^j\\) into two equal halves, and using `mins` to find the minimum of each of them:

![RMQ Example Fixed]({{ site.baseurl }}{{ site.assets }}rmq_ex3_fixed.png)

Formally, we say that:

$$
mins[i][j] = min(mins[i][j-1], mins[i+2^{j-1}][j-1])
$$

How do we kick it off? We will be building `mins` by interval size: first we build every interval of size 1, then every interval of size 2, then 4, etc. Thus, all we have to do is to start by manually filling every interval of size 1, for every \\(i\\), which is trivial, and then use the above general formula to build the rest of it.

## Implementation

Below you can find  a sample implementation in C++, with a mock `main()` function that provides a rudimentary way to test it. There is not much to say at this point; understanding the theoretical background of RMQ should be enough to dive into code. Have fun with this!

```cpp
#inccude <iostream>
#include <vector>
#include <algorithm>

using namespace std;

vector<int> values;
vector<vector<int> > mins;

/* Assumption: val > 0 */
static vector<int>::size_type int_log(vector<int>::size_type val)
{
	vector<int>::size_type shifts = 0;
	for (; val != 1; val >>= 1, shifts++);
	return shifts;
}

static void preprocess()
{
	for (vector<int>::size_type i = 0; i < values.size(); i++) {
		mins.push_back(vector<int>());
		mins[i].push_back(values[i]);
	}

	for (vector<int>::size_type j = 1; (1U << j) <= values.size(); j++) {
		for (vector<int>::size_type i = 0;
		     i+(1 << j)-1 < values.size();
		     i++) {
			int val = min(mins[i][j-1], mins[i+(1<<(j-1))][j-1]);
			mins[i].push_back(val);
		}
	}
}

static int rmq(vector<int>::size_type from, vector<int>::size_type to)
{
	vector<int>::size_type l = int_log(to-from+1);
	return min(mins[from][l], mins[to+1-(1<<l)][l]);
}

int main()
{
	vector<int>::size_type elems;
	cout << "How many elements?" << endl;
	cin >> elems;

	while (elems--) {
		int val;
		cin >> val;
		values.push_back(val);
	}

	preprocess();

	cout << "Enter an interval [i,j] for RMQ." << endl;

	vector<int>::size_type i, j;
	while (cin >> i >> j)
		cout << "RMQ(" << i << ", " << j << ") = " << rmq(i, j) << endl;

	return 0;
}
```
