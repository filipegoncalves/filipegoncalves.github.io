---
layout: post
title: Flattening multi-dimensional linked lists, part 1 - Parsing
---

Further ideas on parsers and flattening data structures

-----

Recently, I have been writing a lot about flattening multi-dimensional arrays. The next series of posts will be along the same lines, but this time we will be discussing multi-dimensional linked lists. 

Ok then, so what is a multi-dimensional linked list? Basically, it's a set of linked lists connected together, such that you can depict them into a multi-layer structure. Something like this:

```
1 - 2 - 3 - 4
    |       |
    5 - 6   7
```

`-` represents a pointer to the next element (as usual in linked lists), and `|` represents a connection to another "dimension". There are 3 lists in that example: the list `1 - 2 - 3 - 4`, the list `5 - 6`, and finally, the list `7`. This can be represented in a C structure in the following way:

```c
struct mdl_node {
  int val;
  struct mdl_node *child;
  struct mdl_node *next;
};
```

Where `child` is the representation for `|`, and `next` has the usual meaning in linked lists. Note that both pointers are unidirectional; this means that `2` only has references to `3` and `5`, and `5` only has a reference to `6`.

And now, the question:

> Given a multi-dimensional linked list structure, where each node can have a pointer to a child and a pointer to the next node, write a function, flatten(), that receives a pointer to the head node, and flattens the list, making it possible to print every element on the list starting on the head and just following "next" pointers.

I won't be coding this now - today, we're going to be developing a format to read multi-dimensional linked lists from input and build them according to `struct mdl_node`, which is itself a pretty nice exercise, and perhaps as complex as the question. Part 2 will cover the implementation of `flatten()`. So, can you think of a suitable textual representation to describe a multi-dimensional linked list? It should be easy to parse, and it should also be nice for a human reader - we don't want to get all confused when inputing data. I challenge the reader to think about it before moving on - this is not fun if you jump right into the solution!

The first step is to devise a representation for a regular linked list. Choosing a format to read a linked list is easy. We can go with:

```
x>y>z>...
```

`x`, `y`, and `z` represent numbers. For example, if we wanted to write the list `1 - 2 - 3 - 4`, our input would be.

```
1>2>3>4
```

The grammar that describes this is simple:

```
list ->   node
        | node '>' list
```

By the way, this article, once again, shows the grammar using a yacc-like notation. Terminals are surrounded with single quotes.

That was quick, and to the point - we don't need anything more complex than that. Now, how can this be extended to handle multi-dimensional linked lists? To the unexperienced, this may seem hard at first because we as humans are used to picturing this using a multi-layer structure that expands into multiple lines. Parsing something like that would be extremely difficult and is utterly unnecessary, it is easier to read everything in one line.

As it turns out, it is not that hard to invent some representation. In fact, we just need to insert another list after a node, in such a way that we can say if that list is a child of the current node, or if it's just what `next` points to. I chose to represent a child list within a pair of parentheses. So, now, when a node has a child, there's an opening parenthesis, the child structure is represented, there's a closing parenthesis, and finally, if the node has `next` elements, the usual representation is used.

An example should clear any doubts remaining. Let's look at our earlier list:

```
1 - 2 - 3 - 4
    |       |
    5 - 6   7
```

`1 - 2 - 3 - 4` can be represented as:

```
1>2>3>4
```

`2`'s child is `5>6`, we can say so by writing:

```
1>2(5>6)>3>4
```

`4`'s child is `7`, so our final representation is:

```
1>2(5>6)>3>4(7)
```

Note that child lists can have children themselves, the notation recursively applies to each one of them. Let's look at another example:

```
1 - 2 - 3 - 4 - 5 - 6 - 7 - 8
|       |               | 
9       28 - 29 - 30    31 - 32 - 33
|            
10 - 18 - 19 - 20 - 21 - 22 - 23 - 42 - 43 - 44 - 45 - 46
|                         |                        |    |
11 - 12 - 13 - 14        24 - 25 - 26             47   48 
      |         |              |
     15 - 16   17             27 
      |         |
     39        34 - 35 - 36 - 37  
      |         |
     40 - 41    38
```

Holy crap, what now? As I said, the notation conveniently applies itself recursively. Here's the textual representation of the above example:

```
1(9(10(11>12(15(39(40>41))>16)>13>14(17(34(38)>35>36>37)))>18>19>20>21>22(24>25(27)>26)>23>42>43>44>45(47)>46(48)))>2>3(28>29>30)>4>5>6>7(31>32>33)>8
```

Let's code the parser for this thing. Before jumping into code, we should define the grammar, so as not to lose ourselves in the way. The grammar is:

```
mdlist -> node
        | node '>' mdlist

node -> id
      | id '(' mdlist ')'

id -> integer (..., -2, -1, 0, 1, 2, ...)
```

