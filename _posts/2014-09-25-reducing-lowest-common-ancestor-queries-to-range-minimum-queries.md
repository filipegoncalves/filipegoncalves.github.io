---
layout: post
title : Reducing Lowest Common Ancestor queries to Range Minimum queries
---

LCA and RMQ are the same thing

-----

Welcome to my 3rd post about range queries, segment trees, sparse tables, and god knows what else. We did a good job on the last couple of posts: I described the Range Minimum Query problem, and 2 solutions with different preprocessing and query answering times: I discussed the solution based on segment trees, where we can answer any query in \\(\mathcal{O}(log(N))\\) time, and I discussed the sparse table solution, which has the amazing property of answering *>any* query in \\(\mathcal{O}(1)\\) time. We also studied how these solutions generalize to any binary associative function. In the RMQ problem, we just happen to have that function be the \\(min()\\) function, but we could have used \\(max()\\), \\(product()\\), \\(sum()\\), or any variant.

In this article, I will talk about the Lowest Common Ancestor (LCA) problem and discuss how it can be reduced to the RMQ problem. I will finally address some of the issues that were left behind in the previous post, namely, I will show a refactored implementation of the segment tree and the sparse table solution using C++ template classes, so that we can have range queries in arrays of any type \\(T\\). I will not cover the details of segment trees or sparse table on this article; the reader should be familiar with that. Everything's thoroughly described in my last 2 articles, [The Range Minimum Query Problem]({% post_url 2014-09-05-the-range-minimum-query-problem %}) and [Understanding Segment Trees]({% post_url 2014-09-13-understanding-segment-trees %}).

## The LCA problem

Given an n-ary tree, and 2 nodes on that tree \\(u\\) and \\(v\\), the lowest common ancestor between \\(u\\) and \\(v\\), denoted \\(LCA(u, v)\\), is the deepest node \\(w\\) such that \\(u\\) and \\(v\\) are descendants of \\(w\\). If you think of a tree as a family tree, the lowest common ancestor between any two people is a good heuristic that shows how related they are. It's the "last" family member they have in common (and in fact, the LCA problem plays a very important role in bioinformatics, where a tree represents several species, and we want to find out how related 2 species are).

The LCA problem has lots of practical real-world uses. We will reveal, in future posts, how important the role of LCA is in suffix trees if we want to support efficient string operations (this grasps my master's thesis topic) - and we will look at suffix trees too.

The limit is the sky; it can get as interesting as how creative you can be. Consider, for example, the following problem: you have a weighted n-ary tree \\(T\\), possibly unbalanced. Nodes can be inserted or deleted, and an edge's weight can be updated at any time. We want an efficient algorithm that, given two nodes \\(u\\) and \\(v\\), finds the distance between \\(u\\) and \\(v\\), where the distance is the sum of the weights of every edge in the path from \\(u\\) to \\(v\\). This problem suddenly becomes simpler to solve when we realize that the distance between \\(u\\) and \\(v\\), \\(dist(u, v)\\), is equal to \\(dist(root, u)+dist(root, v)-2 \times dist(root, LCA(u, v))\\). If we can answer LCA queries in \\(\mathcal{O}(1)\\), which is wonderful, the problem essentially reduces to finding the distance between the root and another node. We can do that in logarithmic time even in highly unbalanced trees with some advanced techniques that I will cover on my next article (that's why you should visit my blog often!).

## Reduction to RMQ

Reducing LCA to RMQ is extremely useful, because RMQ is a widely studied topic with several solutions. This is best explained by example. Consider the following tree:

![LCA to RMQ Example tree 1]({{ site.baseurl }}{{ site.assets }}lca_to_rmq_ex1.png)

Here's what we're going to do: we are going to write down the nodes that we touch in a DFS. In other words, we are going to note down the callstack for a DFS traversal. But notice that this is not a "normal" DFS, because in a normal DFS, we don't print a node more than once. Here, we will, because we will write out every node that we step into, even when we backtrack. Maybe a better approach is to think of it as "navigating" the tree in a DFS fashion. For this example tree, this is what we get:

```
[A, B, E, B, F, K, F, L, F, B, G, B, A, C, H, M, H, N, H, O, H, C, I, C, A, D, J, P, J, Q, J, D, A]
```

The pseudo-code that describes this traversal is:

```
visit-node(u) 
{
    print(u);
    foreach child c in u.children() {
        visit-node(c);
        print(u);
    }
}
```

The resulting array has the desirable property that the lowest common ancestor of any nodes \\(u\\) and \\(v\\) appears at least once between any occurrences of \\(u\\) and \\(v\\) in that array. The formal proof can be found in several textbooks, or even other websites, but the intuition is that if we visited node \\(u\\) and are now on \\(v\\), then for sure we passed on the lowest common ancestor between the two, since we are going around the tree doing a DFS. Not only that is always true, it can also be seen that the lowest common ancestor will correspond to the shallowest node between any occurrence of \\(u\\) and \\(v\\) in the array - again, this is intuitive when we think about the tree and how the recursion unwinds. We never backtrack above \\(LCA(u, v)\\) between visiting \\(u\\) and \\(v\\), since we always visit every children before returning to the upper layer. 

So, essentially, \\(LCA(u, v)\\) reduces to the problem of finding the shallowest node between any occurrence of \\(u\\) and \\(v\\) in that array.

How do we do that? Well, finding an occurrence of \\(u\\) and \\(v\\) can be done in \\(\mathcal{O}(1)\\) if we build a lookup table mapping nodes to array positions. Finding the shallowest node can be made easy if we store each node's depth along with its identifier when we're building the array. Then, we just have to pass an appropriate function to the RMQ implementation that compares array elements based on the depth field of each entry. In other words: when it comes to RMQ, the input array corresponds to the depths of every node we touched in the DFS traversal; we can then answer \\(LCA(u, v)\\) by querying the RMQ implementation to find the minimum depth value between any occurrence of \\(u\\) and \\(v\\). Because each entry in the array stores both the node identifier and its depth, we can easily return the node identifier that corresponds to that depth.

## Implementation

First, we shall start by defining a common interface for the Range Minimum Query problem. We will assume that our input to RMQ is an array of `Value`, a generic type. The base class stores a copy of the input array and a copy of \\(f()\\), the binary associative function, as part of its implementation. It also has a virtual function, `query()`, that is overriden by the implementor classes. In the case of sparse table, this function will take \\(\mathcal{O}(1)\\) time; in the case of the implementation with segment tree, the function will take \\(\mathcal{O}(log(N))\\):

```cpp
#ifndef RMQ_H
#define RMQ_H

#include <vector>

template<typename Value>
class RMQ {
public:
  RMQ(Value (*fun)(const Value v1, const Value v2),
      const std::vector<Value> input_arr) :
    f_ptr(fun), arr(input_arr) { }

  virtual Value query(const typename std::vector<Value>::size_type i,
                      const typename std::vector<Value>::size_type j) const = 0;

protected:
  Value (*f_ptr)(Value v1, Value v2);
  std::vector<Value> arr;
};

#endif
```

So, now, there's 2 possible implementations. Let's start with the segment tree approach:

```cpp
#ifndef RMQ_SEGTREE_H
#define RMQ_SEGTREE_H

#include <vector>
#include "rmq.h"

template<typename Value>
class RMQ_SegTree : public RMQ<Value> {
public:

  RMQ_SegTree(Value (*f)(const Value v1, const Value v2),
              const std::vector<Value> in_arr) :
    RMQ<Value>(f, in_arr), segtree(std::vector<Value>(4*in_arr.size()))
  {
    build_segtree(0, in_arr.size()-1, 0);
  }

  Value query(const typename std::vector<Value>::size_type i,
              const typename std::vector<Value>::size_type j) const
  {
    return query_aux(0, this->arr.size()-1, i, j, 0);
  }

private:

  typedef typename std::vector<Value>::size_type vsize_t;
  std::vector<Value> segtree;

  Value build_segtree(vsize_t l, vsize_t r, vsize_t pos);
  Value query_aux(vsize_t l, vsize_t r, vsize_t in_l, vsize_t in_r, vsize_t pos) const;
  bool interval_intersects(vsize_t i, vsize_t j, vsize_t p, vsize_t k) const
  {
    return k > j ? p <= j : i <= k;
  }
};

template<typename Value>
Value RMQ_SegTree<Value>::build_segtree(vsize_t l, vsize_t r, vsize_t pos)
{
  if (l == r)
    return segtree[pos] = this->f_ptr(this->arr[l], this->arr[l]);

  vsize_t mid = l+(r-l)/2;
  vsize_t left = pos*2+1;
  vsize_t right = left+1;

  return segtree[pos] = this->f_ptr(build_segtree(l, mid, left),
                                    build_segtree(mid+1, r, right));
}

template<typename Value>
Value RMQ_SegTree<Value>::query_aux(vsize_t l, vsize_t r,
                                    vsize_t in_l, vsize_t in_r, vsize_t pos) const
{
  if (l == in_l && r == in_r)
    return segtree[pos];

  vsize_t mid = l+(r-l)/2;
  vsize_t left = pos*2+1;
  vsize_t right = left+1;

  bool recurse_left = interval_intersects(l, mid, in_l, in_r);
  bool recurse_right = interval_intersects(mid+1, r, in_l, in_r);

  if (recurse_left && recurse_right)
    return this->f_ptr(query_aux(l, mid, in_l, mid, left),
                       query_aux(mid+1, r, mid+1, in_r, right));
  else if (recurse_left)
    return query_aux(l, mid, in_l, in_r, left);
  else
    return query_aux(mid+1, r, in_l, in_r, right);
}

#endif
```

