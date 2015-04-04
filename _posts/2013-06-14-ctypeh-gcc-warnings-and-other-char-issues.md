---
layout: post
title: ctype.h, gcc warnings, and other char issues
---

GCC Warnings, Unicode, strings, and internationalization issues

-----

About a month ago, I started working on my own solutions for K&R exercises. I thought I knew C until I started reading K&R, and doing the exercises. Doing the exercises is an excellent way of learning even more beyond what the book teaches. It has definitely been a tough time since then, because some exercises are a little bit harder, and I want to make sure I understand everything that is happening with my code.

Solving K&R exercises is something that everyone should do. It changes you. It makes you a better person, a better professional. It changes the way you think, the way you code. It's very good.

Now, I like to get everything done in the best possible way, and it has always been my policy to compile my code with the highest warning level and to treat warnings as errors, meaning, I do not write one more single line of code until gcc compiles my source file with no warnings. Compiler warnings should not be ignored, they normally give useful indications about where the code is most likely to break.

So today I want to look at a warning with which I wasn't entirely familiar: 

> warning: array subscript has type `char`

This happens on every line of the following code that calls `isdigit()`:

```c
int getch(void);
void ungetch(int);

int getop(char s[]) {
	int i, c, t;
	while ((s[0] = c = getch()) == ' ' || c == '\t');
	s[1] = '\0';
	if (!isdigit(c) && c != '.' && c != '-') {
		if (c == '&') {
			s[0] = getch();
			return ASSIGN;
		}
		else {
			return c;
		}
	}
	i = 0;
	if (c == '-') {
		if (isdigit((t = getch())) || t == '.') {
			s[++i] = t;
			while (isdigit(s[++i] = c = getch())); // HERE
		}
		else {
			ungetch(t);
			return c;
		}
	}
	if (isdigit(c))
		while (isdigit(s[++i] = c = getch())); // HERE
	if (c == '.')
		while (isdigit(s[++i] = c = getch())); // HERE
	s[i] = '\0';
	if (c != EOF)
		ungetch(c);
	return NUMBER;
}
```

I know this code can look intimidating to someone who's not familiar with K&R exercises and examples. This function, `getop()`, is used as part of an RPN calculator that the authors of the book implement on chapter 4, section 3 (pages 76, 77 and 78). This is not the orignal code, it was modified to solve the proposed exercises. I know, I know, this code has all kinds of stuff that make it a nightmare for maintenance, like:

* Meaningless, one-letter variable names that say nothing about what they store
* Lots of instructions with side-effects
* Expressions that force you to know precedence rules (e.g. `s[0] = c = getch() == ' '`)
* Loops with no body that do everything in a hard-to-read loop condition
* `if` without opening and closing braces
* Variables are reused for different purposes (`i` is reset to 0 at some point for later reusage)

But this code is for educational purposes, and it's a book about learning C, so we might as well force ourselves to learn precedence rules and other nitty-gritty details of C by writing code like this.

Here's the function prototype for `isdigit()`:

[code language="cpp"]
int isdigit(int);
[/code]

Notice that `isdigit()` receives an `int` argument. Why? Well, according to the standard, what it really expects is some `int` with a value in the range of an `unsigned char` or `EOF`. And why is that? Because `EOF` is guaranteed to be a negative value, and because some machines may use a `signed char` representation, there would be no way to tell the difference between `EOF` and a char with a negative value that could happen to be the same as `EOF` (remember, `EOF` is just a constant). This is the reason why every `ctype.h` function and `stdio.h` functions such as `getchar()` expect `int` as arguments instead of `char`.

What type is the argument that we're passing to `isdigit()`? It's a `char`.

We're calling `isdigit()` with the result of `s[++i] = c = getch()`.

If you read K&R, or, for that matter, if you know C well enough, you'll know that an assignment is an expression with a defined value like any other expression. The value of an assignment is the value of the left hand side after the assignment. And, in this case, since `s` is a `char` array, `s[++i]` is of type `char. So we're basically assigning the an `int` (the variable `c`) to a `char`, and the final expression is of type `char`.

Now, there's nothing wrong with assigning an `int` to a `char` - some bits are chopped off and that's it. The problem arises when a `char` is converted back to `int`. You see, `char` has less bits than `int`, which means that in order to perform the conversion, the `char` needs to be extended up to the size of `int`. And that's where things can go terribly wrong. Some machines can simply add 0 bits on the left, while others may choose to extend the sign. This means that, for example, every `char` with the most significant bit set to 1 may eventually be extended with 1's, yielding a negative number. 

