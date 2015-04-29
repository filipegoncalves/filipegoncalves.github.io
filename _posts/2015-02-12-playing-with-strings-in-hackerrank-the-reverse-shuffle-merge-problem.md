---
layout: post
title : Playing with strings in Hackerrank - The Reverse Shuffle Merge problem
---

How I tackled a tough challenge on Hackerrank

-----

Hackerrank is a great resource for CS students and enthusiasts to keep their coding and algorithmic skills on top. It literally has *thousands* of problems on different CS fields, ranked by how hard they are (although sometimes not very accurately). There's just a small annoying problem though: some of the most fun and hardest exercises have terrible editorials. Problem editorials are supposed to explain the solution, but in the case of hackerrank, more often than not, editorials consist of 2 or 3 paragraphs of poor quality English with a very rough and high-level view of the solution, leaving out extremely important details. In fact, the high-level view is not that hard to grasp - the magic is how to do it efficiently. As if that wasn't enough, the author seems to think that anyone can understand his sloppy, unmaintainable, messy and obnoxious code without spending at least 2+ hours trying to figure out *What The Hell Is This*. As such, the reader is usually told to "refer to the code for further details". Yeah, right. Good luck with that.

In fact, this is what bugs me about competitive programming: it encourages bad practice and bad code. Not in the sense of inefficient code, more like unreadable code. And the problem is that it's code that will be used once, and so whoever keeps writing it is never going to experience the consequences and the nightmare of maintaining it. This is why I think competitive programming per se, without any other side projects, may be harmful. You basically write throw-away code.

