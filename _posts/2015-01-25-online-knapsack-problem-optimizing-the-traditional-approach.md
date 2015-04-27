---
layout: post
title : Online Knapsack Problem - optimizing the traditional approach
---

An interesting variant of the knapsack problem

-----

The knapsack problem is traditionally used as an example of how to use dynamic programming. Eventually, every Computer Science student will (hopefully) be exposed to dynamic programming, be it on an algorithms class or by reading an algorithms textbook, and come across the well-known Knapsack Problem. I actually remember the day I learned about this. I remember because it was an algorithms class and the Professor decided to introduce the students to the knapsack problem by coming up with an utterly unethical real-world example where solving the knapsack problem would be useful: imagine, if you will, that you are a thief inside a jewelery. You have a bag that can carry at most \\(W\\) lbs, and as a "legitimate" business man that contemporary society misunderstands and marginalizes, you want to maximize the profit of this heist. There are \\(N\\) items to choose from, and each item \\(i\\) weighs \\(w_i\\) and is worth \\(v_i\\) dollars. You are to choose the set of items that maximizes the total value in dollars that can fit in the bag. Haaah... if only thieves knew about the knapsack problem!

Well, in fact, this is the 0/1 version of the knapsack problem, a variant in which the we are allowed to take at most one copy of item \\(i\\) (the unbounded variant assumes that each item \\(i\\) can be taken as many times as you want, and the bounded variant restricts the number of copies of item \\(i\\) up to a known bound).

Yes, not exactly the most appealing example for common hardworking and honest people like us. Fear not, this can be used in less dubious ways: the problem often arises in resource allocation. In general, decision-making processes that are constrained by some financial or capacity limit. A traditional example is how to find the least wasteful way to cut raw materials. It even has applications in cryptography.

The knapsack problem has been studied for over a century now, and is known to be NP-Complete. There is a pseudo-polynomial algorithm that solves it in \\(\mathcal{O}(NW)\\) using dynamic programming. It's pseudo-polynomial because \\(W\\) is not polynomial in the length of the input.

In this article, I would like to focus specifically on the 0/1 knapsack problem.

## A fresh reminder

Let's start out with a fresh reminder of the DP-based solution. Let \\(m\\) be a 2-dimensional array of \\(N+1\\) by \\(W+1\\) such that \\(m[i][w]\\) stores the maximum total value we can get by using the first \\(i\\) items with a weight limit of \\(w\\). This array is initialized like so:

$$
m[0][w] = 0
$$

Now, to fill \\(m[i][w]\\), there are two possibilities: either \\(w_i \leq w\\), or \\(w_i > w\\). In the latter, there is no space left to take item \\(i\\), so \\(m[i][w] = m[i-1][w]\\). On the other hand, if there is space for item \\(i\\), then we make the best choice between \\(m[i-1][w-w_i]+v_i\\) and \\(m[i-1][w]\\), that is, we see how much we get if we take \\(i\\), how much we get if we don't take \\(i\\), and choose the best possibility.

In short, for \\(i > 0\\), we have:

$$
m[i][w] = m[i-1][w] \text{    if } w_i > w
$$

$$
max(m[i-1][w], m[i-1][w-w_i]+v_i) \text{     if } w_i \leq w
$$

The answer is given by \\(m[N][W]\\), the total value we get if we use the first \\(N\\) items (that is, all of them) with a bag that can carry \\(W\\) lbs.

## An online variant

Consider that we're working on a querying system that resembles the 0/1 knapsack problem, but with some interesting properties. The system implements 3 commands: `A` to add an item with a given weight and value to the set of items we can pack in our knapsack, `D` to delete the oldest item, and `Q` to solve the knapsack problem and output the maximum value possible with the current set of items.

Note that only the oldest item can be deleted. Also, at any moment, there are at most \\(L\\) items to choose from. We will assume that the input is well-formed: there are no attempts to delete the oldest item if no items are available, and any sequence of adds and deletes must be such that there are never more than \\(L\\) items to choose from.

The immediate and easy approach is to keep a circular buffer of size \\(L\\) with the active items, and for each query build the dynamic programming table \\(m\\) from scratch and output \\(m[N][W]\\). This is \\(\mathcal{O}(1)\\) per add, \\(\mathcal{O}(1)\\) per delete, and \\(\mathcal{O}(NW)\\) per query, where \\(N\\) is the number of items active at the moment of that query. For a sequence of \\(Q\\) queries, this gets as bad as \\(\mathcal{O}(Q \times N \times W)\\).

