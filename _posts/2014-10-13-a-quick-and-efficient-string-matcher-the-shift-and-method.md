---
layout: post
title : A quick and efficient string matcher - the shift-and method
---

A dirty, hacky string matching algorithm

-----

The string matching problem is a widely studied topic. Many brain cells have been devoted to the task of solving exact string matching linearly, or, in some cases, even sub-linearly (Boyer-Moore algorithm). There's even an entire book about string matching problems, *Algorithms on Strings, Trees, and Sequences*, by Dan Gusfield, which is an amazing book, albeit slightly dense. This book is the bible of suffix trees and string algorithms. The first part deals with the exact matching problem:

> Given a string S[1..N], and a pattern P[1..M], find all the occurrences of P in S.

There are quite a number of fancy clever techniques that solve this problem linearly, with the Knuth-Morris-Pratt algorithm being by far the number one choice for most applications. Everyone learns Knuth-Morris-Pratt in school. And maybe you had the opportunity to hear about the Z-Algorithm too, and Boyer-Moore, if your algorithms and data structures courses were planned by a wise Professor. These algorithms are as good as you can get - any comparison-based method can't take any less than linear time in a worst case scenario, but come on - do you really remember all of the nitty-gritty details of Knuth-Morris-Pratt? Could you implement it right now? What if I just want a quick, dirty, although efficient, way to match a string to a pattern? Do I have to go through all of that again?

There's a good chance that most patterns are relatively small - for example, the size of an English word. When this is the case, the Shift-And method is a great candidate to solve this problem. The Shift-And method is a numerical method that solves the string matching problem by reducing it to an arithmetic problem. This algorithm is particularly interesting because its theoretical running time is as bad as the naive algorithm, but in practice it is extremely fast, possibly even faster than other complex algorithms that have better asymptotic bounds.

The Shift-And method is based off two arrays. One of them is in fact a bi-dimensional array that maps each prefix of the pattern into a bit-vector of size \\(N\\). Thus, there will be \\(M\\) bit-vectors of size \\(N\\). Let's call this array \\(A\\). We say that entry \\(i, j\\) of \\(A\\) is 1 if and only if \\(P[1..i]\\) matches \\(S[j-M+1..j]\\). In other words, \\(A[i][j]\\) is 1 if there is a match of the pattern prefix \\(P[1..i]\\) ending in position \\(j\\) of \\(S\\). The other array, which we will call \\(U\\), maps each letter of the alphabet into a bit-vector of size \\(M\\), where bit \\(i\\) is 1 if that letter is in position \\(i\\) in the pattern. So, for example, if we have the pattern `ababacdaa`, then \\(U['a'] = 101010011\\), \\(U[b] = 010100000\\), \\(U['c'] = 000001000\\), and \\(U['d'] = 000000100\\). This array will be useful to build the columns of \\(A\\).

Clearly, the problem of matching \\(P\\) to \\(S\\) can be solved by finding the last row of \\(A\\), or, equivalently, by calculating the `M`-th bit-vector. Having the `M`-th row of \\(A\\) means that we know every position in \\(S\\) where a match ends. So, once we build \\(A\\), the solution follows trivially.

Let's look at an example. Consider `S = mississippi` and `P = `issi`. Then \\(A\\) will look like this:

```
            |'m'|'i'|'s'|'s'|'i'|'s'|'s'|'i'|'p'|'p'|'i'|
---------------------------------------------------------
'i' - A[1]  | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 1 |
---------------------------------------------------------
's' - A[2]  | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
---------------------------------------------------------
's' - A[3]  | 0 | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
---------------------------------------------------------
'i' - A[4]  | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 |
---------------------------------------------------------
```

As you can see, the solution lies in \\(A[4]\\), where we get to know that there is a match ending in position 5 and another ending in position 8 of \\(S\\) (and so, the output would indicate that there are two matches starting at positions 2 and 5 - we just subtract the pattern length and add 1).

With this in mind, we are ready to build \\(A\\). As it turns out, it is easier to build \\(A\\) column by column, rather than the intuitive row-by-row approach. This becomes clearer if we look at each column as a bitwise encoding of the active state in a finite deterministic automata representing the pattern. A trivial automata for a pattern of size \\(M\\) has \\(M+1\\) states. State 1 corresponds to the initial state, where no character has been processed. According to this reasoning, bit \\(j\\) of a column of \\(A\\) is 1 if state \\(j+1\\) of the automata is active at that point.

For example, consider the eighth  column in the above table. It is `1001` (reading top to bottom). This means that if we were to match the text against the pattern, we could either be in state 2 or state 5 of the automata. State 5 is an accepting state, since we have consumed every letter of the pattern by then, so this is a match. Note that this is not exactly how a DFA works; if we want to be pedantic, no more than 1 state should be active at any given moment in a DFA. To be accurate, we should say that a column of \\(A\\) actually represents the set of all possible active states if we start from the beginning on each column and at the same time keep track of the previous active state.