This fact, together with the fact that `char` can be either signed or unsigned in a particular machine, makes indexing an array with `char` extremely dangerous. If the platform's native `char` type is backed by `signed char` and an array is indexed with a `char`, there is a chance of indexing a negative position. Oops! This what the gcc warning is all about.

And then comes the fun part. I kept looking at my code and wondering why on earth gcc was warning me about it. Because the warning talks about indexing arrays, I pretty much convinced myself that the problem was in `s[++i]`. That's the place where I'm indexing, right? But, what the hell, `i` is of type `int`, as well as `c`.

It took me a few minutes to find out that the problem is not in `s[++i]`. The warning is because of how `isdigit()` is implemented. Here's an interesting comment in the source code that implements `isdigit()`:

```
These macros are intentionally written in a manner that will trigger
a gcc -Wall warning if the user mistakenly passes a 'char' instead
of an int containing an 'unsigned char'.  Note that the sizeof will
always be 1, which is what we want for mapping EOF to __ctype_ptr__[0];
the use of a raw index inside the sizeof triggers the gcc warning if
__c was of type char, and sizeof masks side effects of the extra __c.
Meanwhile, the real index to __ctype_ptr__+1 must be cast to int,
 since isalpha(0x100000001LL) must equal isalpha(1), rather than being
an out-of-bounds reference on a 64-bit machine.
```

Haha! We're on to something! It turns out that `ctype` functions are not that easy to implement. `isdigit()` is surely *not* a simple `return c >= '0' && c <= '9'`.

