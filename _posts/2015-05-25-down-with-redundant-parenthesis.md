---
layout: post
title : Down with redundant parenthesis
---

Removing redundant parenthesis in arithmetic expressions

-----

Redundant parenthesis are ugly: they often clutter an expression with unnecessary characters, making it larger and harder to read, with no added benefit. Excessive use of utterly unnecessary parenthesis in an arithmetic expression is annoying.

We should take care of that. The obvious question is: how? Given an arbitrary arithmetic expression, how can we remove redundant parenthesis?

My goal here is to write an expression parser that builds a new expression without redundant parenthesis, in addition to evaluating the original expression (or the new expression, it doesn't matter, since both should be equivalent).

Stop reading here. Give yourself a chance - how would you do this?

I would say that the first baby step is to get a working parser for arithmetic expressions that understands operator precedence and parenthesized expressions. This is relatively straightforward to do with a recursive descent parser.

Readers with a solid Computer Science background will probably recognize this as one of the classical problems in parser theory. Parsing arithmetic expressions is one of the very first examples in the Dragon book. The authors carefully work their way through a clever, brief and simple grammar that unambiguously recognizes an arithmetic expression. The grammar is unambiguous because there is exactly one parse tree for a given expression. Furthermore, the grammar is built in such a way that operator precedence and associativity rules are inherently part of the parse tree, which is wonderful. In the real world, this means that if we write the code carefully, any arbitrarily complex arithmetic expression can be evaluated with no auxiliary data structures - all we need is stack space (because of recursion). How beautiful is that? Yes, maybe for some of you it doesn't sound that fancy anymore, which is also great, but it's still a considerable achievement.

Here's the original grammar, taken from the Dragon book. The starting rule is `expr`. Terminals are surrounded by `'`. We don't show the expansion rule `number` - it represents any integer:

```
expr -> expr '+' term
      | expr '-' term
      | term

term -> term '*' factor
      | term '/' factor
      | factor

factor -> number
        | '(' expr ')'
```

There is quite a lot to say about this beautiful grammar; I'll leave it up to you to go dig in the Dragon book (I highly recommend it). Anyway, this is mostly what we'll need.

It is worth pointing this out though: we can't simply implement the grammar as it is shown above, at least not as a recursive descent parser. The problem lies within left recursion. Look at the rule for `expr`. It says: to parse an `expr`, first, parse an `expr`. Then, process `'+'`, and then parse a `term`. This makes sense on the paper, because it is visually easy to recognize the first (leftmost) term in an expression, and pair it with the last `expr` expansion (since `'+'` and `'-'` are left-associative, `expr` reads the expression right-to-left; the deepest nodes in the parse tree are typically on the left sub-tree, and these correspond to the leftmost subexpressions).

We can't implement it like this, because in practice, the recursion would never stop. Left recursion is usually not desirable. There are known rules to remove left recursion from a grammar. The Dragon book explains the rationale behind such manipulations: consider a left-recursive grammar, `A -> Aa | B`, where `a` and `B` are terminals or non-terminals that do not start with `A`. We can see that the recursion in `A` stops with a `B`. In fact, the grammar `A -> Aa | B` recognizes a language consisting of a `B` followed by zero or more `a`s. So, it is equivalent to `A -> B R`, where `R -> a R | e` (here, and in the next examples, we treat `e` as epsilon - the empty string). The equivalent grammar is right recursive, and it is now possible to implement it as a simple recursive descent parser.

Applying this technique to the arithmetic expressions grammar, we get:

```
expr -> term expr_r

expr_r -> '+' term expr_r
        | '-' term expr_r
        | e

term -> factor term_r

term_r -> '*' factor term_r
        | '/' factor term_r
        | e

factor -> number
        | '(' expr ')'
```

There will be exactly 5 functions that implement each rule: `expr()`, `expr_r()`, `term()`, `term_r()`, and `factor()`. `expr()` is the top-level entrypoint, and it simply calls `expr_r()` after calling `term()`. For convenience, the result of the first `term()` will be passed to `expr_r()`. `expr_r()` just has to read the next character. If it is addition or subtraction, it calls `term()` to get the second operand, then it applies the corresponding operator to the previous term (which in the first call comes from `expr()`) and the second term, and passes this result recursively to itself, until it hits the empty rule. `term()` and `term_r()` operate similarly. Finally, `factor()` looks at the next character and decides which production to apply. If it sees a parenthesis, it calls `expr()`, starting the process all over, and returning that as a result. Otherwise, it reads and parses a number, and returns it.

Here's one such implementation. Essentially, this is a calculator:

```c
#include <stdio.h>
#include <assert.h>

static int expr(const char *expr_str, size_t *cursor);
static int expr_r(const char *expr_str, size_t *cursor, int res_so_far);
static int term(const char *expr_str, size_t *cursor);
static int term_r(const char *expr_str, size_t *cursor, int res_so_far);
static int factor(const char *expr_str, size_t *cursor);

static int expr(const char *expr_str, size_t *cursor) {
	return expr_r(expr_str, cursor, term(expr_str, cursor));
}

static int expr_r(const char *expr_str, size_t *cursor, int res_so_far) {
	if (expr_str[*cursor] == '+' || expr_str[*cursor] == '-') {
		size_t saved_cursor = *cursor;

		(*cursor)++;
		int term_val = term(expr_str, cursor);

		if (expr_str[saved_cursor] == '+') {
			res_so_far += term_val;
		} else {
			res_so_far -= term_val;
		}

		return expr_r(expr_str, cursor, res_so_far);
	} else {
		return res_so_far;
	}
}

static int term(const char *expr_str, size_t *cursor) {
	return term_r(expr_str, cursor, factor(expr_str, cursor));
}

static int term_r(const char *expr_str, size_t *cursor, int res_so_far) {
	if (expr_str[*cursor] == '*' || expr_str[*cursor] == '/') {
		size_t saved_cursor = *cursor;

		(*cursor)++;
		int factor_val = factor(expr_str, cursor);

		if (expr_str[saved_cursor] == '*') {
			res_so_far *= factor_val;
		} else {
			res_so_far /= factor_val;
		}

		return term_r(expr_str, cursor, res_so_far);
	} else {
		return res_so_far;
	}
}

static int factor(const char *expr_str, size_t *cursor) {
	int val;
	if (expr_str[*cursor] == '(') {
		(*cursor)++;
		val = expr(expr_str, cursor);
		assert(expr_str[*cursor] == ')');
		(*cursor)++;
	} else {
		int read;
		sscanf(&expr_str[*cursor], "%d%n", &val, &read);
		*cursor += read;
	}
	return val;
}

// Top-level expression evaluator
int eval_expr(const char *expr_str) {
	size_t cursor = 0;
	return expr(expr_str, &cursor);
}

static char buff[512];
int main(void) {
	printf("Enter an arithmetic expression to evaluate it (no spaces).\n");
	printf("> ");

	while (scanf("%s", buff) == 1) {
		printf("%d\n", eval_expr(buff));
		printf("> ");
	}

	return 0;
}

```

# Next steps

So now that we have an expressions parser working, how can we change it to recognize redundant parenthesis?

The rule that parses parenthesized expressions is `factor -> '(' expr ')'`. There is probably something we can do here to determine whether these parenthesis are really necessary.

Let's look at a few examples. Consider the expression `3*(4*(5+2))`. We know that the first pair of parenthesis is redundant, but the second isn't. This is so intuitive that it may be hard to formalize it, but really - how do we do it?

Why do we know that the second pair of parenthesis is necessary? How did we check it?

We know because multiplication has higher priority than addition, so, `4*(5+2)` is not the same as `4*5+2`. What about `4+(5+2)`? In this case, we know that it's redundant to add parenthesis because addition is commutative and associative.

Playing with these examples leads to a simple rule:

- Parenthesis are needed if there is an operator with higher precedence than the lowest precedence operator inside the parenthesized expression either to the left or to the right of the parenthesized expression.

So, in `4*(5+2)`, the parenthesis are needed because the lowest precedence operator inside the parenthesized expression is `+`, and there is a higher precedence operator (multiplication) either to the left of `(5+2)` or to the right of `(5+2)` (in this case, to the left). In `3*(4*(5+2))`, the first pair is redundant because multiplication is commutative, and since `(5+2)` is treated as an atom (because the parenthesis are really needed, the next expression sees it as an atom element, so the operators in that subexpression are not accounted for), the lowest precedence operator in `(4*(5+2))` is `*`, and there is no higher precedence operator surrounding the parenthesis, so they are unnecessary.

We can easily keep track of which operators are seen while parsing an `expr`. This information could be returned from `expr()` back to `factor()`. Thus, when `factor()` sees an `'(' expr ')'`, after calling `expr()`, it will have a list of the operators seen in that expression. Then, it can look at whatever comes before and after the parenthesized expression, and compare it with the lowest precedence operator inside `expr` as reported by `expr()`. This is enough to know if the current pair of parenthesis is necessary. If it is determined that they are necessary, then it means the current element will be treated as an atom upper in the recursion, so we need to clear the list of operators seen for that expression. Otherwise, we keep it, because in that case it means we are removing the parenthesis, so the expression is not treated as an atom anymore and its operators may be relevant upper in the recursion (if it's inside another parenthesized expression).

There is only one tiny little detail: this won't work with things like `4-(3-2)`. The lowest precedence operator inside `(3-2)` is `-`, and there is no higher precedence operator to the left or to the right of `(3-2)`, so the algorithm says that the parenthesis are redundant - Oops! A similar problem exists with division: the parenthesis in `4/(3/2)` are not redundant.

This happens for two reasons:

- Subtraction and division are not commutative
- Subtraction and division are left associative

Note that we have to be careful with this; left-associativity implies that the parenthesis in `(4-3)-2` are redundant, whereas in `4-(3-2)` they are needed.

Again, looking at the examples, we can infer some new rules. Since subtraction is left associative, any occurrence of operator `-` to the left of a parenthesized expression makes the parenthesis necessary, unless that expression only contains operators of higher precedence (multiplication and division). Why so? Well, if the parenthesized expression contains at least one of `+` or `-`, then the parenthesis override the default associativity, and since subtraction is not commutative, they are needed, because without them, the default will be left associativity. If the parenthesized expression contains `*` and `/` only, then they aren't needed because multiplication and division have higher precedence anyway; they would always be evaluated before subtraction.

A similar rule applies to division, but since there are no operators with higher precedence than division, the rule is simpler: if there is a division to the left of a parenthesized expression, then the parenthesis are always needed.

So, putting it all together, we know that parentheses are needed when at least one of the following conditions is true:

- There is an operator with higher precedence than the lowest precedence operator inside the parenthesized expression either to the left or to the right of the parenthesized expression.
- There is a `-` operator to the left of the parenthesized expression, and the parenthesized expression contains at least a `+` or `-`.
- There is a `/` operator to the left of the parenthesized expression

Also, recall that whenever a pair of parenthesis is doomed redundant, the expression inside will not be treated as an atom anymore, and as such, the operators inside the redundantly parenthesized expression count as operators inside the outer expression. On the other hand, if the parenthesis are needed, the subexpression is now treated as an atom, and the operators inside of it are not counted to the set of operators of the outer expression.

The code that verifies and applies these rules lives inside `factor()`; more specifically, it is executed when a `factor -> '( expr ')'` expansion happens. The function for each grammar expansion rule returns a wrapper structure holding the result of the expression, along with the list of every operator that was found in that expression. Along the way, whenever redundant parentheses are found, `factor()` records the positions of such parentheses in an array, `ign[]`, (as in *ignore*), so that later we can build a new expression string by copying the original expression and ignoring the characters marked in `ign[]` (which always correspond to parenthesis).

The implementation can be found [here](https://github.com/filipegoncalves/interview-questions/blob/master/recursion_dp/redundant_parens/solution.c). Each arithmetic expression entered is evaluated, and the result is printed, along with the equivalent expression where all redundant parenthesis are removed. The result is quite elegant.