How can this help? We know for sure that a state \\(q\\) is active if and only if state \\(q-1\\) was active and the next character in input corresponds to a transition from \\(q-1\\) to \\(q\\). Looking at our trivial automata, this is equivalent to saying that if we were on state \\(q-1\\) and the next character from the text appears on position \\(q-1\\) of the pattern, then there is a valid transition to state \\(q\\). Or, if you don't like the automata approach, think of it like this: the only way that \\(A[i][j]\\) is 1 is if \\(A[i-1][j-1]\\) was 1 (meaning that the pattern so far matches up to position \\(i-1\\)) AND \\(P[i] = T[j]\\).

So here's how we can build column \\(j\\) out of column \\(j-1\\): we shift column \\(j-1\\) down by 1 bit, and then `AND` it with \\(U[T[j]]\\). Confused? This is the exact meaning of what I said in the previous paragraph: if we do this, bit \\(i\\) of column \\(j\\) will be 1 if bit \\(i-1\\) of column \\(j-1\\) was 1 and \\(T[j]\\) occurs in position \\(i\\) of the pattern. This is why we need \\(U\\) - it's just a quick way to enforce the second part of the rule - it essentially clears out invalid transitions on the automata. There's only a small fix worth mentioning: when shifting down, the new bit that "slides in" must be a 1, instead of the traditional 0. We need a 1 because at each step we are also attempting to reevaluate the automata from the beginning, which means that state 2 is possibly active. It may be the case that it shouldn't be (if the first character of the pattern doesn't match \\(T[j]\\)), but in that case, the `AND` will clear it out. If we don't slide in a 1 for each shift, we would be missing occurrences.

The first column is explicitly obtained by comparing \\(P[1]\\) to \\(S[1]\\); if they match, then the first bit is a 1, otherwise, the first column consists of 0's. 

Notice that there are at most \\(\mathcal{O}(nm)\\) bit operations, which makes this algorithm as bad as the naive approach. However, even though both share the same asymptotic bounds, bitwise operations are extremely fast, the memory usage for this method is very low - especially considering that we don't need to keep \\(A\\) in memory, since each column can be built from the previous one - and finally, if the pattern is at most the size of a CPU-word, this can all be done with 2 integers (excluding \\(U\\), of course). All of these practical implementation details make the Shift-And method a viable and very appealing choice that is easy to implement and reasonably fast for small to medium-sized patterns. The short and concise implementation is a huge plus:

```c
#include <stdio.h>
#include <string.h>
#include <limits.h>
#include <assert.h>
#define MAX_PATTERN_SZ (sizeof(unsigned long long)*CHAR_BIT)
#define MAX_TEXT_SZ 256

void string_matcher(const char *text, size_t text_sz, const char *pattern, size_t patt_sz)
{
	unsigned long long letters_pos['z'-'a'+1] = { 0 };

	assert(0 < patt_sz && patt_sz <= MAX_PATTERN_SZ && text_sz > 0);

	for (size_t i = 0; i < patt_sz; i++)
		letters_pos[pattern[i]-'a'] |= 1ULL << i;

	unsigned long long col = text[0] == pattern[0];

	for (size_t i = 1; i < text_sz; col = (col << 1 | 1) & letters_pos[text[i]-'a'], i++)
		if (col & 1ULL << (patt_sz-1))
			printf("Occurrence starting at S[%zu]\n", i+1-patt_sz);

	if (col & 1ULL << (patt_sz-1))
		printf("Occurrence starting at S[%zu]\n", text_sz-patt_sz+1);
}
```

For convenience, a column is mapped top-to-bottom into a 64-bit integer from least significant to most significant bit. Strictly speaking, this code supports patterns of at least 64 characters, since that's the minimum width of `unsigned long long`, but different machines may have wider types. It also assumes that the alphabet consists of every lower-case letter from `a` to `z`, and that the platform uses an ASCII-like encoding where these characters appear consecutively. Here's a simple `main()` body to perform some interactive tests:

```c
int main(void)
{
	char text[MAX_TEXT_SZ+1];
	char pattern[MAX_PATTERN_SZ+1];

	while (1) {
		printf("Text (max. %d chars): ", MAX_TEXT_SZ);
		fgets(text, sizeof(text), stdin);
		size_t text_sz = strlen(text);

		printf("Pattern (max. %zu chars): ", MAX_PATTERN_SZ);
		fgets(pattern, sizeof(pattern), stdin);
		size_t patt_sz = strlen(pattern);

		if (text[text_sz-1] == '\n')
			text_sz--;
		if (pattern[patt_sz-1] == '\n')
			patt_sz--;

		string_matcher(text, text_sz, pattern, patt_sz);
	}

	return 0;
}
```

The implementation can be extended to deal with arbitrarily long patterns if instead of `unsigned long long` we use `char` arrays with the correct number of elements - for a pattern of at most \\(M\\) characters, we need at most  \\(\frac{M-1}{CHAR\\_BIT}+1\\) chars to store a column, where `CHAR_BIT` is the number of bits in a char (defined in `limits.h`). Of course, the code will not be as elegant, and depending on how big \\(M\\) is, this method loses advantage. Nevertheless, it's a nice algorithm to have in your pocket.
