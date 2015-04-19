---
layout: post
title: Flattening multi-dimensional linked lists, part 2 - The algorithm
---

Part 2 of the multi-dimensional linked list problem

-----

Woeh! Let me start this post by apologizing to my readers for my silence, but it's been a tough time since then. The main reason for my lack of time comes from several places: projects to deliver at university, StackOverflow addiction, and learning a new language. Oh, and I *finally* managed to start reading *Compilers - Principles, Techniques & Tools*, something I've been planning to do for a long time.

So, as you can see, a lot has been happening under the hood! But let's turn into today's topic. Following [my previous post]({% post_url 2014-03-23-flattening-multi-dimensional-linked-lists-part-1-parsing %}), today, we will be looking at the algorithm that flattens an arbitrarily complex multi-dimensional linked list.

How do we do this? Where should we start? One good way to look at this kind of problems is to solve a few of them by hand with a paper and a pen. When we draw a multi-dimensional linked list in a paper, we can intuitively flatten it out. The question, as it stands, is how to do it in such a way that is easy to code. Let's look at the first example I mentioned in the previous article:

```
1 - 2 - 3 - 4
    |       |
    5 - 6   7
```

One possible approach is to go through the list layer by layer. The second layer contains nodes 5, 6, and 7. We can grab all of these and append them to the first layer, we would get:

```
1 - 2 - 3 - 4 - 5 - 6 - 7
```

This is easy to do for arbitrarily complex multi-dimensional lists on the paper, but think about how you would implement it: that would be a nightmare. To begin with, how would you even find a layer? `7` and `6` are not directly linked. This means that we would have to iterate through all the nodes in the previous layer to group all of their children. Then we would have to link them together, and then append that to the end of the list. And how do you append it to the end efficiently? We don't have a pointer to the end. Maybe we need a preprocessing step to find the end? What if a layer has children itself? When we bring this layer to the top, will its children become part of the second layer? How do we link children with a different ancestor together? We keep a pointer to the last children in the group built so far? 

This is insanely complex and error-prone. As Jon Bentley himself says in this column *Aha! Algorithms*, *A problem that seems difficult may have a simple, unexpected solution*.

The *aha!* insight comes when we look at this from another perspective. Instead of flattening layer by layer, we can flatten node by node. If a node doesn't have a child, no action is necessary. If a node has a child, we recursively flatten the list whose head is that child. After that, we have a node with a child that is the head of a flattened list; all we need to do is insert this list between the current node and the next node. This keeps happening for every node in the list that has a child, until we're out of nodes.

This needs some polishing and / or clarification, but let's look at the same example with our new algorithm:

```
1 - 2 - 3 - 4
    |       |
    5 - 6   7
```

`1` doesn't have any children, so we move on to `2`. `2` does have a child, so the first step is to flatten the list beginning on `2`'s child. That list is `5 - 6`, which is already flattened. So now we insert `5 - 6` between `2` and `3`:

```
1 - 2 - 5 - 6 - 3 - 4
                    |
                    7
```

We keep repeating this until we fall of through the end. The next flattening step occurs on `4`, which will flatten the list with one single element: `7`. The final result is:

```
1 - 2 - 5 - 6 - 3 - 4 - 7
```

There is still something we have to figure out though: how exactly are we going to insert list `X` between `Y` and `Z`? 

In pseudo-code, inserting `X` between `Y` and `Z` is as simple as:

```c
Y->next = X;
L = last_element_of(X);
L->next = Z;
```

In the case of our algorithm, `Y` will be the node with `Y->child == X`, and `Z` is `Y->next`. The last element of `X`, the newly flattened list, can conveniently be returned by the recursive call. This allows the parent node to obtain a pointer to the last element of the freshly flattened list as a result of recursion. Let me clarify that with some pseudo-code:

```
flatten_list(Head) {
    while (Head) {
        if (Head->child) {
            /* Recursively flatten the sub-list on the next layer */
            last_element = flatten_list(Head->child);
            last_element->next = Head->next;
            Head->next = last_element;
            /* Keep scanning the list after the last element of the newly inserted flattened list */
            Head = last_element->next;
        }
        else {
            Head = Head->next;
        }
    }
    return the node before head
}
```

The code is not very complex once the algorithm is designed. To return the node before `head`, all we need is a `prev` variable to hold the pointer value from the previous loop iteration. 

Converted to C, this turns into:

```c
struct mdl_node *flatten(struct mdl_node *head) {
  struct mdl_node *n = head;
  struct mdl_node *last, *prev = NULL;
  while (n) {
    if (n->child) {
      last = flatten(n->child);
      last->next = n->next;
      n->next = n->child;
      n->child = NULL; // Delete this line if child references are to be kept.
      n = last->next;
      prev = last;
    }
    else {
      prev = n;
      n = n->next;
    }
  }
  return prev;
}
```

Note that the function returns the tail of the new, flattened list - that's cool! The original question provides a function signature returning `void`; it is easy enough to wrap our `flatten()` inside a routine that matches the original signature and ignores the return value of `flatten()`. In fact, this is the approach taken in the [solution that I uploaded to github](https://github.com/filipegoncalves/codinghighway/tree/master/generic/MDList).