The starting rule is `mdlist`. You should convince yourself that this grammar is correct and fully describes the notation chosen for multi-dimensional lists. Now, all that's left is, as Jon Bentley says, A Small Matter of Programming.

Parsers are generally built on top of *tokens* that are read, recognized, and returned by a token function. We will call this function `next_token()`. For this exercise, tokens can be one of `(`, `)`, `>`, and integers. For the purposes of our small program, we can assume that there is a globally known variable with enough information holding the result of the last invocation of `next_token()`. This variable must be capable of telling us what's the type of the token (parenthesis, number, etc), and its value. We can do so with a simple structure:

```c
struct mdl_token {
  enum token_type {
    NUMBER,
    OPEN_P = '(',
    CLOSE_P = ')',
    PTR = '>',
    END
  } type;
  int n; /* meaningless if type != NUMBER */
} token;
```

`next_token()` will read from `stdin` and place the new token read in `token`. We can then use `token.type` to learn about what was just read, and it if was a number, we can read its value in `token.n`. `END` is just a special token type that denotes the end of input.

With this in mind, we can write our `next_token()` function:

```c
void next_token(void) {
  int c;
  c = getchar();
  if (c == '(')
    token.type = OPEN_P;
  else if (c == ')')
    token.type = CLOSE_P;
  else if (c == '>')
    token.type = PTR;
  else if (isdigit(c) || c == '-') {
    ungetc(c, stdin);
    scanf("%d", &token.n);
    token.type = NUMBER;
  }
  else
    token.type = END;
}
```

The call `ungetc(c, stdin)` is necessary because by the time `next_token()` realizes it read a digit from input, it must push this digit back to make it available to `scanf()`. The explicit test `c == '-'` is there to support negative numbers. It is assumed that any invalid token signalizes end of input.

The parser routines are built on top of `next_token()`. The idea is to bind every I/O operation on `next_token()`, so that the rest of the parser can think about tokens rather than single characters coming from input. 

Now comes the fun part: implementing the grammar. Recall that our grammar for multi-dimensional linked lists is:

```
mdlist -> node
        | node '>' mdlist

node -> id
      | id '(' mdlist ')'

id -> integer (..., -2, -1, 0, 1, 2, ...)
```

Basically, each non-terminal is turned into a function that is smart enough to recognize which rule is selected to expand. `md_list()` will implement the logic for the expansions in `mdlist`, and `md_node()` will implement the logic for `node`. There is no need to implement an equivalent function for `id` because that's part of `next_token()`'s job.

For example, `md_list()`, which is the entry point, will call `md_node()` to read the leading node. Note that this node can be an arbitrarily complex child list, but after `md_node()` returns, we have processed `node` from `mdlist` in the grammar. Now, `md_list()` must look at the next token to decide which rule to expand - both of the options start with `node`, so we can only tell them apart when we look at the next token after processing `node`. The choice is simple: in the absence of further tokens, the first rule is chosen and `md_list()` returns; if the next token is `>`, `md_list()` recursively calls itself to process the rest of the list, and the cycle begins again. Eventually, `md_list()` will find an empty token sequence after processing `node`, reaching the base case.

The implementation for `md_node()` is similar, and it follows the grammar appearance: unsurprisingly enough, it starts by reading an integer, and then it decides whether to call `md_list()` (which in turn will call `md_node()` again) based on the next token - it will restart the cycle if the next token is an opening parenthesis. However, notice, if you will, that inside `md_node()` there is further processing to be made after `md_list()` returns: we have a closing parenthesis to read and consume (otherwise, that would be seen by the `md_list()` instance that invoked the current `md_node()` - it is very easy to screw things up in code like this).

So basically, this is what we end up with:

```c
struct mdl_node {
  int val;
  struct mdl_node *child;
  struct mdl_node *next;
};

struct mdl_node *md_node(void);
struct mdl_node *md_list(void) {
  struct mdl_node *n = md_node();
  if (token.type == PTR) {
    next_token();
    n->next = md_list();
  }
  return n;
}

struct mdl_node *md_node(void) {
  struct mdl_node *n = malloc(sizeof(*n));
  n->val = token.n;
  n->child = n->next = NULL;
  next_token();
  if (token.type == OPEN_P) {
    next_token();
    n->child = md_list();
    if (token.type != CLOSE_P)
      fprintf(stderr, "Missing close parentheses\n");
    else
      next_token();
  }
  return n;
}
```

As usual in this kind of stuff, we need a lookahead token, so that the parser knows which rule to expand next. This is assured by using the convention that both `md_list()` and `md_node()` always read one more token than they need, so that it is easy to look at the next token and take appropriate action. It is a good exercise to try to implement your own version of the parser based on the grammar I presented here, and to compare it with my code. It may look easy, or it may look scary and confusing. I must confess, had I not wrote down the grammar before implementing the parser, I would definitely get it wrong, and readers of the code would stare at it wondering how and why it magically works.

Stay tuned for the next article, where we will be covering the implementation for `flatten()`.