This is pretty similar to the implementation from my previous article, with the necessary adaptions to make it a template class. There is, however, something that might not actually be clear at first: did you notice that I made the `segtree` array 4 times bigger than the input array? Is that enough? Was this a shot in the dark? Where does that 4 come from? I'm specifically referring to the constructor initializer:

```cpp
  RMQ_SegTree(Value (*f)(const Value v1, const Value v2),
              const std::vector<Value> in_arr) :
    RMQ<Value>(f, in_arr), segtree(std::vector<Value>(4*in_arr.size()))
  {
    ...
  }
```

Let's go back to basics: as it turns out, the array representation of a perfectly balanced segment tree for an array of size \\(N\\) will never be more than \\(4*N\\) elements long. Why?

Since we split each interval into two equal halves, the segment tree will have depth \\(\lceil log(N) \rceil\\). Because it is a perfectly balanced binary tree, it will have at most \\(2^{\lceil log(N) \rceil + 1} - 1\\) nodes: recall that a balanced binary tree rooted on a node \\(R\\) has got `size(R->left) \times 2+1` nodes. This property is recursively propagated down to the leaf nodes, which represent trees of size 1, and as we move up, we keep applying that formula. Try it out with a few examples; it can then be seen that a segment tree for an array of size \\(N\\) will have at most \\(\sum\limits_{i = 0}^{\lceil log(N) \rceil} 2^{i}\\) nodes, which is the same as \\(2^{\lceil log(N) \rceil + 1} - 1\\).

We now show that \\(2^{\lceil log(N) \rceil + 1} - 1\\) is bounded by \\(4 \times N\\). First, we know that:

$$
2^{\lceil log(N) \rceil + 1} - 1 < 2^{\lceil log(N) \rceil + 1} = 2 \times 2^{\lceil log(N) \rceil}
$$

The final step is to find a bound for \\(2^{\lceil log(N) \rceil}\\). We know for a fact that \\(\lceil log(N) \rceil\\) is certainly bounded by \\(log(N)+1\\), which leads us to:

$$
2^{\lceil log(N) \rceil} < 2^{log(N)+1}
$$

So, putting it all together, we know that:

$$
2^{\lceil log(N) \rceil + 1} - 1 < 2 \times 2^{\lceil log(N) \rceil} < 2 \times 2^{log(N)+1} = 4 \times 2^{log(N)} = 4 \times N
$$

And there you go. This shows why a segment tree uses linear amount of memory, and why an array of size \\(4N\\) is enough. The rest of the code is very similar to that of my other article, so I think we can now skip to the sparse table solution:

```cpp
#ifndef RMQ_SPARSE_TABLE_H
#define RMQ_SPARSE_TABLE_H

#include <vector>
#include <stdexcept>
#include "rmq.h"

template<typename Value>
class RMQ_SparseTable : public RMQ<Value> {
  public:

  RMQ_SparseTable(Value (*f)(const Value v1, const Value v2),
		  const std::vector<Value> in_arr) :
    RMQ<Value>(f, in_arr)
  {
    build_rmq_array();
  }

  Value query(const typename std::vector<Value>::size_type i,
	      const typename std::vector<Value>::size_type j) const
  {
    typename std::vector<Value>::size_type l = int_log(j-i+1);
    return this->f_ptr(rmq_array[i][l], rmq_array[j+1-(1<<l)][l]);
  }

private:
  std::vector<std::vector<Value> > rmq_array;

  typename std::vector<Value>::size_type int_log(typename std::vector<Value>::size_type val) const;
  void build_rmq_array();
};

template<typename Value>
typename std::vector<Value>::size_type RMQ_SparseTable<Value>::int_log(typename std::vector<Value>::size_type val) const
{

  if (val == 0)
    throw std::invalid_argument("val");

  typename std::vector<Value>::size_type shifts = 0;
  for (; val != 1; val >>= 1, shifts++);
  return shifts;

}

template<typename Value>
void RMQ_SparseTable<Value>::build_rmq_array()
{

  for (typename std::vector<Value>::size_type i = 0; i < this->arr.size(); i++) {
    rmq_array.push_back(std::vector<Value>());
    rmq_array[rmq_array.size()-1].push_back(this->f_ptr(this->arr[i], this->arr[i]));
  }

  for (typename std::vector<Value>::size_type j = 1; (1U << j) <= this->arr.size(); j++)
    for (typename std::vector<Value>::size_type i = 0; i + (1 << j) - 1 < this->arr.size(); i++)
      rmq_array[i].push_back(this->f_ptr(rmq_array[i][j-1], rmq_array[i+(1 << (j-1))][j-1]));

}

#endif
```

This is almost copied from the previous article, nothing has changed other than adapting it to a template class.

## An example n-ary tree class

This is the tree representation that we will be using for our LCA tests. It's nothing very fancy, it's just an n-ary tree of `T`. Classical.

```cpp
#ifndef N_TREE_H
#define N_TREE_H

#include <iostream>

template<typename T>
class N_Tree {
public:

  typedef typename std::vector<N_Tree *>::const_iterator const_iterator;
  typedef typename std::vector<N_Tree *>::iterator iterator;
  typedef unsigned long long size_type;

  N_Tree(T value) : node_value(value) { }

  void add_child(N_Tree *child) {
    children.push_back(child);
  }

  const_iterator begin() const
  {
    return children.begin();
  }

  iterator begin()
  {
    return children.begin();
  }

  iterator end()
  {
    return children.end();
  }

  const_iterator end() const
  {
    return children.end();
  }

  T &operator*()
  {
    return node_value;
  }

private:
  T node_value;
  std::vector<N_Tree *> children;
};
#endif
```

## LCA to RMQ implementation

We are now ready to implement an LCA-to-RMQ solver. Simply put, this will consist of a template class, which I will call `LCA_Solver`, that has an RMQ solver and uses it to answer LCA queries, after converting an `N_Tree` to an array.

Converting the tree to an array that serves as input for RMQ boils down to that visit-node pseudo-code showed earlier. But there are a few implementation details we need to keep in mind: remember that we need to store both the node itself (or, in this case, the pointer to the node in the tree) and its depth. Storing the depth is crucial, since that's what RMQ will look at, but knowing the depth of a node is not of much use, so we keep the extra pointer information.

So, to do this, our input array to RMQ will be an array of something that I called `RMQ_Elem`. An `RMQ_Elem` is a structure holding the pointer to the node it represents, and its depth. That's as simple as that. How do we tell the RMQ solver that we only care about depth? It is really easy: the RMQ solver will be initialized with a function that we will provide it, and that is supposed to know how to compare two items of the input array. This means that we just need to code a function that receives an `RMQ_Elem`, but only works with the depth field.

Another point worth mentioning is the need to remember where a node occurs in the array we build for RMQ. For maximal performance, we don't want to linearly scan the array to find occurrences of the target nodes. Therefore, upon building the array, we build a lookup table at the same time, so that we can know the index of a node in constant time.

At this point, the code pretty much speaks by itself:

```cpp
#ifndef LCA_H
#define LCA_H

#include <map>
#include <vector>
#include <algorithm>
#include <stdexcept>
#include "rmq_sparse_table.h"
#include "rmq_segtree.h"
#include "n_tree.h"

template<typename Value>
class LCA_Solver {
public:
  enum class Impl { RMQ_SegTree, RMQ_SparseTable };

  LCA_Solver(N_Tree<Value> *root, Impl rmq_impl) : tree_root(root)
  {
    reduce_to_rmq(rmq_impl);
  }

  N_Tree<Value> *query(N_Tree<Value> *n1, N_Tree<Value> *n2)
  {
    typename std::vector<RMQ_Elem>::size_type i, j;
    i = occurrences[n1];
    j = occurrences[n2];

    return rmq_solver->query(std::min(i, j), std::max(i, j)).node;
  }
  
private:

  struct RMQ_Elem {
    typename N_Tree<Value>::size_type depth;
    N_Tree<Value> *node;
    
    RMQ_Elem(typename N_Tree<Value>::size_type d, N_Tree<Value> *n) :
      depth(d), node(n) { }
    RMQ_Elem() : depth(0), node(NULL) { }

  };

  N_Tree<Value> *tree_root;
  RMQ<RMQ_Elem> *rmq_solver;
  std::vector<RMQ_Elem> rmq_input_array;
  std::map<N_Tree<Value> *, typename std::vector<RMQ_Elem>::size_type> occurrences;

  static RMQ_Elem rmq_f_min(const RMQ_Elem v1, const RMQ_Elem v2)
  {
    return v1.depth < v2.depth ? v1 : v2;
  }

  void reduce_to_rmq(Impl rmq_impl)
  {

    build_rmq_array(tree_root, 0);
    
    switch (rmq_impl) {
    case Impl::RMQ_SegTree:
      rmq_solver = new RMQ_SegTree<RMQ_Elem>(rmq_f_min, rmq_input_array);
      break;
    case Impl::RMQ_SparseTable:
      rmq_solver = new RMQ_SparseTable<RMQ_Elem>(rmq_f_min, rmq_input_array);
      break;
    default:
      throw std::invalid_argument("rmq_impl"); // Should never happen, we're using enum class
      }

  }

  void build_rmq_array(N_Tree<Value> *node, typename N_Tree<Value>::size_type depth)
  {
    rmq_input_array.push_back(RMQ_Elem(depth, node));
    occurrences[node] = rmq_input_array.size()-1;

    for (typename N_Tree<Value>::const_iterator it = node->begin();
	 it != node->end();
	 it++) {
      build_rmq_array(*it, depth+1);
      rmq_input_array.push_back(RMQ_Elem(depth, node));
    }
  }
  
};

#endif
```

And yes, you guessed it, the `Impl` enum is a simple way for user code to specify which RMQ implementation should be used by the LCA solver. 

On a side note, do notice that our "lookup" table, an `std::map`, is not exactly a constant time lookup table. C++ maps lookup operations are logarithmic in time. But this is rarely a problem.

As a final wrap up, here's a small, quick and dirty `main()` to do some basic testing on your own:

```cpp
#include <iostream>
#include <vector>
#include <limits>
#include "n_tree.h"
#include "lca.h"

using namespace std;

int main()
{
  cout << "How many nodes?" << endl;
  vector<N_Tree<int> >::size_type nodes;
  cin >> nodes;

  vector<N_Tree<int> > tree;
  for (vector<N_Tree<int> >::size_type i = 0; i < nodes; i++)
    tree.push_back(N_Tree<int>(i));

  cout << "Enter lines of the form:" << endl;
  cout << "i n ..." << endl;
  cout << "Where i is the node number, n the number of children, and then list every children." << endl;

  vector<N_Tree<int> >::size_type node, children;
  while (cin >> node >> children) {
    while (children--) {
      vector<N_Tree<int> >::size_type child;
      cin >> child;
      tree[node].add_child(&tree[child]);
    }
  }

  //  LCA_Solver<int> lca_solver(&tree[0], LCA_Solver<int>::Impl::RMQ_SegTree);
  LCA_Solver<int> lca_solver(&tree[0], LCA_Solver<int>::Impl::RMQ_SparseTable);

  cout << "Type in nodes ids for lca query." << endl;

  cin.clear();
  cin.ignore(numeric_limits<streamsize>::max(), '\n');

  vector<N_Tree<int> >::size_type n1, n2;

  while (cin >> n1 >> n2)
    cout << "LCA(" << n1 << ", " << n2 << ") = " <<
      **(lca_solver.query(&tree[n1], &tree[n2])) << endl;

  return 0;
}
```

All of the code can be found on [my github repo](https://github.com/filipegoncalves/codinghighway/tree/master/generic/LCA_RMQ).