This is, of course, suboptimal, to say the least. Is there any optimization you can think of? The idea, of course, is to avoid having to reconstruct \\(m\\) from scratch for each query.

## A step towards a better solution

One of the techniques used to solve hard algorithmic puzzles is to rely on relaxing the requirements, solving the problem for the relaxed version, and then work our way towards the final solution (this is actually one of the 5 approaches suggested in *Cracking the Coding Interview* to tackle hard algorithms questions).

What if delete operations erased the *newest* item instead of the oldest? Wouldn't that make it easier?

If we can only delete the newest item, then we can keep a counter \\(t\\) that stores how many items are available at any moment. For each ADD, we increment \\(t\\) and compute the next row of the dynamic programming table. Since we can only delete the newest item, deleting an item consists of decrementing \\(t\\). A query can be answered by outputting \\(m[t][W]\\). This works because only the last item is deleted, so the set of items that correspond to the first \\(x\\) items remains the same, and as such, \\(m\\) is updated by removing the last row. Furthermore, queries and deletes can now be answered in \\(\mathcal{O}(1)\\), while adds take \\(\mathcal{O}(W)\\) (the cost of computing a new row based on the last row of \\(m\\)).

This easy-to-understand insight can be further explored to arrive at a similar solution for the problem of deleting the oldest item. I suggest stop reading here to see if you can come up with a solution.

## The final touch

As it turns out, the same technique can be used when only the oldest item can be deleted - all it takes is to revert the meaning of \\(m\\). Instead of having \\(m[i][w]\\) being the maximum total value for the first \\(i\\) items and weight \\(w\\), we will have \\(m[i][w]\\) be the maximum value we can get by using the *last* \\(i\\) items and a weight limit \\(w\\).

It's exactly the same, except reversed.

Why?

Because deleting the oldest item is the same as deleting the last row of \\(m\\) (note that in reality we don't really delete the last row, we just decrement \\(c\\)). \\(m[c][w]\\) will then correspond to the optimal solution if we use the last \\(c\\) items, without the oldest (that we just removed).

This is good, there's just a small problem: what about inserts? When an item is inserted, \\(m\\) must be reconstructed from scratch, because the set of the last \\(x\\) items changes. We have to rebuild \\(m[1]\\) because now the latest item is this new item, and since \\(m[1]\\) was rebuilt, we must reconstruct \\(m[2]\\), and so forth. We get \\(\mathcal{O}(1)\\) deletes and queries, but \\(\mathcal{O}(NW)\\) adds. Meh.

So what do we do?

We know how to get \\(\mathcal{O}(1)\\) deletes. We know how to get \\(\mathcal{O}(W)\\) adds. We just have to combine the two. Unsurprisingly, to combine the two - guess what - we will keep two dynamic programming tables: one for deletes, and one for inserts. Let them be \\(del\\) and \\(ins\\). There will be two counters, \\(del_c\\) and \\(ins_c\\). \\(del_c\\) is the index of the last row that was deleted in \\(del\\); \\(ins_c\\) is the index of the next row that is to be written in \\(ins\\) should an ADD arrive.

\\(del[i][w]\\) is the optimal value we get if we can carry at most \\(w\\) lbs and we use the *last* \\(i\\) items. \\(ins[i][w]\\) is the same, but it considers the *first* \\(i\\) items.

Now, before I get into further details, I should be a little more precise: in reality, \\(del[i]\\) considers only the last \\(i\\) items as of the last time that \\(del\\) was built. For example, if there's a new insert, \\(del\\) remains unchanged, because as we will see, inserts only change \\(ins\\), and thus \\(del\\) is not really updated anymore, because the last \\(x\\) items are now different. The overall idea is to insert new items in \\(ins\\) and delete the oldest item from \\(del\\) by decrementing \\(del_c\\). Of course, \\(del\\) may eventually become empty, and at that point we have to reconstruct it from \\(ins\\) (ignoring the first row of \\(ins\\) - because that's the oldest element at that moment) - this is the moment where by the time \\(del\\) is reconstructed, it will really reflect the last \\(x\\) items accurately. Any inserts that come after that will leave \\(del\\) immediately outdated (until the next time it becomes empty), but this is not a problem. The oldest element is still in the last row of \\(del\\) no matter what.

This means that we will have to combine both tables to answer a query, which blows the possibility of having \\(\mathcal{O}(1)\\) queries, but at least we will get something better than \\(\mathcal{O}(NW)\\).