Anyway, getting back on track: I was doing some practice and found the [Reverse Shuffle Merge problem](https://www.hackerrank.com/challenges/reverse-shuffle-merge):

> Given a string S, we define some operations on the string as follows:
>
> a. reverse(S) denotes the string obtained by reversing the string S. Eg: reverse("abc") = "cba"
>
> b. shuffle(S) denotes any string that's a permutation of the string S. Eg: shuffle("god") ∈ {'god', 'gdo', 'ogd', 'odg', 'dgo', 'dog'}
>
> c. merge(S1,S2) denotes any string that's obtained by interspersing the two strings S1 and S2, maintaining the order of characters in S1 and S2. Eg: S1 = "abc" & S2 = "def", one possible result of merge(S1,S2) could be "abcdef", another could be "abdecf", another could be "adbecf", and so on.
>
> Given a string S, find the lexicographically smallest string A such that S ∈ merge(reverse(A), shuffle(A)).
>
> Input format
> A single line containing the string S
>
> Output format
> A string which is the lexicographically smallest valid A
>
> Constraints:
> S contains only lowercase English alphabets.
> The length of string S is <= 10000

It's an interesting problem, and unlike many other problems, even a brute force solution doesn't immediately come to mind.

Sadly, for the reasons mentioned above, the editorial is not very helpful. I decided it may be beneficial for others to write about my solution here, hopefully making it easier to understand than the editorial.

Let's start off by trying to find relations between `S` and `A`. So, we are merging `reverse(A)` with `shuffle(A)`. `reverse(A)` is in fact a specific permutation of `shuffle(A)`, so we are essentially interspersing two permutations of the same string (while keeping the relative order of the characters in each permutation). Or, in other words, given `A`, to get `S`, we reverse `A`, then we shuffle another copy of `A` - call it `A'` - and place each character `c` of `A'` in some spot of `reverse(A)` after the position where we inserted the last character. This means that:

* If \\(n\\) is the length of \\(S\\), then \\(n\\) is even, and \\(A\\) has size \\(\frac{n}{2}\\)
* If \\(S\\) has \\(i\\) copies of character \\(c\\), then \\(i\\) is even, and there are exactly \\(\frac{i}{2}\\) copies of \\(c\\) in \\(A\\)
* `reverse(A)` is a subsequence of \\(S\\)
* consequently, \\(A\\) is a subsequence of `reverse(S)`

Suppose \\(c_i\\) is the number of occurrences of character \\(c\\) in \\(S\\). Then, we're looking for a string \\(A\\) that contains every character \\(c\\) seen in \\(S\\) exactly \\(\frac{c_i}{2}\\) times *and* is the lexicographically smallest subsequence of `reverse(S)` matching these requirements.

A brute force approach is to build the lexicographically smallest string \\(T\\) that obeys the character distribution requirements, and keep generating the next permutation of that string until it is a subsequence of `reverse(S)`.

Building the smallest string can be done in \\(\mathcal{O}(n)\\) by using a character frequency table \\(t\\) where \\(t[i]\\) holds the number of occurrences of character \\(i\\) in \\(S\\). We would then iterate through \\(t\\), and for a character \\(j\\), write \\(j\\) \\(t[j]/2\\) times into the string. 

From there, while \\(T\\) is not a subsequence of `reverse(S)`, get the next permutation of \\(T\\), and keep doing that until a subsequence is found. That first subsequence is the lexicographically smallest \\(A\\) we can have for \\(S\\). Getting the next permutation is easy in C++ and can be done with a single call to `std::next_permutation()`, defined in the header `algorithm` (the curious reader can learn about its implementation on [StackOverflow](http://stackoverflow.com/questions/11483060/stdnext-permutation-implementation-explanation).

The algorithm is correct, but it's darn slow. Checking if a string \\(T\\) is a subsequence of another is \\(\mathcal{O}(T.size())\\), and retrieving the next permutation of \\(T\\) is also linear in the worst case. Plus, we don't know how many permutations we will have to generate. In the worst case, that may be all of them. How much is that?

Let \\(s\\) be the ordered sequence of distinct characters in \\(S\\). For example, if \\(S\\) is "ajjdkeaaab", then \\(s\\) is the sequence `a, b, d, e, j, k`. Let \\(C\\) be the length of \\(s\\).
Furthermore, let \\(\sigma_i\\), \\(1 \leq i \leq C\\), be the number of occurrences of character \\(s[i]\\) in \\(S\\) divided by 2 (which gives the number of times that \\(s[i]\\) will appear in \\(A\\)). For convenience, define \\(\sigma_0\\) to be 0. Then, the number of permutations is:

$$
\prod_{j=1}^{C} {\frac{n}{2} - \sum_{i=0}^{j-1} \sigma_i \choose \sigma_j}
$$

For those not familiar with combinatorics, this may seem daunting (but believe me, it's easy), but the point is, there are quite a lot of permutations.

Side note: I always like to show the brute force solution first for several reasons. One of them is that, by nature, brute force solutions will give us the right answer, and so it helps to test the efficient, non-trivial algorithm by comparing outputs.

How can we solve this efficiently? Playing around with examples on paper is always a good idea. It can be seen that the solution can be built step by step. That is, if \\(A\\) is going to have size \\(\frac{n}{2}\\), it is possible to build \\(A\\) iteratively, starting with \\(A[0]\\), then \\(A[1]\\), etc... How is that possible? There is really nothing better than playing with a few examples on paper, but either way, one thing is true for sure: some positions in \\(S\\) are what I like to call "bottleneck" positions, since they constrain the solution. For example, if we know that \\(A\\) is going to have two `b` characters, but after position 10 in `S` there are no more `b`s, then by the time our iterative algorithm reaches position 10 in `S`, we must have written every `b` there is to write. Makes sense, right?

Generally, it would be nice if we could know which characters should have been written by the time we reach position `i`. Let's take an example. Suppose `S` is `aaeeacacbabeae`. As I mentioned earlier, `A` is a subsequence of `reverse(S)`, so first we will reverse `S`, let the result be `S'`:

```
S':
e  a  e  b  a  b  c  a  c  a   e  e  a  a
0  1  2  3  4  5  6  7  8  9  10 11 12  13
```

Consider `S'` is 0-indexed. Here's the character distribution of `S'`:

```
a: 6
b: 2
c: 2
e: 4
```

The character distribution for `A` will be:

```
a: 3
b: 1
c: 1
e: 2
```

Now, look at position 10 of `S`. There is only one `e` after that position, and the result needs two, which means that when we move to the next position (11), the resulting string by then must have at least one `e` somewhere. If it doesn't, then we will not be able to make a subsequence of `S'` that matches the character distribution constraints.

This notion can be generalized to every other position. This happens with `e` at position 10. Other positions give information about other characters. After position 12, there is only one `a`, so when we go from 12 to 13, the resulting string must have at least two `a`s. For each character, there is a position in `S` where we can learn about constraints for that character. A position `i` in `S'` holds a character, so, it will induce at most one constraint for character `S'[i]`. Thus, we can assign a number to each position `i` in `S'` denoting how many times character `S'[i]` must have been written before we go to position `i+1`. Suppose these numbers are stored in an array `needs[0..n-1]`. It's easy to understand with our example:

```
i:     0   1   2   3   4   5   6   7   8   9   10  11  12  13 
S':    e   a   e   b   a   b   c   a   c   a   e   e   a   a
needs: 0   0   0   0   0   1   0   0   1   1   1   2   2   3
```

Simply put, if `needs[i]` is `j`, then before moving to position `i+1` in `S'`, the character `S'[i]` must occur at least `needs[i]` times in the result. This auxiliary array can be built in \\(\mathcal{O}(n)\\) as follows: preprocess `S'` by building a character frequency table `freq`. Go through `freq` and divide every value by 2, to get the character distribution of `A`. Initialize another character frequency table, `freq_seen`, with 0's in every position. Then, traverse `S'` once again, but this time from right to left. For each position `j` (`j` goes from `n-1` down to 0), if `freq_seen[S'[j]] < freq[S'[j]]`, then `needs[j] = freq[S'[j]]-freq_seen[S'[j]]`, otherwise, `needs[j] = 0`. Finally, increment `freq_seen[S'[j]]`. 

So, we count how many times we've seen each character so far, and at each position, we store how many times a specific character must have been written taking into account how many times it appears afterwards.

Maybe slightly hard to understand at first, but it's nothing fancy. Why is this useful? Suppose we keep another frequency table, `written`, such that `written[i]` is the number of times character `i` has been written into the result string.

With the help of `needs`, we know exactly what is going to be the next bottleneck position. We start from `S'[0]` and find the first position `i` such that `written[S'[i]] < needs[i]`. That's a bottleneck position: before we write a character that goes beyond that position, this must hold: \\(written[S'[i]] \geq needs[i]\\). But, perhaps even more useful than that, we know we can write every character `c` lexicographically smaller than `S'[i]` for which `written[c] < freq[c]` as many times as we want, provided that:

* we don't write more than `freq[c]` of them, and
* we don't write more than the number of times `c` shows up before position `i`

Going back to our example, the first bottleneck position is `i = 5`. Let the result string be `r`; it is initialized to the empty string. Can we append an `a` to `r`? Yes, because the first `a` is in position 1 (before 5), and out of the 3 `a`s we're going to need, we have written 0. So, append an `a`. Can we append another `a`? We can, because the next `a` is in position 4 (before 5), and out of the 3 `a`s, we have written 1. So, write an `a`. Can we append another `a`? No, we can't, because now the next `a` crosses the bottleneck position. So, move on to the next character. Can we append a `b`? Yes, because out of the 1 `b` we will need, we have written 0, and the next position that has a `b` after the position of the last character we wrote is position 5, which doesn't cross the bottleneck, so we write a `b`. Here's how `r` looks like now:

```
aab
```

At this point, we have reached the bottleneck position, so we need to find the next bottleneck. Notice also that `written['b'] >= 1`, which is mandatory by the time we reach position 5.

The next bottleneck is position 8, because we haven't written any `c`, and position 8 requires that we have written at least 1 `c`. The last position where we stopped was 5, so we move on from there, and the whole process repeats: can we write `a`? Yes, because out of the 3 `a`s we need, we have used 2, and the next `a` (in position 7) doesn't cross the bottleneck position. So, write an `a`. So far, we have:

```
aaba
```

We wrote the `a` from position 7. Let's move on. Can we write another `a`? No, because all 3 `a`s have been written. But even if they hadn't, we still couldn't write an `a`, because the next `a` after position 7 is in position 9, and that crosses the bottleneck. Alright then, can we write a `b`? No, we can't: we have written all the `b`s we could, and again, even if we didn't, the next `b` after 7 crosses the bottleneck. Can we write a `c`? Absolutely! And we reached the bottleneck again, so we need to update it. The next bottleneck is 10. So far, we have:

```
aabac
```

So, we stopped on 8. The smallest character we can write without exceeding its frequency or crossing the bottleneck is the `e` from position 10:

```
aabace
```

Again, we hit the bottleneck, so we update it to 11, since that's the first position where \\(written[S[i]] < needs[i]\\) (we have written 1 `e`, and position 11 requires 2). And again, the smallest character we can write is `e`:

```
aabacee
```

`r` is now 7 characters long, exactly half of the length of `S`. We're done! The answer is `aabacee`. Remember that `S` was `aaeeacacbabeae`. Let's double check that everything adds up.

`reverse(A)` is `eecabaa`

`shuffle(A)` could be `aaacbee`

Merging these yields `aaeeacacbabeae` if the following steps are taken:

* Prepend the first 2 `a`s of `shuffle(A)` to `reverse(A)`
* Insert the third `a` of `shuffle(A)` after the 2nd `e` of `reverse(A)`
* Insert the first `c` of `shuffle(A)` after the 1st `a` of `reverse(A)`
* Insert the substring `be` of `shuffle(A)` after the substring `ba` in `reverse(A)`
* Append the last `e` of `shuffle(A)` to `reverse(A)`

And now, I challenge you to find any string lexicographically smaller than `aabacee` that could still lead to `S`. Haha! There isn't any (yes, I double checked with the brute force algorithm).

There was a lot happening here, but the core algorithm - the steps we take after building the frequency table and `needs`, are outlined below:

* Find the next bottleneck position (the next position `i` such that \\(written[S'[i]] < needs[i]\\)).
* Find the smallest character `c` that can still be written (\\(written[c] < freq[c]\\)) and its next occurrence is before `i`.
* Write `c` as much times as possible, possibly until `written[c]` matches `needs[j]`, where `j` is the current position of `c`

 I use two auxiliary structures to quickly know the next position for a given character. The first is a conceptual map associating each character to an array of positions where that character appears in `S'` (I called it `positions`). The second is an array that for each character stores how many positions I have consumed for that character, so that I don't have to remove elements from the arrays in `positions`, because removing from a vector is expensive.

The rest, as they say, is implementation details:

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

string smallest_A(string &str) {
    reverse(str.begin(), str.end());    
    
    static string::size_type freq[26];
    static vector<string::size_type> positions[26];
    static typename vector<string::size_type>::size_type next_to_use[26];
    
    for (string::size_type i = 0; i < str.size(); i++) {
        freq[str[i]-'a']++;
        positions[str[i]-'a'].push_back(i);
    }
    
    for (size_t i = 0; i < sizeof(freq)/sizeof(*freq); i++) {
        freq[i] /= 2;
        next_to_use[i] = 0;
    }
    
    vector<string::size_type> needs(str.size());
    static string::size_type freq_seen[26];
    for (string::size_type i = 1; i <= str.size(); i++) {
        
        string::size_type j = str.size()-i;
        unsigned hash = str[j]-'a';
        
        needs[j] = freq[hash]-freq_seen[hash];
        freq_seen[hash] = min(freq_seen[hash]+1, freq[hash]);
    }
    
    string res;
    
    string::size_type next_bottleneck = 0;
    string::size_type last_pos = 0;
    static string::size_type written[26];
    
    while (res.size() != str.size()/2) {
        
        while (!(written[str[next_bottleneck]-'a'] < needs[next_bottleneck]))
            next_bottleneck++;    
                  
        string::size_type next_char = 0;
        while (written[next_char] == freq[next_char] ||
               positions[next_char][next_to_use[next_char]] > next_bottleneck)
            next_char++;
        
        string::size_type str_pos = positions[next_char][next_to_use[next_char]];
        
        while (last_pos != str_pos) {
            next_to_use[str[last_pos]-'a']++;
            last_pos++;
        }
        
        do {
            res.push_back(next_char+'a');
            written[next_char]++;
            next_to_use[next_char]++;
        } while (written[next_char] < needs[str_pos]);
        
        last_pos = str_pos+1;
        
    }
    
    return res;
}

int main(void) {
    string str;
    cin >> str;
    cout << smallest_A(str) << endl;
    return 0;
}
```

And always remember to have fun!
