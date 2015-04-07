---
layout: post
title: scanf survival guide
---

On the nitty-gritty details of scanf() and why you should learn it

-----

`scanf()` can be a real pain in the ass for someone who doesn't know how it works, and it can literally look like magic when someone who understands it uses in front of you. Let's pick a simple example. Imagine you are to read a set of dates from standard input, one date per line, where each line has the form:

```
Date: dd/mm/yyyy;
```

You are told that the input is always well formed, so you wisely decide to use `scanf()` to pick the day, month and year values. You might be tempted to write something like this:

```c
#include <stdio.h>

int main() {
  int m, d, y;
  while (scanf("Date: %d/%d/%d;", &d, &m, &y) != EOF) {
    /* date stored, proceed to further processing... */
  }
  return 0;
}
```

Surprisingly enough, this doesn't work. It will read the first date, and after that it will loop forever without actually reading any more dates. Sounds weird? Not if you know how `scanf()` works. It's not magic, believe me (and, for that matter, there's nothing magic about magic, it's yet another example of seeing someone mastering an art that you never learned. Once you know the tricks, there's no more magic). If you read this article, it will be crystal clear for you why this code behaves the way it does.

The key to success with `scanf()` is understanding a simple, yet often underlooked, concept: *buffered input*. Buffered input means that the input is stored in a buffer - easy. The characters read from input are placed in a sequential buffer, using a FIFO (Fist In, First Out) policy. For example, if there is `alfdh` in input, then the buffer will contain `alfdh`, and after you read the first `a` from the buffer, it will contain `lfdh`, and the next one to be read is `l`, and so on.

`scanf()` works with this buffer to read the input, and as long as it matches what is expected, we're good. Now, because no one can predict the future, `scanf()` will sometimes pick the next character in the buffer and find out that it shouldn't have read it, so it kind of cancels this last read. For example, when you provide a `%d` modifier to `scanf()`, indicating that an integer is expected, what happens is that `scanf()` doesn't know how long your number will be, so it just keeps reading digits from the input until eventually something that is not a number (for example, a space) shows up. At that point, `scanf()` has to pretend this space was never read and push it back to the input, since you told it to read a number only.

Imagine the following buffer:

```
5568 is the line number
```

If you call `scanf("%d", &number)` with this input, scanf will read `5`, `5`, `6`, `8`, and space. At this point, it realizes it has read too much, and it sends this space back to the input buffer, leaving it untouched for the next I/O operation.

What can we learn from this example? Something that is very, very important, and that you can never forget: `scanf()` will push a character back to the buffer when it's an unexpected character. The character is pushed back to the beginning of the buffer, and **subsequent calls to `scanf()` will read this same character AGAIN**, because it's the first one in the buffer. This is *the* key concept to understand everything about `scanf()`, along with some specific behavior descriptions that I'll show shortly. In the previous example, after calling `scanf()` to read the number, our input buffer would now look like:

```
 is the line number
```

Note the leading space. If you can understand this, we can now move forward and talk about specific `scanf()` behavior. Its first argument is a formating string, which basically accepts:

* Blanks (tab, space, newline, formfeed, and any other character for which `isspace()` from `ctype.h` returns true
* Ordinary characters (not `%`)
* Conversion specifiers, which are identified by a leading `%` (to insert a literal `%`, just escape it with another `%`, like so: `%%`)

That's not very hard, is it? From all possible characters you can put in the format string, you can quickly see which group they fit in. Let's see what each of these characters do. 

Blanks are very easy to understand. The rule is that a blank will force `scanf()` to consume every character that is considered a space (again, for which `isspace()` returns true) until a non-space character or `EOF` is found, even if that means consuming the whole input buffer and waiting for more to arrive. It will only stop when a non-blank character (or `EOF`) is received. This means that `scanf()` has the ability of crossing lines to read its input. This can be very useful, but it can also be a huge drawback, depending on your needs.

Ordinary characters are also very easy to understand. When faced with an ordinary character, scanf will attempt to match it to the next character in the input buffer. So, for example, `scanf("Hello")` will attemmpt to match an `H`, followed by `e`, followed by `l`, followed by another `l`, and finally followed by `o`. If the input buffer fails to match this set of rules at some point, `scanf()` will push the problematic character back to the buffer and return prematurely. Imagine that the buffer was `Helo`. After reading `Hel`, an unexpected `o` is read, so it is pushed back to the buffer, leaving us with `o` in the input, and with `Hel` consumed. This is a very important fact about `scanf()` that I will stress out later in this article.

Last, but not least, conversion specifiers are special sequences that `scanf()` interprets in order to conveniently convert some input patterns for you. For example, `%d` will attempt to read a number and convert it to `int`, `%f` will do the same for `float`, `%lf` for `double`, etc. I'm not going to talk about every modifier mode and usage, for that, please refer to the manpage. However, it is important to note that any modifier has an implicit space in the beginning to ignore non-blanks, so `scanf("%d", &a)` and `scanf(" %d", &a)` - with a leading white space in the format string - is exactly the same thing. There is one exception though: the `%c` modifier, which grabs the next character in the input, will really grab the next character, whatever that is: space, newline, tab, number, an alphabetic character, you get the point. What if you want `%c` to grab the next non-blank character? Well, `scanf(" %c")` will do it, because the leading white space will consume everything until something that is not a space is read, and `%c` will catch it.

**This is very important**: `scanf()` returns as soon as an unexpected character is read. And remember, because unexpected characters are pushed back to the buffer, the next call will read this same character again, this will keep happening until something matches it (and, consequently, consume it). And what does it return? `scanf()` always returns the number of successfull conversions and assignments made, or `EOF` when a read error has ocurred (often, this happens when end of file is reached, so the input stream is finished). For example, if `scanf("%d", &a)` is able to match a number and assign it to `*a`, it will return `1`. `scanf("%d %d", &a, &b)`, or the equivalent form `scanf("%d%d", &a, &b)` will return at most `2`, when both `a` and `b` were assigned. `scanf()` with string formatters without modifiers, such as `scanf("Hello")`, always returns `0`, because there are no assignments and conversions to make (or it can also return `EOF` if nothing could be read). `EOF` is defined to be a negative constant.

With this set of rules, you now have the basic tools to fully understand `scanf()` behavior that would otherwise look cryptic. Let's go back to our loop example:

```c
#include <stdio.h>

int main() {
  int m, d, y;
  while (scanf("Date: %d/%d/%d;", &d, &m, &y) != EOF) {
    /* date stored, proceed to further processing... */
  }
  return 0;
}
```

So what's happening here? Let's imagine a sample file. It can be like this:

```
Date: 01/07/2013;
Date: 02/07/2013;
Date: 03/07/2013;
```

When you read from `stdin`, the shell will tipically pass input data to the waiting program when you press return (this is a different behavior from when you read a file, in which case the underlying standard library implementation will read the file in blocks of size `BUFSIZ` - yes, `BUFIZ`, I didn't forget an `E` in the end - declared to be a value that is optimal for the filesystem. `BUFSIZ` is defined in `unistd.h` for POSIX compliant systems, which in turn is included by `stdio.h`). So, after you enter the first date, here's the buffer's content:

```
Date: 01/07/2013;\n
```

Let's see what happens with `scanf()`. We called it like this:

```c
scanf("Date: %d/%d/%d;", &d, &m, &y);
```

First of all, notice that the space after `:` is useless, because `%d` already incorporates an implicit white space in the beginning, so, instead, we could have made it like this:

```c
scanf("Date:%d/%d/%d;", &d, &m, &y);
```

Also, note that because of this, things like `Date: 04/   08/\n\n\n/2013;` are allowed.

Either way, `scanf()` will read everything in the buffer, because it can match everything, and it will stop with `\n`, which is pushed back. This leaves our buffer with a single newline character:

```
\n
```

In the next call, `scanf()` tries to match the first `D`, and grabs the first character in the buffer. It's a `\n`, and because it doesn't match `D`, it is pushed back to the buffer and `scanf()` returns immediately. In the next call, `scanf()` pops the same `\n`, tries to match against `D` again, with no success, and thus pushes back `\n` and returns. And this keeps happening forever. The values stored in `d`, `m` and `y` never change, so you will be processing the same date over and over again forever.

How can we fix this? Well, we just need to insert a blank either at the beginning or at the end of the format string, to force `scanf()` to consume blanks. This will do it:

```c
#include <stdio.h>

int main() {
  int m, d, y;
  while (scanf(" Date: %d/%d/%d;", &d, &m, &y) != EOF) {
    /* date stored, proceed to further processing... */
  }
  return 0;
}
```

You can put spaces both at the beginning and at the end of the format string, but that's redundant. However, putting it at the beginning allows for spaces and newlines to occur before the first date line comes. Either way, with this code, you can see that the leading space makes a hell of a difference, because now the code does what it's supposed to do. Furthermore, dates can now be separated by any kind and number of blanks. This will be correctly processed:

```
Date: 01/07/2013;
Date: 02/07/2013;
Date: 03/07/2013;
```

As well as this:

```
Date: 01/07/2013;





Date: 02/07/2013;        Date: 03/07/2013;
Date: 04/07/2013; Date: 05/07/2013;
```

Whether or not that's your desire depends on your situation, but we will look at input validation later in this article.

Let's play a little bit more with `scanf()`. What happens if the input is: 

```
D
Date: 12/02/2013
```

That is, I write a `D`, then I tap return, and then I write a well-formed entry. Well, it turns out that `scanf()` will successfully match a `D`, but then instead of an `a` it will read a newline character. The character is pushed back to the buffer, and this `scanf()` call returns prematurely, and your program will enter the loop body with meaningless old values in the variables `m`, `d` and `y`. In the next loop iteration, `scanf()` ignores every `isspace()` character, which happens to be a single newline in this case, until the other `D` is reached. This time, the whole input data is successfully matched and `m`, `d` and `y` hold fresh new values for the last entry read.

From this, you can learn that in this example program the only way to get out of the loop is to reach `EOF` in the input stream; typing random characters will only make `scanf()` return 0 and go back to another loop iteration.

Also, before processing dates inside the loop body, we must be sure to check the return value of `scanf()`, because we can only process a new date safely if `scanf()` returned 3.

For a better understanding of how `scanf()` works, I find it really instructive to provide a small, rudimentary implementation. I developed this while solving K&R exercises. My `scanf()` implementation is called `minscanf()` because, well, it's a minimal implementation. It's really very rudimentary because it only supports `%d`, `%f`, `%s` and `%c` modifiers (in fact, this is my solution to exercise 7-4). Let's give it a look:

```c
#include <stdio.h>
#include <stdarg.h>
#include <ctype.h>
#include <stdlib.h>

#define BUFSIZE 100

int getch(void);
void ungetch(int);

int minscanf(char *fmt, ...) {
  void skipblanks(void);
  va_list ap;
  int matched = 0, c, i, sign, done = 0;
  char *p, buffer[BUFSIZE], *s, *initial;
  
  va_start(ap, fmt);
  
  for (p = fmt; !done && *p; p++) {
    if (isspace((unsigned char) *p)) {
      skipblanks();
      continue;
    }
    else if (*p != '%') {
      /* Not a space, not a modifier, match characters */
      if ((c = getch()) != *p) {
        ungetch(c);
        done = 1;
      }
      continue;
    }
    p++;
    /* Modifiers */
    switch (*p) {
      case 'd':
        skipblanks();
        if ((c = getch()) == '-')
          sign = -1;
        else {
          sign = 1;
          ungetch(c);
        }
        i = 0;
        while ((c = getch()) != EOF && isdigit((unsigned char) c))
          buffer[i++] = c;
        ungetch(c);
        buffer[i] = '\0';
        if (i > 0) {
          matched++;
          *va_arg(ap, int *) = sign*atoi(buffer);
        }
        else {
          if (sign == -1)
            ungetch('-');
          done = 1;
        }
        break;
      case 'f':
        skipblanks();
        i = 0;
        if ((c = getch()) == '-')
          sign = -1;
        else {
          sign = 1;
          ungetch(c);
        }
        while ((c = getch()) != EOF && isdigit((unsigned char) c))
          buffer[i++] = c;
        if (c == '.')
          buffer[i++] = '.';
        else
          ungetch(c);
        while ((c = getch()) != EOF && isdigit((unsigned char) c))
          buffer[i++] = c;
        ungetch(c);
        if (buffer[i-1] == '.') { /* Cannot finish with single dot */
          ungetch('.');
          i--;
        }
        buffer[i] = '\0';
        /* Allow for leading '.' in float string */
        if (i > 1 ||
           (i == 1 && isdigit((unsigned char) buffer[0]))) {
          matched++;
          *va_arg(ap, float *) = sign*atof(buffer);
        }
        else {
          if (sign == -1)
            ungetch('-');
          done = 1;
        }
        break;
      case 's':
        skipblanks();
        for (s = initial = va_arg(ap, char *), c = getch(); c != EOF && !isspace((unsigned char) c); s++, c = getch())
          *s = c;
        ungetch(c);
        if (s != initial) {
          matched++;
          *s = '\0';
        }
        else
          done = 1;
        break;
      case 'c':
        /* %c does not skip blanks */
        *va_arg(ap, char *) = getch();
        matched++;
        break;
    }
  }
  va_end(ap);
  return matched;
}

void skipblanks(void) {
  int c;
  while ((c = getch()) != EOF && isspace((unsigned char) c));
  ungetch(c);
}

char buf[BUFSIZE];
int bufp = 0;

int getch(void) {
  return (bufp > 0) ? buf[--bufp] : getchar();
}

void ungetch(int c) {
  if (bufp >= BUFSIZE)
    printf("ungetch: too many characters\n");
  else
    buf[bufp++] = c;
}
```

This is a lot of code and it can look really scary. The first thing to notice is that we use several auxiliary functions: `skipblanks()`, `getch()`, and `ungetch()`. `skipblanks()` is called every time a space is read in the format string, and what is does is - well - it skips every blank until a non-blank is found. A blank is defined to be a character for which `isspace()` returns true; `isspace()` is declared in `ctype.h` and is the portable way of testing for blanks.

What are `getch()` and `ungetch()`? They are used to manipulate the buffer. `getch()` will return the next character in the input, or `EOF`. The next character in the input is picked from our buffer if it is not empty, otherwise, it delegates its job to `getchar()`, which blocks waiting for input. `ungetch()` does exactly the opposite: it is used in those unfortunate situations where `scanf()` has read too much.

With this in mind, we can implement `scanf()`. It consists of a main loop where the format string is scanned, and it keeps running as long as the end of the string is not reached and a premature return condition (signaled by setting `done` to 1) is not reached. The assignment `done = 1;` could be replaced by a direct return instruction. I didn't do it because we need to call `va_end(ap)` before returning, to do some house cleaning for the variable arguments list. So, I chose to centralize the clean up and return instructions in one place. The main loop body is not very hard to understand, it begins by checking for easy cases, like a blank or something that is not a modifier. When a modifier is found, some conversions need to be made. I think the code is pretty much self explanatory and it helps to really bring together and understand what is behind `scanf()`.

I challenge you to go through the code for `%d` and `%f` really carefully. For `%d`, we need to discard things like `-Hello!`. This is tricky because you can read `-` and still think that it's going to be a negative number, but when you read `H`, you realize you just read 2 characters that you shouldn't, and you must ensure that they are pushed back in the right order (characters must be pushed back in a LIFO - Last In First Out - order). Something similar might happen with `%f`. For example, we have to allow things like `-.36`, but we cannot consume any input in `-.some text here`. That is, if you call `scanf("%f")` and then `scanf("%s")` for the input `-.some text here`, the first call cannot consume any input characters, even though it will have to read `-`, `.`, and `s` before determining that it shouldn't have read those 3 characters in the first place. Finally, in any of these cases, we never know how many digits our number will have, so we just have to keep on reading until something that is not a digit comes up, at which point it must also be pushed back to the input. 

As you probably understand, `scanf()` has a lot of boundary conditions and tricky situations like these, and it is very, very easy to mess up your input when implementing `scanf()`. I think it is a very good exercise to look at my code and really understand how it works, believe me, it's a very good way to understand `scanf()`.

If you're into this as much as me, you might be wondering how is `getchar()` implemented (or its friend `putchar()`, that can be used in a minimal `printf()` implementation). Well, `getchar()` and `putchar()` are really just wrappers for system functions (ever heard about system calls? Think of them as a set of basic low-level funtions, or an API, that is provided by the host operating system). `getchar()` and `putchar()` are just macros to the `read()` and `write()` system calls. They call these functions with `stdin` and `stdout` as arguments, respectively. `stdin` and `stdout` are constants that refer to the file descriptor of a file that represents the standard input and another that represents standard output. In Linux, these file descriptors are tipically `0` and `1` (and `2` for `stderr`, another constant). In fact, `getchar()` and `putchar()` are macros that call other macros defined in `stdio`, which in turn have buffered input, and will return characters from that buffer if there's something in there, or call `read()` to read a new chunk of bytes from a file (buffered input is used to minimize system calls, because a system call involves a little more overhead than normal function calls - for example, the CPU has to change from user mode to kernel mode, so minimizing system calls is a clever way to speed things up). It's sort of what we're doing here with `getch()` and `ungetch()`.

Actually, you can have some control over `stdio.h` input buffer by using the macro `ungetc()`. `ungetc()` lets you push *at most* one character back to the buffer, so it's not as powerful as the `getch()` and `ungetch()` functions, but it works properly with any I/O function defined in `stdio`. For example, you can use `ungetc()` to interleave `getchar()` and `scanf()` calls, and because deep down they use the same structure, which, just out of curiosity, is a `FILE` structure, consisting of a `char` buffer, a counter to the number of characters left in the buffer, a pointer to the next character in the buffer, a pointer to the beginning of the buffer, file flags, etc, you can actually insert a new virtual character in the input stream. Consider this code:

```c
#include <stdio.h>

int main() { 
  int c;
  ungetc('H', stdin);
  while ((c = getchar()) != '\n') {
    printf("%c", c);
  }
  printf("\n");
  return 0; 
}
```

Try saving a file `input` with the content `ello, World!\n`, and run this program with `stdin` redirected to this file. You will see that it prints `Hello, World!\n`. You can also do this without redirecting `stdin`, but it's not so fun because it will print `H` before you have the chance to write `ello World` and press return.

This all seems very nice, but let's get down to it: how can we validate input using `scanf()`? Well, that can be a pain in the ass. The drawback of using `scanf()` is that you have a very limited ability to validate user's input. Anyway, there are a few primitive mechanisms we can use to help us. `scanf()` returns the number of successfully matched and assigned variables. For example, `scanf("%d", &a);` will return `1` if it was able to read a number and assign it to `*a`. `scanf()` calls with no conversion specifiers in the format string will always return either `0` or `EOF`, because there aren't any possible matches and assignments to make.

OK, so, `scanf("%d", &a);` can be used to capture a number, but remember, `%d` allows any number of leading white space characters. What if you don't want to allow any white spaces? Well, there's a special conversion modifier you can use: the square brackets. Quoting from the manpage:

> [
>
> Matches a nonempty sequence of characters from the specified set of accepted characters; the next pointer must be a pointer to char, and there must be enough room for all the characters in the string, plus a terminating null byte. The usual skip of leading white space is suppressed. The string is to be made up of characters in (or not in) a particular set; the set is defined by the characters between the open bracket [ character and a close bracket ] character. The set excludes those characters if the first character after the open bracket is a circumflex (^). To include a close bracket in the set, make it the first character after the open bracket or the circumflex; any other position will end the set. The hyphen character - is also special; when placed between two other characters, it adds all intervening characters to the set. To include a hyphen, make it the last character before the final close bracket. For instance, [^]0-9-] means the set "everything except close bracket, zero through nine, and hyphen". The string ends with the appearance of a character not in the (or, with a circumflex, in) set or when the field width runs out.

That's great news. This means that, for example, if our input is supposed to start with `Date: ` (that is, the word `Date` followed by `:` followed by one space), we could do something like this:

```c
scanf("%[^ ]s%c", firstWord, separator);
```

Where `firstWord` is a `char` array, and `separator` is a pointer to a single `char`. Then we would have to compare `firstWord` against `Date:`, and check that `*separator == ' '`. However, you must pay close attention to the negated character class. The semantic meaning of a negated character class is *Match something that is NOT <...>* (in our case, match something that is not a space). It's not *Do not match <...>*. The meaning is a little bit different: the negated character class must match at least one character, think of `[^ ]` as a shortcut to list every possible characters except space. If the first character in the input is a space, `scanf()` will return immediatly without consuming any character in the input. If you're doing that in a loop, your program will enter an infinite loop condition. Also, note that the normal meaning of a white space character for `scanf()` is dropped. `[^ ]` will match anything that is not a literal white space, so it will match newline. If you didn't want space and newline, you'd have to use `[^ \n]`.

What about matching integers without leading a space? We can't use `%d`, because it skips leading blanks. If you don't want to allow leading blanks, you're out of luck. My suggestion is to capture the numbers to a char array, as in

```c
scanf("%[0-9]s", number);
```

And then call `atoi(number)`. Note that this will not match `-256`, because you didn't allow `-`. We could allow it with:

```c
scanf("%[0-9-]s", number);
```

But this also matches things like `-256-987-325-6114`. Well, that's not a big problem, because `atoi()` stops converting as soon as a non-digit character is found, but we can do better:

```c
scanf("-%[0-9]s", number);
```

The problem is that now it will only match negative numbers. As you can see, input validation can be hard with `scanf()`. The only way I can think of to read numbers without leading space is to try both `scanf()` calls: first, we try `scanf("-%[0-9]s", number)`. If it returns 0, then we try `scanf("%[0-9]s", number)`. If all of the calls returned 0, then we have invalid input. Otherwise, we have the number, either positive or negative, stored in the variable number, which we will pass to `atoi()`.

You might as well get a whole line of input, either using `getline()` from `stdio.h`, or using `scanf("[^\n]s", line)`. Remember that `scanf()` will not consume the trailing newline, so the next call to `scanf("[^\n]s", line)` will return immediatly without touching the input. You can add a trailing space in the format string.

I wouldn't recommend validating input in C directly with `scanf()` or `getline()`. I mean, yeah, if it's really a simple set of rules, you can code your own function to interleave `scanf()` and `getchar()` calls to figure out if the input comes as expected, but a better and more robust approach is to use C's regex library, `regex.h`, that offers a powerful way to validate input using regular expressions. And, as a final note, let me tell you that if you think you know regular expressions but you've never read *Mastering Regular Expressions, 3rd Edition*, by Jeffrey Friedl, then you don't know regular expressions, even if you've been using it for 10 years. Believe me. That book is worth its weight in gold, and for whatever it's worth, you should go and buy it right now!

Have fun!