With that in mind, let's get started on the algorithm itself.

\\(del\\) and \\(ins\\) are initialized just as before:

$$
del[0][w] = 0
$$

$$
ins[0][w] = 0
$$

The easiest operation is to add an item. Adding an item remains unchanged: we compute the next row by looking at the previous row and filling in every position (and finally increment \\(ins_c\\)).

```c
// Insert a new item with weight nw and value nv
for (w = 0; w < W+1; w++)
    if (nw <= w)
        ins[ins_c][w] = max(ins[ins_c-1][w], ins[ins_c-1][w-nw]+nv);
    else
        ins[ins_c][w] = ins[ins_c-1][w];
ins_c++;
// New row added
```

Deleting, as I briefly mentioned before, can become slightly more expensive. It depends on the value of \\(del_c\\). When \\(del_c > 1\\), then we just have to decrement \\(del_c\\) - after all, as we saw earlier, the oldest item is always in the last row of \\(del\\).

On the other hand, if \\(del_c = 1\\), \\(del\\) is empty and we don't have enough information to delete the oldest item in \\(\mathcal{O}(1)\\). We have to reconstruct \\(del\\) from scratch, row by row, from the most recent item added to the least recent. To make this possible, we will have to keep a circular buffer of size \\(L\\) that stores the last \\(L\\) items inserted. Then, delete the oldest item from the buffer, by advancing one of the buffer's cursor to the right, and then read the values and weights of the remaining items from the most recent to the least recent, building a row at a time. Finally, since \\(del\\) is now updated, we can safely clean \\(ins\\) by setting \\(ins_c = 1\\).

It looks pretty intuitive to me at this point, but I've been messing around with this for a while, so do make sure that you double-read the last paragraph before moving on. The delete operation is very important. Here's some pseudo-code to illustrate:

```c
// Delete the oldest item
buffer_delete_oldest();
if (del_c > 1) {
    del_c--;
} else {
    // del_c == 1
    foreach item in buffer_rbegin() {
        weight = item.weight;
        value = item.value;
        for (w = 0; w < W+1; w++) {
            if (weight <= w)
                del[del_c][w] = max(del[del_c-1][w], del[del_c-1][w-weight]);
            else
                del[del_c][w] = del[del_c-1][w];
        }
        del_c++;
    }
    // Clear insertion table
    ins_c = 1;
}
```

Recall that besides maintaining \\(ins\\) and \\(del\\) we are required to keep a circular buffer \\(B\\) capable of storing at most \\(L\\) elements (namely, the last \\(L\\) items added). `buffer_delete_oldest()` deletes the oldest element from the circular buffer. `buffer_rbegin()` is assumed to be a function that returns an iterator to the buffer's elements in reverse order of insertion (from tail to head) - this ensures that \\(del\\) is built exactly the way we want. In practice, \\(B\\) is something as simple as an array \\(A\\) and two index values \\(h\\) and \\(t\\) (head and tail); elements are stored in order of insertion in \\(A[h..t-1]\\). Adding an element amounts to storing it in \\(A[t]\\) and incrementing \\(t\\), deleting consists of incrementing \\(h\\), traversing in order of insertion can be done by reading \\(A[h..t-1]\\), and traversing from most recent to least recent is done by reading backwards (\\(A[t-1..h]\\). Of course, for all of this to work smoothly, \\(h\\) and \\(t\\) and incremented and decremented modulo \\(L+1\\) (the size of \\(A\\)).

Last, and perhaps most important, we have queries. Answering a query depends on the state of \\(ins\\) and \\(del\\). If either of them is empty, the query can be answered immediately by reading the last row of the other, non-empty table.

What if neither happens to be empty? We have to combine them. Combining \\(del\\) and \\(ins\\) can be thought of as reserving a certain amount \\(a\\) of weight for \\(del\\), and using the remaining \\(W-a\\) on \\(ins\\). Mathematically, we're looking for:

$$
max(\{del[del_c-1][a]+ins[ins_c-1][W-a] : 0 \leq a \leq W\})
$$

In pseudo-code:

```c
// Answer a query
if (ins_c == 1 || del_c == 1) {
    if (ins_c == 1)
        return del[del_c-1][W];
    else
        return ins[add_c-1][W];
}
res = -INF // -infinity
for (w = 0; w < W+1; w++)
    res = max(res, del[del_c-1][w]+ins[ins_c-1][W-w]);
return res;
```