That surely works on a machine using ASCII charsets, but it may not work in other charsets. For example, in [EBDIC](http://en.wikipedia.org/wiki/EBCDIC), the test `c >= 'a' && c <= 'z'` will not be enough to know if `c` is a valid alphabetic char. That's what the `ctype` header is for: it defines `isdigit()`, `isalpha()`, etc, in a portable manner that everyone can rely on.

The thing is, these functions were implemented using a lookup table method. You can think of it as a big array with each position corresponding to a known char, and for each position there's a set of bits that have different meanings. For example, the position that corresponds to the code of `'0'` will have the bit that indicates whether this char is a digit set to 1.

So, really, what `isdigit()` is doing, is indexing an array with whatever it gets as an argument, and making some bitwise operations to check appropriate bits in that position of the array. And because we're passing a `char` to `isdigit()`, we are risking ouselves to force `isdigit()` to index a negative array position and blow everything up.

How can we solve it? Well, a good solution would be to make `s` an `int` array, but I didn't do it because this function is called in lots of other places. And it would be wasting memory anyway, because we never really store `EOF` in this array, we're only storing char values. 

This is the kind of problem that can be fixed by using casts. Whenever you see casts, you should be very pedantic about it and really make sure that they're not messing up your code (I am talking about things like casting `void*` to `int`, or some ugly variant of that). But this time, we really should use a cast to `unsigned char`. Why isn't this wrong? Casting `char` to `unsigned char` is not a big deal. No information is lost by doing this. The only thing that changes is that after the cast, the `char` will be interpreted as `unsigned char`, so, if it was a negative char, now it's some (high) positive value. For example, a `signed char` with a value of `11011101` (\\(-34\\) in decimal), is interpreted as decimal \\(221\\) after the cast. The bit pattern is the same, the interpretation is not. So, by casting to `unsigned char`, `isdigit()` will now interpret it as a positive number, eliminating the possibility that it will index a negative position, whether or not the machine's native char typ is signed or unsigned.

However, I didn't make the cast. If `c` is an `int`, why not passing it directly to `isdigit()`? It makes no sense to convert a valid value for `isdigit()` into a char just to assign it to the array and then converting it back to call `isdigit()`. So, what I did was removing the assignment to `s[++i]` from the loop condition. Every loop turned into this equivalent form:

```c
while (isdigit(c = getch())) {
	s[++i] = c;
}
s[++i] = c;
```

This fixed the warning. Note that the final assignment after the loop is needed - the original code actually performed the assignment to `s[++i]` before calling `isdigit()` inside the loop condition, so, even when `isdigit()` returned `false`, the assignment was already done by that time.

### Indexing arrays with char

Let's look at an exercise: imagine you are to write a function that receives a string to be scanned and a string with delimiters. You want your function to count the number of characters (excluding the delimiter character) in the first string until the first occurrence of one of the delimiters. If there are no delimiters in the target string, the function should return -1. Let's call it `charcount()`. So, for example we have:

```c
charcount("hello", ".,-!") == -1;
charcount("hello, world!", ".:-,!") == 5;
charcount(".:-,!", "hello, world!") == 3;
charcount("", ":,-") == -1;
charcount(":,-", "") == -1;
```

A naive solution is:

```c
int charcount(char *target, char *delims) {
	int delimsIndex;
	char *original = target;
	while (*target) {
		for (delimsIndex = 0; delims[delimsIndex]; delimsIndex++) {
			if (*target == delims[delimsIndex]) {
				return target-original;
			}
		}
		target++;
	}
	return -1;
}
```

The problem with this solution is that for a target string of size `n`, in the worst case this code will be scanning the delimiters string `n` times, meaning this is \\(\mathcal{O}(n^2)\\). A good programmer will notice that this can be done in linear time if we create some kind of "buffer", or "hash" of our delimiters string in the beginning, allowing the inner loop to be replaced by a constant time check to see if the current character is a delimiter. The most straightforward way to do this is to create an array indexed by `char` value, initialized to 0 in positions whose key is not part of the delimiters string, and initialized to 1 in positions whose key is a delimiter. I am talking about something like this:

```c
int charcount(char *target, char *delims) {
	short delims_map[256] = { 0 };
	char *original = target;
	while (*delims) {
		delims_map[*delims] = 1;
		delims++;
	}
	while (*target) {
		if (delims_map[*target]) {
			return target-original;
		}
		target++;
	}
	return -1;
}
```

That surely looks nicer. But again, gcc complains about `delims_map[*delims]`. And notice the size of the array - why 256 positions? It's ugly to hardcode values like this in your code.

Instead, to ensure code portability (and also to fix the warning), we will have to make sure that our chars will not index negative positions. Like I said before, this can be done with casts to `unsigned char`. But what size should our array be? Well, it must be able to hold any possible `unsigned char`, so it must be as long as every value representable by unsigned char. Type ranges can be read from `limits.h` header; the limit for `unsigned char` is the constant `UCHAR_MAX`. So our code should look like this:

```c
#include <limits.h>
int charcount(char *target, char *delims) {
	short delims_map[UCHAR_MAX] = { 0 };
	char *original = target;
	while (*delims) {
		delims_map[(unsigned char) *delims] = 1;
		delims++;
	}
	while (*target) {
		if (delims_map[(unsigned char) *target]) {
			return target-original;
		}
		target++;
	}
	return -1;
}
```

Little question: what if we didn't have `limits.h`? Or, put it another way, how is `UCHAR_MAX` calculated?

The max. representable value `unsigned` type is just setting every bit to 1, right? Then `UCHAR_MAX` is just something like `((unsigned char) ~0)`.

The solution we came up with for charcount is pretty acceptable. It's clean and portable, but what if this code had to deal with international strings? This brings me to the next point.

### Charsets, locales and unicode

There seems to be a lot of confusion around these terms, so, before moving on to other char issues that might arise, let's try to learn what is this all about (you can also read [Joel's article about unicode](http://www.joelonsoftware.com/articles/Unicode.html)).

So what is a charset? A charset is nothing more than a mapping from characters to numbers. That's it. That's all you have to know. As you can imagine, different charsets consist of different mappings. I can have a charset where `a` is represented by 1, `b` by 2, `c` by 3, etc. Or I can have something like ASCII, making `a` 97, `b` 98, `c` 99, etc. Or I can have `a` being 21, and `b` represented by 1. Although I think no such thing exists, in theory, your binary representation doesn't have to be ordered the same way as the alphabet.

What about *locale*? A locale is a set of rules. These rules define things like how a date and time should be formatted, what letters are considered upper case, what letters are considered lower case, what is the correspondence between a lower case and an upper case letter, what is a valid line or sentence terminator, etc. A locale is normally chosen on a per-user basis, and it consists of a language and region identifier. A locale also includes a charset reference that indicates which charset is used.

And then, finally, we have unicode. Unicode was created as a response to the need for a standard that explicitly lists all characters from every language. Think of it as a logical charset (though it's much more than a standardized charset). Unicode is a logical mapping between a letter and something called <em>code point</em>. The code point is just a hexadecimal number varying from `0x000000` to `0x10FFFF`. 1,114,112 characters can be represented using Unicode code points. 

For example, the copyright symbol is represented by the hexadecimal number `00A9`. Unicode is not just a mapping, it defines a whole set of rules that state, for example, lower case and upper case equivalence, which letters are title letters (there are languages and alphabets with special letters that can only be used to start a sentence or paragraph), what is considered a space, a line break, what combining characters are available (letters with accents can have more than 1 way to be represented; there's a single code point, but there's also a code point for the base letter and the accent, so you can combine code points). Unicode is the product of a huge effort towards standardization that took a lot of time to gain shape and be useful. 

Remember, unicode is a *logical* mapping. In other words, it's a concept, and because it's a concept, there are several ways to *implement* that concept. This means that the act of actually representing unicode characters is, itself, another process. It's completely different. There's UCS-2, which represents everything as 2 bytes, there's UTF-16, where characters are represented by a variable number of bytes, there's UTF-8, using 1 up to 4 bytes for a character, there's UTF-32. There are a dozen ways and charsets that implement Unicode. The take home message is that Unicode is used by several charsets, which in turn are referred by locales.

The bottom line? Well, if you want your program to be portable, you cannot assume anymore that a character is something greater than or equal to `'a'` and less than or equal to `'z'` (or their upper-case equivalents). And you can't assume it's represented by 1 byte. You have to know what charset is being used and make sure your code works in a portable, smooth manner.

UTF-8 gained a lot of popularity in the Web and is the most common way to represent unicode characters. It has the nice feature that every original ASCII character less than or equal to 127 is represented by the same value in UTF-8, and higher characters are represented by a variable number of bytes (up to 4). This is cool because you do not waste space using too much bytes to represent something that would otherwise perfectly fit in 1 byte; you really only use more bytes when there's a need for that. UTF-8 can represent all of the 1,114,112 characters defined by unicode.

### Impacts in C code
The standard C library provides a couple of functions to deal with international strings. Strings containing input you get from the user will be encoded with the user's locale charset, and the strings passed to `printf()` must be encoded in the locale's charset as well. Inside your program, you are free to encode strings the way you want, but if you want to print them, you may have to convert them to the proper charset.

There are two definitions to keep in mind:

*Multibyte string* - This is a string represented in a particular charset encoding. It can be UTF-8, but it can be any other encoding that doesn't use multibyte representation (in fact, strings where a char is 1 byte can also be called multibyte strings. I know, it's an unfortunate name).

*Wide character string* - Have you ever heard about [wchar_t](http://en.wikipedia.org/wiki/Wide_character) type in C? It's a type of character that is normally a 32-bit integer used to hold unicode code points. Modern Unix based systems normally require strings in `wchar_t` to be encoded using [UTF-32](http://en.wikipedia.org/wiki/UTF-32). This is good because it lets you deal appropriately with international unicode strings, but it's bad because if your code only uses 8-bit traditional ASCII characters, it will waste 24 bits for each character (though space is not at a premium anymore, desktops have lots of memory nowadays, but still, it's a sad feeling!). However, don't just assume that `wchar_t` is enough to hold unicode code points. Compilers sometimes make it only 16-bits, which is not enough to represent every unicode value. Windows is an example where this happens. The `wprintf()` function allows you to print arrays of `wchar_t` - there are equivalents for almost any I/O function that uses `char`. There's `towupper()` and `towlower()`, and a dozen more for you to see in `wchar.h`. Oh, and there are locale specific functions, like `strcoll()`, that provides sorting for international strings, or `strftime()`, that formats a date and time according to the current locale. These are all defined in `locale.h`, because they depend on which locale is being used (in order to be able to use these functions, you must initialize them with `setlocale`). Wide character strings can be hardcoded in a C program by prefixing the string constant with `L`, as in `L"Hello, World!"`.

Ok, so, how can you make code that behaves properly with international strings? It's a whole new world, and there's not a definite answer to that. The first thing to keep in mind for production code is that `char` is not a character anymore. If you are reading a file encoded with UTF-8 containing international characters and you assume that it's merely a bunch of chars, you are simply ignoring the variable length representation that UTF-8 uses, and you will interpret the file's content in a very different way. It's a disaster.

Let's choose a charset. I'll go with UTF-8, because it's the most popular nowadays. Many users have their locale set to UTF-8, it's used on the web, it doesn't waste much space and it can represent every unicode character. Where's the catch? Well, it turns out that you can't do string indexing in constant time anymore. Because of the variable length feature, you do not know exactly where character `i` lies. Indexing is a linear time operation.

Assuming your program uses UTF-8 for internal string manipulation, two things can happen. Either the user's locale is UTF-8, which is brilliant, or it is not. How do you know the current locale's charset? What if he doesn't use UTF-8 and I'm using UTF-8? Well, then you are responsible for doing the right conversions.

Remember, a good software product must pay attention to this kind of stuff. As a user, it is really annoying to read those mangled weird chars that show up when the underlying program is not smart enough to do proper string processing. It's your choice and responsability to ensure that this will not happen.