With this solution, we get an \\(\mathcal{O}(W)\\) worst-case upper bound for inserts and queries. Deletes are interesting because a naive analysis would show that deletes are \\(\mathcal{O}(NW)\\), because after all, in the worst case, *we have to build The Table from scratch*.

However, when we build the table, then the next couple of deletes will be constant. This is a great opportunity to use amortized analysis. Using the [accounting method](http://en.wikipedia.org/wiki/Accounting_method), it can be shown that the amortized cost of a delete is \\(\mathcal{O}(1)\\). Recall that it is an error to attempt to delete an item if there are no active items. Therefore, each delete is matched up with an earlier insert. For each insert, we will charge an extra cost of \\(W\\). This "money" is used to pay the cost of later reconstructing the row for this item in \\(del\\), which takes exactly \\(\mathcal{O}(W)\\). Because we paid in advance for this cost, reconstructing the row for that item in \\(del\\) is essentially free. When we reconstruct the table \\(del\\), we have paid in advance, for each item, the cost of reconstructing the corresponding row in \\(del\\), and so the amortized cost of a delete is \\(\mathcal{O}(1)\\). Ohhh... the joys of amortized analysis.

If you're not familiar with amortized analysis, or you haven't seen it for a long time, you might be having a hard time believing me right now, but the conclusion is that for any valid sequence of deletes and adds, it's as if a single delete is \\(\mathcal{O}(1)\\). To be technically correct, deletes amortize to \\(\mathcal{O}(1)\\). Note that this is not the same as saying that each delete is \\(\mathcal{O}(1)\\). No, that's wrong, and it's clearly not true. Amortized analysis looks at the cost of an operation given a bounded number of times that that operation is executed. It means that no matter how many deletes there are, the total work performed by every delete together is linear in the number of deletes (and thus each one amortizes to \\(\mathcal{O}(1)\\)), even though some of them did more work than others.

## Implementation

I should mention beforehand that I decided to make a generic implementation using templates in C++. This generic implementation has the desiring property of not only computing the optimal value, but it also lists the set of elements leading to that value - clearly, something our thief would want to know.

Anyway, the point is, the implementation may be somewhat harder to read than the pseudo-code examples, but the core algorithm remains the same. It is implemented in the class `OnlineKnapsackSolver<T>`. The constructor receives the maximum number of active elements active at any point, as well as the knapsack capacity. `elem_buff` is the private attribute that implements the circular buffer. Note that the buffer stores elements of type \\(T*\\), so it is imperative that the code interfacing with this class does not free the pointers passed before destroying the solver instance.

Solutions are reported using the helper class `KnapsackSolution<T>`. The total value can be read by calling `get_value()`, the size (number of items picked) by calling `size()`, and the actual elements can be accessed through an iterator of type `KnapsackSolution<T>::const_iterator`, obtained by calling `begin()` (as usual in C++, the iterator can be incremented, and the sequence of elements ends when that iterator compares equal to the one returned by calling `end()`). Because only pointers are stored, dereferencing the iterator yields a \\(T*\\), not a \\(T\\).

Finally, when a pair of solutions \\(s_1\\) and \\(s_2\\) have the same value, \\(s_1\\) is preferred over \\(s_2\\) if \\(s_1\\) has less items. If there is still a tie, the lexicographically shorter solution is chosen (type \\(T\\) must overload the less-than operator to do the lexicographic comparison). These rules can be changed by editing `KnapsackSolution<T>::operator<()`.

The code below can be found in [my github repo](https://github.com/filipegoncalves/codinghighway/tree/master/generic/OnlineKnapsack). It implements the system we've been discussing throughout the article. Remember that it is assumed that any sequence of adds and deletes is valid, i.e., there are never more active items than the allowed number. You must be careful and ensure that you generate valid inputs.

```cpp
#include <algorithm>
#include <set>
#include <iostream>
#include <vector>
#include <cassert>

using namespace std;


template<typename T>
class KnapsackSolution {
private:

	class ElementPtrComparator {
	public:
		bool operator()(T* const &lhs, T* const &rhs) {
			return *lhs < *rhs;
		}
	};

	unsigned total_value;
	set<T*, ElementPtrComparator> elements;

public:
	typedef typename set<T*, ElementPtrComparator>::const_iterator const_iterator;
	typedef typename set<T*, ElementPtrComparator>::size_type size_type;

	KnapsackSolution() : total_value(0) { }

	unsigned get_value() const { return total_value; }

	typename KnapsackSolution::const_iterator begin() const {
		return elements.begin();
	}

	typename KnapsackSolution::const_iterator end() const {
		return elements.end();
	}

	typename KnapsackSolution::size_type size() const {
		return elements.size();
	}

	void add_element(T *new_element) {
		total_value += new_element->value();
		elements.insert(new_element);
	}

	bool operator<(const KnapsackSolution &rhs) const {
		if (total_value < rhs.get_value())
			return true;
		if (elements.size() > rhs.size())
			return true;
		if (total_value == rhs.get_value() && elements.size() == rhs.size())
			return lexicographical_compare(rhs.begin(), rhs.end(),
						       this->begin(), this->end());
		return false;
	}
};

template<typename T>
KnapsackSolution<T> operator+(KnapsackSolution<T> lhs, const KnapsackSolution<T> &rhs) {
	for (typename KnapsackSolution<T>::const_iterator it = rhs.begin();
	     it != rhs.end();
	     it++)
		lhs.add_element(*it);
	return lhs;
}

template<typename T>
class OnlineKnapsackSolver {
private:
	vector<T*> elem_buff;
	typename vector<T*>::size_type elem_head;
	typename vector<T*>::size_type elem_tail;
	typename vector<T*>::size_type elem_buff_sz;

	vector<vector<KnapsackSolution<T> > > insert_tbl;
	vector<vector<KnapsackSolution<T> > > delete_tbl;
	typename vector<vector<KnapsackSolution<T> > >::size_type insert_cnt;
	typename vector<vector<KnapsackSolution<T> > >::size_type delete_cnt;
	
	typename vector<T*>::size_type max_elements;

	unsigned knapsack_capacity;

	KnapsackSolution<T> solution_buff;

public:
	typedef typename vector<T*>::size_type size_type;
	OnlineKnapsackSolver(size_type nmax_elements, unsigned nknapsack_capacity) :
		max_elements(nmax_elements), knapsack_capacity(nknapsack_capacity) {

		elem_buff = vector<T*>(max_elements+1);
		elem_head = elem_tail = 0;
		elem_buff_sz = max_elements+1;

		insert_tbl = vector<vector<KnapsackSolution<T> > >(max_elements+1);
		for (typename vector<vector<KnapsackSolution<T> > >::iterator it = insert_tbl.begin();
		     it != insert_tbl.end();
		     it++)
			*it = vector<KnapsackSolution<T> >(knapsack_capacity+1);
		insert_cnt = 1;

		delete_tbl = vector<vector<KnapsackSolution<T> > >(max_elements+1);
		for (typename vector<vector<KnapsackSolution<T> > >::iterator it = delete_tbl.begin();
		     it != delete_tbl.end();
		     it++)
			*it = vector<KnapsackSolution<T> >(knapsack_capacity+1);
		delete_cnt = 1;
	}

	void insert(T *new_element) {

		assert(0 < insert_cnt && insert_cnt < max_elements+1);

		elem_buff[elem_tail] = new_element;

		for (unsigned w = 0; w < knapsack_capacity+1; w++) {
			unsigned nweight = elem_buff[elem_tail]->weight();

			if (nweight <= w) {
				KnapsackSolution<T> tmp = insert_tbl[insert_cnt-1][w-nweight];
				tmp.add_element(elem_buff[elem_tail]);

				insert_tbl[insert_cnt][w] = max(insert_tbl[insert_cnt-1][w], tmp);

			} else {
				insert_tbl[insert_cnt][w] = insert_tbl[insert_cnt-1][w];
			}

		}

		elem_tail = (elem_tail+1)%elem_buff_sz;
		insert_cnt++;
	}

	void delete_oldest() {

		assert(elem_head != elem_tail);
		assert(delete_cnt > 0);

		if (delete_cnt > 1) {
			elem_head = (elem_head+1)%elem_buff_sz;
			delete_cnt--;
			return;
		}

		assert(delete_cnt == 1);
		assert(insert_cnt > 1);

		typename vector<T*>::size_type i = (elem_tail == 0 ? elem_buff_sz-1 : elem_tail-1);
		while (i != elem_head) {

			assert(delete_cnt < max_elements+1);

			for (unsigned w = 0; w < knapsack_capacity+1; w++) {
				unsigned nweight = elem_buff[i]->weight();
				if (nweight <= w) {
					KnapsackSolution<T> tmp = delete_tbl[delete_cnt-1][w-nweight];
					tmp.add_element(elem_buff[i]);

					delete_tbl[delete_cnt][w] = max(delete_tbl[delete_cnt-1][w], tmp);

				} else {
					delete_tbl[delete_cnt][w] = delete_tbl[delete_cnt-1][w];
				}
			}

			i = (i == 0 ? elem_buff_sz-1 : i-1);
			delete_cnt++;
		}

		elem_head = (elem_head+1)%elem_buff_sz;
		insert_cnt = 1;
	}

	const KnapsackSolution<T> &solve() {
		if (insert_cnt == 1 || delete_cnt == 1) {
			if (insert_cnt == 1)
				return delete_tbl[delete_cnt-1][knapsack_capacity];
			else
				return insert_tbl[insert_cnt-1][knapsack_capacity];
		}

		assert(insert_cnt > 1 && delete_cnt > 1);

		solution_buff = insert_tbl[insert_cnt-1][knapsack_capacity]+delete_tbl[delete_cnt-1][0];
		for (unsigned w = 1; w < knapsack_capacity+1; w++) {
			const KnapsackSolution<T> &ins_tmp = insert_tbl[insert_cnt-1][knapsack_capacity-w];
			const KnapsackSolution<T> &del_tmp = delete_tbl[delete_cnt-1][w];
			unsigned comb_value = ins_tmp.get_value()+del_tmp.get_value();

			if (comb_value > solution_buff.get_value())
				solution_buff = ins_tmp+del_tmp;
		}

		return solution_buff;
	}
};



class Item {
private:
	unsigned id;
	unsigned val;
	unsigned w;

public:
	void set_info(unsigned nid, unsigned nvalue, unsigned nweight) {
		id = nid;
		val = nvalue;
		w = nweight;
	}

	unsigned value() const { return val; }
	unsigned weight() const { return w; }

	bool operator<(const Item &rhs) const {
		return id < rhs.id;
	}

	friend ostream &operator<<(ostream &os, const Item &it);
};

ostream &operator<<(ostream &os, const Item &it) {
	os << it.id;
	return os;
}

#define MAX_INSERTS 10000
static Item items[MAX_INSERTS+1];
static size_t items_i;

void op_query(OnlineKnapsackSolver<Item> &solver) {
	const KnapsackSolution<Item> &solution = solver.solve();

	cout << "The best choice is to take " << solution.size() << " items (total: " << solution.get_value() << ")" << endl;
	cout << "Items:";
	for (KnapsackSolution<Item>::const_iterator it = solution.begin();
	     it != solution.end();
	     it++)
		cout << " " << **it;
	cout << endl;
}

void op_add(OnlineKnapsackSolver<Item> &solver) {
	unsigned value, weight;

	cin >> value >> weight;
	assert(cin);

	items[items_i].set_info(items_i, value, weight);
	solver.insert(&items[items_i]);
	items_i++;
}

void op_del(OnlineKnapsackSolver<Item> &solver) {
	solver.delete_oldest();
}

int main() {

	unsigned items_window, knapsack_cap;

	cout << "How many items will be active at once?" << endl;
	cin >> items_window;
	assert(cin);

	cout << "What is the knapsack's capacity?" << endl;
	cin >> knapsack_cap;
	assert(cin);

	OnlineKnapsackSolver<Item> slv(items_window, knapsack_cap);

	cout << "Each item added is assigned a sequential id starting from 0." << endl;
	cout << "You are allowed to perform at most " << MAX_INSERTS << " insertions" << endl;
	cout << "TO perform queries: QUERY" << endl;
	cout << "To insert an item: ADD value weight" << endl;
	cout << "To delete the oldest item: DEL" << endl;
	cout << "To quit: QUIT" << endl;

	string op;
	while (cin >> op) {
		if (op == "QUERY")
			op_query(slv);
		else if (op == "ADD")
			op_add(slv);
		else if (op == "DEL")
			op_del(slv);
		else if (op == "QUIT")
			break;
		else
			assert(0);
	}

	return 0;
}
```

## Practical results

The results are astonishing. I have implemented the brute force solution and compared it with this optimized online version. For small testcases it doesn't make much difference, but huge testcases yielded amazing results: for tremendously big testcases, the online algorithm took around 1 minute and 7 seconds. One of the tests took 1 minute and 58 seconds. No testcase ever took more than 2 minutes with the online algorithm. **The exact same testcases took 2 hours and 15 minutes to complete with the brute force approach**. That's an amazing 98.5% gain over the brute force algorithm.
