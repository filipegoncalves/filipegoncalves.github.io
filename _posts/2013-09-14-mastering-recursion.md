---
layout: post
title: Mastering recursion
---

The ultimate guide on how to deal with recursion

-----

Back in June this year, my sister was about to finish 9th grade, and she was preparing for national exams. In Portugal, you must be evaluated with final Portuguese and Maths exams. She was working pretty hard (harder than me when it was my time, I was a lazy ass back at her age!), on her way to have great marks, but she never quite get along with Maths. Anyway, she studied, and practiced, until the day of the exam. By the end of the day, when I was back home, I asked her how it was. She was waving the exam in her hand, ranting about some weird, "impossible" exercise that no one really understood, but apart from that, she said it was ok.

So I grabbed it, gave it a quick look, and discussed with her some of her answers. She had done some minor mistakes in 2 or 3 exercises, but it wasn't that big of a deal. Eventually, I had the chance to look at the infamous exercise that most kids struggled with under the pessure of the examination:

> Given two integers m and n, such that m > 0, n > 0, and m > n, the greatest common divisor between m and n is the same as the greatest common divisor between n and m-n. For example, gcd(16,12) = gcd(12,4).
>
> By repeatedly applying this property, calculate gcd(32,80), until you are left with the greatest common divisor of the same number.

This is the Euclidean algorithm. It is very easy to implement it as a recursive function. If you've never heard of it, it might be worth taking a quick look at the wikipedia page that describes it. The complexity analysis is not trivial.

Ok, I can't be straight on this matter, because I'm from a computer science background. But hell, is it THAT complicated? The problem description is itself pretty much intuitive. Seriously, what's so complex about writing

```
gcd(32,80) = gcd(80,32) = gcd(32,48) = gcd(48,32) = gcd(32,16) = gcd(16,16) = 16
```

The tricky part here is not the "recursive" call, it's more the fact that you must pay attention to the condition `m > n` and so you sometimes need to swap the order of arguments, or you will end up with negative numbers.

Anyway, I confess that I'm probably biased - my mom, who comes from a linguistics background, lost herself reading the exercise. So I just convinced myself that maybe it is not as easy to the average 14-year-old kid. Recursion is difficult because you don't see it in your day-to-day routine. Your mind is used to deal with physical objects from the real world, and it's not very common to have recursive objects in the real world, if that even makes sense... Ok, maybe it does ... Hmmmm, matryoshka dolls? Hahahahahahah, that was the best I could think of!

No, wait a second. That's not the best I could think of. Actually, a few days ago, I was with a friend who told me about *recursive dreaming*. What?! Yeah! He claimed that he had a dream where he was sleeping and dreaming, *inside his dream*, and when he woke up from the 2nd dream, he thought he was back to real world, until he *really* woke up and realized he had just been in a 2-depth dream! WHAT THE HELL?!?!?!

I don't know if he was serious, but he told me he was not kidding. This is scary. What if our whole life is inside a recursive dream? How do you get out of an n-depth dream? Of course, you have to wake up `n` times, but `n` is unknown. And this gets scarier if you try to google "recursive dreaming"...

But, back to real life...

What can I say? Well, as a matter of fact, I believe that the kids who solved this would most likely get along with computer science, while the others would try to get as far as they can from it. For some reason, the human brain has difficulties understanding something that is defined in terms of itself. And, as usual, understanding recursion and dealing with it requires practice, requires experimenting. In the end, that's what makes you special - really understanding those things that are not very intuitive. Why would we need such a powerful cpu in our head if we don't want to use it? 

Believe it or not, I actually *wanted* to teach recursion to my brother. And I wanted to teach it to my sister too, but she was not very amused, so I turned into my brother. He didn't ignore me like my sister did, but he didn't get it. Too young. But I'll try again next year (he's just 13!). God, it would have been really cool if someone got me into recursion (and pointers, and algorithms, and all that) when I was 14! I, like nearly everybody else, only found the concept of recursion in my freshman's year in college. It was in an introductory programming course, and we were given the most basic, traditional example of recursive function: the factorial, of course.

```c
unsigned factorial(unsigned n) {
  if (n <= 1)
    return 1;
  else
    return n*factorial(n-1);
}
```

To understand recursion, we have to make a mental jump. We literally have to think recursively. There are different approaches to understanding recursion. Depending on the specific problem you're tackling, one or the other may seem more convenient. Either way, they're not mutually exclusive; in fact, they will most likely overlap, in the sense that when you use one, you will eventually mix the other one in the middle.

## Base Case and Build

Base Case and Build means that first you solve the problem for an easy, trivial base case. In our factorial example, we solve the case for `n <= 1`. It's trivial, we immediately know the result.

After coding the base case, we go on to code the general case, assuming we have the solution of a simpler subcase. In our factorial example, we are assuming we know factorial of `n-1`, so it is easy to compute `factorial(n)`. And then there's the recursive "jump" - applying repeatedly the same rule. Assuming we know factorial of `n-2`, then it is easy to know factorial of `n-1`. And this keeps happening over and over until eventually we get to the base case, and the recursion winds back.

Another way to see it is something like this: if I had a function that did part of the work, how could I use it to do the full work? In the factorial example: if I had a function that gave me `factorial(n-1)`, how could I know `factorial(n)`? With this approach, you are dealing with the general case first. It can be easier to deal with, depending on the problem, and some people prefer to make the general case first. Why? Because sometimes, it is not trivially obvious what is the base case. If you make the general case first, you can then run it through a simple example and determine the base case.

Again, using the factorial function, we would have written the general case first - `n*factorial(n-1)`, and then we could set `n = 3`, and simulate the call stack to eventually find out that a good base case is `n <= 1`.

This example may be silly because it is very easy, but the concepts are very important to keep in mind. Although this is a good approach, I find the next one more valuable and useful. We will use it to develop a nice solution to an inherently recursive exercise.

## Exemplify</strong>
This may not seem like a very good advice, but it's the key to master recursion. Recursion exercises can be very difficult to assimilate. If you practice, you can develop some kind of mindset that allows you to identify certain patterns in problems and immediately derive and apply a recursive algorithm. But that's not always the case - recursive exercises can be very different among each other, so don't try to generalize.

Now, I'm going to teach you *how* to exemplify. And how am I going to do that? Well, guess what, let's see an example exercise. Yes, I'm going to show you how to exemplify with an example.

### Example 1 - Check if a list is a palindrome

Imagine you want to write some code to check if a list of integers is a palindrome. For example, the lists

```
0

1->1

1->2->2->1

1->2->3->2->1
```

are palindromes (they read the same forwards and backwards), but these are not:

```
1->2

1->2->2

1->2->3->2>2->3
```

Of course, the whole fun of the exercise is that these are single linked lists, you have no pointer to the previous node.

You could just create a reversed copy of the list and compare it to the original list; if they're the same, it's a palindrome (in fact, you only need to compare the first half, since the second half of the reversed list is the same as the first half of the original list). But that's not very interesting, and not very efficient: you'll have to iterate through the whole list first just to copy it. Before starting some actual useful work, you already used \\(\mathcal{O}(n)\\) time.

### Example 1.1 - Reversing a list

BUT, even though we do know this is not THE solution, let's stop for a while and ask ourselves: how can we reverse a single linked list? You may want to consider using a stack of list nodes, so that when you reach the end of the list, you just have to create a list by popping each element off the stack and setting its `next` pointer to the next element in the top of the stack. Or you can understand that function calls use a FIFO policy - if you call `f()`, that calls `x()`, that calls `y()`, then `y()` returns to `x()`, and `x()` returns to `f()` - and use the function call stack mechanism as your stack and make a recursive function:

```c
struct list_node {
	int val;
	struct list_node *next;
};

struct list_node *reverseList(struct list_node *head) {
	struct list_node *reverseListAux(struct list_node *, struct list_node **, struct list_node *);
	struct list_node *newHead;
	reverseListAux(head, &newHead, head);
	return newHead;
}

struct list_node *reverseListAux(struct list_node *origHead, struct list_node **result, struct list_node *curr) {
	if (curr->next == NULL)
		*result = curr;
	else
		reverseListAux(origHead, result, curr->next)->next = curr;
	if (curr == origHead)
		curr->next = NULL;
	return curr;
}
```

Brief and beautiful. Anyway, the point is that algorithms where you need a stack are eligible to recursive functions due to its LIFO call policy (unless, of course, the stack is used as a memory mechanism, such as a character buffer). You literally reuse the function call stack to fit your needs.

OR, if you think a little bit more about it, you can hopefully see that you can do it in a very simple way by creating a new list and iterating in the original list, inserting each element in the head of the new list. Or, for that matter, you could do it in-place with \\(\mathcal{O}(1)\\) memory by using a pointer `prev`, initialized to `NULL`, and a pointer `current`, initialized to point to the list head, and a pointer `tmp`. While `current` is not `NULL`, all you have to do is set `tmp` to `curr->next`, then set `curr->next` to `prev`, set `prev` to `curr` and `curr` to `tmp` (confused? sketch it on paper, you'll get it - it's the best way to reverse a list because it does so in \\(\mathcal{O}(n)\\) time and uses only 3 pointers). 

But I wanted to show you how to do it recursively, as an exercise. How did I come up with `reverseListAux()`? By exemplifying. Intuitively, you know that if you want to reverse the list `a->b->c->d`, first you have to reverse the list `b->c->d`, which will turn out to be `d->c->b`, and append `a` to the end. Let's simulate the call stack. This is really important, let me repeat that:

**Simulate the call stack**

Simulating the call stack is very, very important to make "the jump" to recursive definitions. You may have gotten my intuitive description, but you are probably not still enough sure how you will exactly implement it. So let's simulate the recursion:

```
reverse(a->b->c->d)
        reverse(b->c->d) _b_
                reverse(c->d) _c_
                        reverse(d) _d_
                                return d
                        d->next = c
                        return c
                c->next = b;
                return b
        b->next = a;
        a->next = NULL;
        return the new head (d)
```

Everything that happens inside the same call is at the same level of indentation. Return values are surrounded with `_` so that they are easily spotted in the call hierarchy. How did I do this? I drew the call hierarchy down to `reverse(d)`, and then I thought, *Well, the reverse of a list with 1 element is the list itself. So let's just return it*. And then I saw that it was really cool if every call returned its list head to the above call, because we could use that to change the `next` pointer.

Building the call stack is important because that's when you see how it will really work. With this in mind, we can see that:

* Each call will return its list head to the previous call
* Somehow, we will have to pass the last call return value up to the top root call
* After the recursive call returns `X`, we have to set `X->next` to the list head of the current call
* The top-root call is returning the new head, but earlier we said each call returns its list head. We need to somehow return the new list head by other means.
* We must know when the top root call is about to return, so that we can set `head->next` to `NULL` (the old head is now the last element).

Point number 3 is the core of the algorithm, it's where the recursion occurs. It's what makes it all work, the rest are details. Number 1 is also important because it is part of the recursion. We will achieve number 2 by passing in a pointer to pointer to `list_node`; this allows us to pass the new list head by reference up to the top root call. In fact, this solves both point number 2 and point number 4. We just have to make sure that the former caller will pass in a pointer to pointer to `list_node` where the new list head will be stored. 

Finally, because of number 5, we see that we need to identify the top root call to update the original list's `head->next`. We will do this by forcing the original caller to pass in a pointer to the list head, so that we can check whether the current call list head is the same as the original head. Putting it all together, the code above should make sense.

Ok, that was a good example, but remember that this is not what we were trying to solve. We wanted to code a function to see if a list is a palindrome. The basic idea is that we need to compare the 1st element with the last one, the 2nd with the 2nd to last, etc, until we have compared every pair. If the list length is odd, the middle element is not paired up with anyone; if its length is even, there is no middle element.

Let's imagine we have a list with an odd number of elements. For example, the list `a->b->c->b->a`. Now, let's simulate how the call stack would look like, using the same notation as before:

```
isPalindrome(a->b->c->b->a) _NULL_
        isPalindrome(b->c->b->a) _a2_
                isPalindrome(c->b->a) _b2_
                        return b2
                b1 == b2 ? continue... : not palindrome
                return b2->next (a2)
        a1 == a2 ? continue... : not palindrome
        return a2->next (NULL)
```

Here, the numbers after the element indicate whether it's the 2nd node with the same value or the first node. For example, `a2` refers to the `a` node in the end, and `a1` refers to the first `a` node. We can see from this simulation that the recursion stops when we reach the middle element. That's the base case, and because the list length is odd, the middle element is not compared, so we pass `middle->next` to the previous call. The previous call compares its own list head with that and sees that both nodes values are the same, and it returns `b2->next`, `a2`, which will be compared in the previous call with `a1`. And we keep doing this as long as everything matches up. Again, simulating the call stack is a very good way to derive a recursive algorithm. 

Now, what if the list length is even? In that case, there is not one middle element, there's two of them. In `a->b->b->a`, the middle elements are the b's. Well, we will have to agree on what exactly we will assume to be the middle element when the list length is even. We can only use one of them as middle. So, for a moment, let's abstract ourselves from the original problem and think how to find the middle element in a list. Of course, you could iterate through the list to find its length as a preprocessing step. But that's ugly: we just need to iterate once. Have you ever heard about the runner method? It's a technique in which you have two pointers that iterate through the list at different pace. It is a very useful method. Usually, this method is used with a pointer that iterates through the list node by node - called the `slow` pointer - and another one that "walks" through the list at a rate of 2 nodes per iteration; this is called the `fast` pointer. Thus, if `slow` and `fast` are initialized to point to the list head, by the time `fast` reaches the end of the list, `slow` is exactly in the middle. If it's an odd-length list, `slow` points to the middle element, if it's an even-length list, it points to the rightmost middle element. This method can be implemented in a few lines of code:

```c
struct list_node *findMiddle(struct list_node *h, int *isOdd) {
	struct list_node *slow;
	struct list_node *fast;
	
	slow = fast = h;
	
	while (fast && fast->next) {
		fast = fast->next->next;
		slow = slow->next;
	}
	
	if (fast)
		*isOdd = 1;
	else
		*isOdd = 0;
	
	return slow;
}
```

I chose to inform the caller not only about the middle element, but also about whether the list is even or odd, because this is important for the recursive algorithm, as we saw earlier.

So now that we have an agreement on what is the middle element in an even-length list and a fair method to find the middle element, we can go back to the palindrome problem. By analyzing the example call stack, we can develop the following pseudo-code:

```
list_node *isPalindromeRec(list L, list_node middle, bool_ptr result, bool isOdd) {
	if (L->head == middle)
		if (isOdd)
			return middle->next;
		else
			return middle;
	compareTo = isPalindromeRec(L->next, middle, result, isOdd);
	if (*result == false) /* not palindrome, don't bother checking */
		return;
	if (L->head->val != compareTo->val)
		*result = false;
	return compareTo->next;
}
```

That should be more or less self explanatory if you examined the example call stack carefully. Notice that in the case of an odd-length list, the middle element is never compared, so the base case immediately skips it and returns the next node. If it's an even-length list, we need to compare `middle` with the previous node, so we pass `middle` to the previous call instead of skipping it. Translating this into real C is not very hard:

```c
struct list_node *findMiddle(struct list_node *, int *);
struct list_node *isPalindromeRec(struct list_node *, struct list_node *, int *, int);
int isPalindrome(struct list_node *h) {
	struct list_node *middle;
	int isOdd;
	int result;
	
	middle = findMiddle(h, &isOdd);
	isPalindromeRec(h, middle, &result, isOdd);
	return result;
}

struct list_node *findMiddle(struct list_node *h, int *isOdd) {
	struct list_node *slow;
	struct list_node *fast;
	
	slow = fast = h;
	
	while (fast && fast->next) {
		fast = fast->next->next;
		slow = slow->next;
	}
	
	if (fast)
		*isOdd = 1;
	else
		*isOdd = 0;
	
	return slow;
}

struct list_node *isPalindromeRec(struct list_node *head, struct list_node *middle, int *result, int isOdd) {
	struct list_node *compareTo;
	if (head == middle) {
		*result = 1;
		if (isOdd)
			return middle->next;
		else
			return middle;
	}
	compareTo = isPalindromeRec(head->next, middle, result, isOdd);
	if (*result && head->val != compareTo->val)
		*result = 0;
	return compareTo->next;
}
```

That's a good solution. What is the time and space complexity? To find the middle, we need \\(\mathcal{O}(n/2)\\) time and \\(\mathcal{O}(1)\\) memory. The recursive palindrome function uses \\(\mathcal{O}(n)\\) time - every element in the list is visited exactly once - and \\(\mathcal{O}(n/2)\\) memory, because the stack keeps growing until middle is found. So, overall, our algorithm runtime is \\(\mathcal{O}(n/2 + n) = \mathcal{O}(3n/2)\\), and the total space complexity is \\(\mathcal{O}(n/2)\\). Not bad!

But bear with me for a minute, we can do better. What bothers me about big-O notation is that we can drop constants, so we could say our algorithm runtime is \\(\mathcal{O}(n)\\). And even if we called `findMiddle()` a thousand times, technically, our algorithm would still be \\(\mathcal{O}(n)\\). That sucks. It sucks because constants can make a hell of a difference in real world.

Going back to our problem, you might have noticed that we don't really need to find the middle before calling the recursive function. We can do that in the recursive function itself and save \\(\mathcal{O}(n/2)\\) time, with no additional side costs. We can forget about `middle` and `isOdd` arguments, and replace them by `slow` and `fast`. We literally incorporate the runner method in our recursive calls. Again, in a pseudo-semi-C code, this is what I mean:

```
list_node *isPalindromeRec(list L, bool_ptr result, list_node *slow, list_node *fast) {
        if (fast && fast->next) {
                fast = fast->next->next;
                slow = slow->next;
        }
        else {
                if (fast)
                        return slow->next;
                else
                        return slow;
        }
        compareTo = isPalindromeRec(L->next, result, slow, fast);
        if (*result == false)
                return;
        if (L->head != compareTo)
                *result = false;
        return compareTo->next;
}
```

I think it's easy to make this change after we develop the earlier algorithm with `findMiddle()` separated. I just thought about forgetting `findMiddle()` after I coded the original algorithm, so don't worry if you didn't think about it earlier. Anyway, this should be accessible for you now. I really wish I could give you some more help in here, but I can't explain it any better. Turning this into real code becomes:

```c
struct list_node *isPalindromeRec(struct list_node *, int *, struct list_node *, struct list_node *);

int isPalindrome(struct list_node *h) {
	int result;
	isPalindromeRec(h, &result, h, h);
	return result;
}

struct list_node *isPalindromeRec(struct list_node *head, int *result, struct list_node *slow, struct list_node *fast) {
	struct list_node *compareTo;
	if (fast && fast->next) {
		fast = fast->next->next;
		slow = slow->next;
	}
	else {
		*result = 1;
		if (fast)
			return slow->next;
		else
			return slow;
	}
	compareTo = isPalindromeRec(head->next, result, slow, fast);
	if (*result && head->val != compareTo->val)
		*result = 0;
	return compareTo->next;
}
```

So..................... deep breath...

Now, we have a real \\(\mathcal{O}(n)\\) solution. The space complexity remains \\(\mathcal{O}(n/2)\\). I'm happy with that.

### Example 2 - RPN calculator

Let me show you another exercise where exemplifying is very useful. Have you ever heard of RPN calculators? It stands for Reverse Polish Notation, and it's used to present arithmetic expressions in an unambiguous way. Rather than using the traditional infix notation, it uses postfix notation, so no parentheses are needed and there's no need to define operator precedence. So, for example, the expression

```
2 3 +
```

Evaluates to 5, and

```
2 3 + 2 -
```

is equivalent to

```
5 2 -
```

which in turn is evaluated to 3. The expression

```
3 2 6 8 9 + * / -
```

is equivalent to `3-(2/(6*(8+9)))`, and evaluates to 3. The grammar that corresponds to an RPN calculator is defined below:

```
exp: NUMBER 
    | exp exp +
    | exp exp -
    | exp exp *
    | exp exp /
```

As you can see, the grammar is naturally recursive (this is very common when you're dealing with parsers). A simple way to implement an RPN calculator is to insert each number read in a stack, until an operator is read. Assume you've read a `+` operator. Now, we pop two elements from the stack, `n1` and `n2`, and push into the stack `n2+n1` (note that the left operand of the corresponding arithmetic expression is the second element popped). We keep doing this until no more input is given, and at that time we just print whatever is stored in the top of the stack. Thus, for the above example expression

```
3 2 6 8 9 + * / -
```

We would push `3`, `2`, `6`, `8` and `9`. Then, `+` is read, so we pop `9` and `8`, and push `8+9 = 17`. Our stack now looks like this (top to bottom):

```
17
6
2
3
```

Next, we read `*`, so we pop `17` and `6`, and push `6*17 = 102`. Now we read `/`, we pop `102` and `2`, and push `2/102 = 0`. Finally, we read `-`, so we pop `0` and `3`, and push `3-0 = 3`. No more input is available, so the final result is 3.

K&R implements this in section 4.3 using a simple array as a stack. Like I said before, algorithms that use a stack can be converted to a recursive implementation. In this example, I want to show you how to implement an RPN calculator recursively. Let's see how we would recursively parse this expression:

```
3 2 + 5 *
```

Intuitively, we know we have to make recursive calls until we find an operator. When we find it, we have to inform the upper caller that he just hit "the bottom", and so the upper caller must say to its upper caller that he's the 2nd operand for operation `X`, who will compute the result and make a new recursive call with this result. Here's the call stack:

```
parse(3 2 + 5 *)
    parse(2 + 5 *) _2_
        parse(+ 5 *) _+_
            return "operation +"
        return "2nd operand = 2"
    value = 3 + 2 = 5
    parse(5 5 *)
        parse(5 *) _5_
            parse(*) _*_
                return "operation *"
        return "2nd operand = 5"
    value = 5*5 = 25
    parse(25) _25_
        return "25 - no more input!"
    return 25
```

With this example, we see that:

* The recursion winds back when an operator is read
* We will have to handle different return meanings. In particular, we must be able to distinguish between the cases where a recursive call returned because it read an operator, because it is the number before an operator (2nd operand in an expression), or because there's no more input
* We need to store the operator read, because it will be used 2 layers above
* Somehow, we will need to insert artificial input that simulates the `push()` operation that takes place after a value is calculated

The points mentioned above will be solved with the use of global variables. Global variables are not very good programming style, but they serve for the educational purposes of this article. We could, instead, pass these values as arguments between every recursive call, but I preferred not to. Because this is not a trivial program, I chose to build some pseudo-code first. Here's how `parse()` will behave from a high-level point of view:

```
parse():
	call nexttoken()
	if no_more_input
		return 0
	if read_operation_char
		finish_recursion_global = true --signalizes end of recursion to upper layers
		return 0
	else --nexttoken() must have read a number
		v = number_read_global
		n = parse()
		
		if (no_more_input)
			return v 
		
		if (finish_recursion_global)
			--if we get here, this call is the 2nd operand of an expression
			finish_recursion_global = false
			op2_set_global = true
			return v
		else
			if (op2_set_global)
				--if we get here, this call is the 1st operand of an expression,
				--and we just returned from the 2nd operand
				v = v OPERATION_global n
 				op2_set_global = false
				push v to the input
			return parse()
```

I built this code from the sample call stack provided above. `parse()` will return the numeric result of an expression. Everytime a non-numeric result is to be returned (like informing that an operation symbol was read, or that there is no more input), 0 is returned, and the appropriate global flags are set. This forces us to carefully check the global flags in the previous calls, to make sure we will not be reading inconsistent return values. A token is either a number or an operation. `nexttoken()` is responsible for retrieving the next token from the input. It will return constant numeric values indicating whether an operation or a number was read. Here's the pseudo-code:

```
nexttoken():
	ignore blanks; if \n is read, return and set end_of_input_global to true
	if it's EOF
		exit program
	if it's operation
		set OPERATION_global = operation_name
		return OP
	else
		capture whole number
		set number_read_global = number_value
		return NUMBER
```

With this in mind, we are ready to implement it in C. You should convince yourself that the pseudo-code above works, if you're skeptical, there really is nothing more you can do other than dry running it with sample input and seeing that it works. I don't know what else you can do besides looking at examples, building pseudo-code, and testing it. In this kind of recursive exercises, it is crucial that you thoroughly test your pseudo-code with nominal case and boundary case conditions, *before implementing it*. Finding a case in which your solution doesn't work after implementing it often leads to very hard to fix bugs, and then you don't want to throw your code away, and you end up with a highly sloppy and obfuscated solution that not even you will understand a couple of months from now. So, if you don't follow my advice, then *shame on you*, that's all I have to say.

And now, my implementation:

```c

char operation;
int number;
int finished_recursion;
int input_ended;
int op2_set;

int nexttoken(void);
void pushback(int);
int parse(void) {
	int v, n, t;
	
	t = nexttoken();
	if (input_ended)
		return 0;
	if (t == OP) {
		finished_recursion = 1;
		return 0;
	}
	else {
		v = number;
		n = parse();
		if (input_ended)
			return number;
		if (finished_recursion) {
			finished_recursion = 0;
			op2_set = 1;
			return v;
		}
		else {
			if (op2_set) {
				switch (operation) {
					case '+':
					v += n;
					break;
					case '-':
					v -= n;
					break;
					case '*':
					v *= n;
					break;
					case '/':
					v /= n;
					break;
				}
				op2_set = 0;
				pushback(v);
			}
			return parse();
		}
	}
}
```

For `nexttoken()`, I'll be using the `getch()` abstraction provided by K&R to create a temporary buffer and be able to push characters back into the input. As K&R mentions, sometimes, a program doesn't know it has just read too much input until it has read too much, so we need a mechanism to push it back.

```c
#include <stdlib.h>
#include <ctype.h>
#include <stdio.h>
#define NUMBER 0
#define OP 1
#define END 2

int getch(void);
void ungetch(int);

int nexttoken(void) {
	int c;
	int t;
	int sign;
	while ((c = getch()) != '\n' && isspace((unsigned char) c));
	if (c == '\n')
		return input_ended = END;
	if (c == EOF)
		exit(0);
	if (c == '+' || c == '*' || c == '/') {
		operation = c;
		return OP;
	}
	else if (c == '-' && !isdigit((unsigned char) (t = getch()))) {
		ungetch(t);
		operation = c;
		return OP;
	}
	else {
		/* assert: c != '-' OR (c == '-' and there's a digit after */
		number = 0;
		sign = 1;
		if (c == '-') {
			c = t;
			sign = -1;
		}
		for (; isdigit((unsigned char) c); number = number*10 + c - '0', c = getch());
		number *= sign;
		ungetch(c);
		return NUMBER;
	}
}

#define BUFSIZE 100

int buf[BUFSIZE];
int bufp = 0;

int getch(void) {
	return (bufp > 0 ? buf[--bufp] : getchar());
}

void ungetch(int c) {
	if (bufp >= BUFSIZE)
		printf("ungetch: too many characters\n");
	else
		buf[bufp++] = c;
}
[/code]

Using getch() allows us to easily push back artificial input, just what we needed for pushback(). The implementation is simple:

[code language="cpp"]
void pushnumberback(int n, int sign) {
	ungetch((n%10)*sign+'0');
	if (n /= 10)
		pushnumberback(n, sign);
}

void pushback(int n) {
	int sign;
	pushnumberback(n, sign = (n > 0 ? 1 : -1));
	if (sign == -1)
		ungetch('-');
}
```

Ironically, `pushnumberback()` is recursive as well, but it didn't have to be that way. It's easily converted to an iterative approach, but I decided to let it stay as an exercise for the reader.

And finally, putting it all together and adding a `main()` routine, we get:

```c
#include <stdlib.h>
#include <ctype.h>
#include <stdio.h>
#define NUMBER 0
#define OP 1
#define END 2

char operation;
int number;
int finished_recursion;
int input_ended;
int op2_set;

int parse(void);
int main() {
	while (1) {
		input_ended = 0;
		op2_set = 0;
		finished_recursion = 0;
		printf("%d\n", parse());
	}
}

int nexttoken(void);
void pushback(int);
int parse(void) {
	int v, n, t;
	
	t = nexttoken();
	if (input_ended)
		return 0;
	if (t == OP) {
		finished_recursion = 1;
		return 0;
	}
	else {
		v = number;
		n = parse();
		if (input_ended)
			return number;
		if (finished_recursion) {
			finished_recursion = 0;
			op2_set = 1;
			return v;
		}
		else {
			if (op2_set) {
				switch (operation) {
					case '+':
					v += n;
					break;
					case '-':
					v -= n;
					break;
					case '*':
					v *= n;
					break;
					case '/':
					v /= n;
					break;
				}
				op2_set = 0;
				pushback(v);
			}
			return parse();
		}
	}
}

int getch(void);
void ungetch(int);

void pushnumberback(int n, int sign) {
	ungetch((n%10)*sign+'0');
	if (n /= 10)
		pushnumberback(n, sign);
}

void pushback(int n) {
	int sign;
	pushnumberback(n, sign = (n > 0 ? 1 : -1));
	if (sign == -1)
		ungetch('-');
}

int nexttoken(void) {
	int c;
	int t;
	int sign;
	while ((c = getch()) != '\n' && isspace((unsigned char) c));
	if (c == '\n')
		return input_ended = END;
	if (c == EOF)
		exit(0);
	if (c == '+' || c == '*' || c == '/') {
		operation = c;
		return OP;
	}
	else if (c == '-' && !isdigit((unsigned char) (t = getch()))) {
		ungetch(t);
		operation = c;
		return OP;
	}
	else {
		number = 0;
		sign = 1;
		if (c == '-') {
			c = t;
			sign = -1;
		}
		for (; isdigit((unsigned char) c); number = number*10 + c - '0', c = getch());
		number *= sign;
		ungetch(c);
		return NUMBER;
	}
}

#define BUFSIZE 100

int buf[BUFSIZE];
int bufp = 0;

int getch(void) {
	return (bufp > 0 ? buf[--bufp] : getchar());
}

void ungetch(int c) {
	if (bufp >= BUFSIZE)
		printf("ungetch: too many characters\n");
	else
		buf[bufp++] = c;
}

```

It is a very important exercise for less experienced users to fully understand this code. `parse()` calls itself recursively twice, it is a great exercise to ensure we're on the right track to understand recursion. Beginners may find it harder to understand how `nexttoken()` works. My suggestion is to go one step at a time, and focus in `parse()` and its recursive implementation, abstracting on how `nexttoken()` is implemented and simply believing that it works by reading input and signalizing if it read an operator, an operand, or end of line.

## Think recursively

I know it is easy to just say "think recursively". It's not a very good hint on how to master recursion, but depending on the problem, exemplifying sometimes is not the way to go. It can be a naturally recursive problem.

### Example 3 - Lowest Common Ancestor

Consider that you have a tree structure, where each node has an unknown number of children. Each node has a pointer to its parent; the root's node parent is itself. You want to write a function that receives 2 nodes, `n1` and `n2`, and outputs the lowest common ancestor between `n1` and `n2`. So, for example, if you had this tree:

![LCA Example Tree]({{ site.baseurl }}{{ site.assets }}lca_example.png)

And you wanted to find the lowest common ancestor between `10` and `14`. The expected result is `0`. How do we approach this? Let's pick the deepest node, `n1`. It will have to go up in the tree and somehow it's going to have to stop when it knows it found the first common ancestor.

If you think for a bit, you will discover that first, we should get to a point where `n1` and `n2` are at the same depth. When they are at the same depth, either their parent is the same - and that is the lowest common ancestor - or it's not, in which case the lowest common ancestor of `n1` and `n2` is the same as the lowest common ancestor between `n1->parent` and `n2->parent`. There. That's just it, we have our recursive algorithm. Is it always that easy? It really depends, and there's no general rule I can give you.

## Mutual recursion

There is one final thing you should care about: functions that recursively call each other. This is not very common, but sometimes it happens. It happens very often if you're building parsers by hand. A very nice traditional example is K&R's dcl program, developed in section 5.12. They present an implementation of a recursive descent parser - a pair of functions that call each other recursively - to read the input according to a simplified version of C's declarations grammar. 

### Example 4 - Simple arithmetic expression evaluator

Let's develop a simple descent recursive parser for a sample grammar. The grammar is a very primitive arithmetic expressions evaluator:

```
expr: op + op 
     | op - op 
     | op * op 
     | op / op 
     | -op
op: number 
   | (expr)
```

This is rudimentary because if you want to combine multiple expressions you are forced to use parentheses. The grammar doesn't allow things like `5+6+8-2`, to do that, you'd have to enter `((5+6)+8)-2`; you can reach this terminal expression by doing the following expansions:

```
expr --> op1 - op2
   op2 --> 2
   op1 --> (expr2)
       expr2 --> op3 + op4
           op4 --> 8
           op3 --> (expr3)
               expr3 --> op5 + op6
                   op5 --> 5
                   op6 --> 6
               return "expr3 = 5+6 = 11"
           return "expr2 = op3 + op4 = 11+8 = 19"
       return "op1 = expr2 = 19"
   return "expr = op1 - op2 = 19 - 2 = 17"
```

The way this works may not seem very intuitive at first. Given our grammar, we will have 2 functions to parse each part of the grammar, `expr()` and `op()`. `expr()` calls `op()` to obtain the value of the next operand, and `op()` calls `expr()` if it reads an opening parentheses. This can be directly derived from the grammar. A good way to handle this inside your head is to abstract yourself on how the other function works when you call it. If you're coding `expr()`, just assume that `op()` gives you the next operand. And when you're coding `op()`, just assume `expr()` gives the value of an expression enclosed in parentheses. They will recursively call each other, I know it can be intimidating and confusing, but if you look at the grammar it makes total sense. So let's start by coding `expr()`. Again, we'll be using `getch()` as a buffer. In fact, I could just use `ungetc()`, from `stdio.h` (even though it only guarantees one character of pushback - that's all we're going to need), but I decided to settle with `ungetch()`. Anyway, this is not important. The code that processes `expr` can be implemented like this:

```c
int expr(void) {
	int c, sign, op1;
	
	sign = 1;
	
	if ((c = getch()) == EOF)
		exit(0);
		
	if (c == '-')
		sign = -1;
	else
		ungetch(c);
	op1 = op()*sign;
	switch ((c = getch())) {
		case '+':
		return op1+op();
		case '-':
		return op1-op();
		case '*':
		return op1*op();
		case '/':
		return op1/op();
		case EOF:
		exit(0);
		default:
		printf("Unknown operation: %c\n", c);
		return 0;
	}
}
```

Looking at the grammar, producing this code to parse an `expr` is not very hard. We always have to call `op()` at least once. Notice the `if (c == '-')` - we're checking if this is a case where `expr` expands into `-op`. Is it? Then save that `-` and call `op()` to give us the operand. It's not? Oops, then we have read something that is part of `op()`, so we push it back to the input, so that `op()` can retrieve it when it's time to. 

The operand can be as complicated as you wish, it can call `expr()` again, which will call `op()`, which will call `expr()`, and that can literally happen hundreds of times. You really don't care, you just know that `op()` gives you whatever the result of all that turns out to be. Again, abstract yourself from that.

So, now that we have a function to parse `expr`, we need the other to parse `op`. Once again, looking at the grammar, we can see that there are two terminal symbols: either the next character is a digit or an opening parentheses. If it's a digit, we will keep reading digits to build up the whole number and return that. If it's an opening parentheses, we are reading an `expr`, so it's `expr()`'s job to parse it, so we call it. Because `expr()` always calls `op()`, we know that this call to `expr()` will, sooner or later, result in yet another call to `op()`: that's the nature of the grammar. Here's the code for `op()`:

```c
int op(void) {
	int c;
	int val = 0;
	if ((c = getch()) == '(') {
		val = expr();
		if ((c = getch()) != ')') {
			fprintf(stderr, "Warning: missing close parentheses.\n");
			ungetch(c);
		}
	}
	else {
		while (isdigit((unsigned char) c)) {
			val = val*10 + c - '0';
			c = getch();
		}
		ungetch(c);
	}
	return val;
}
```

Nasty bugs can show up with bad implementations of this parser. You have to be careful not to accidentally read a character and throw it away. For example, it is very easy to miss the `ungetch()` call in that `else` body. If we missed that, no one really knows what could happen. If we were parsing the first operand of some `expr`, then we probably read a `+`, `-`, `*` or `/`, and because we didn't unget it, after we return, the previous caller - inside `expr()` - would enter the default case inside the switch, and report an unknown operation (`%c` would print a number). Or, if this `op()` call was for the 2nd operand, we might just have read a newline and dumped it, or we might have read a closing parentheses, and now whoever was processing that `( expr )` is going to report a missing close parentheses error. Beautiful, and very hard to track bugs can show up when we're messing with parsers and multiple recursion. Learning how to deal with it is an art.

Putting it all together and adding a `main()` procedure, we get this:

```c
#include <stdio.h>
#include <ctype.h>
#include <stdlib.h>

int expr(void);
void skipblanks(void);
int main() {
	while (1) {
		skipblanks();
		printf("%d\n", expr());
	}
}

int getch(void);
void ungetch(int);
int op(void);

int expr(void) {
	int c, sign, op1;
	
	sign = 1;
	
	if ((c = getch()) == EOF)
		exit(0);
		
	if (c == '-')
		sign = -1;
	else
		ungetch(c);
	op1 = op()*sign;
	switch ((c = getch())) {
		case '+':
		return op1+op();
		case '-':
		return op1-op();
		case '*':
		return op1*op();
		case '/':
		return op1/op();
		case EOF:
		exit(0);
		default:
		printf("Unknown operation: %c\n", c);
		return 0;
	}
}

int op(void) {
	int c;
	int val = 0;
	if ((c = getch()) == '(') {
		val = expr();
		if ((c = getch()) != ')') {
			fprintf(stderr, "Warning: missing close parentheses.\n");
			ungetch(c);
		}
	}
	else {
		while (isdigit((unsigned char) c)) {
			val = val*10 + c - '0';
			c = getch();
		}
		ungetch(c);
	}
	return val;
}

void skipblanks(void) {
	int c;
	while (isspace((unsigned char) (c = getch())));
	ungetch(c);
}

#define BUFSIZE 100

int buf[BUFSIZE];
int bufp = 0;

int getch(void) {
	return (bufp > 0 ? buf[--bufp] : getchar());
}

void ungetch(int c) {
	if (bufp >= BUFSIZE)
		printf("ungetch: too many characters\n");
	else
		buf[bufp++] = c;
}
```

As I said, the parser is rudimentary and the grammar is primitive. Because there is no error checking and recovery, if you input an ill-formed expression, the parser will be very confused. My goal with this exercise was not to teach you about compilers, but instead, show you a good example of multiple recursion.

Mutual recursion doesn't have to be only between 2 functions. Ok, I know it gets scary by this point, but this summer, for my internship, one of the components I had to develop used a semi-triple recursive call. Let me explain it: we were dealing with an interface to control a physical robot; this robot served as a 3rd automatic player for a game with 2 human beings. We wanted to make the robot look at a specific position in the game board before making his move. The problem is, "making his move" can yield very different actions in the board. Depending on the kind of action the robot decided to do, he might need to look at a place, make an action, then look at another place, make another move, etc. The robot interface had a callback system where you registered the type of events you'd want to be informed about when they were finished (like "Hey, my head just finished turning to look at that place you told me, you can now execute your code"). So, I created a function to schedule the execution of an action, which asks the callback system to notify function `X` when the robot finished moving. Then, function `X` calls the action's execute method, which makes the move in the board. After making the move, this function needs to know if there are any other actions that can be done before skipping; if there are, it calls the scheduler function with the next action in the list, and the whole cycle resets again, until eventually the execute function sees that no more actions of that type can be done and doesn't call the scheduler anymore, and the robot clicks the "skip" button.

Mastering recursion makes the difference between brilliant programmers and average ones. Throughout this article, I've showed you a couple of techniques to help you better understand and deal with recursion:

* Base Case and Build
* Exemplify (includes making a full example of a call stack - VERY IMPORTANT, helps a lot)
* Think recursively
* Use abstraction

This post is getting rather long, but after all, I hope I helped you and contributed to improve your skills. Want to learn more cool stuff? Stay tuned, come back to this blog regularly! Oh, and keep this in mind:

> But always remember, no matter how lowly or exalted a hacker you are, you are a child process of the universe, no less than the disk controller, or the stack frame mechanism.

in *Expert C Programming - Deep C Secrets*, by Peter Van Der Linden - page 347, 3rd paragraph.

See you around! And watch out for your dreams, try not to dive into a buggy recursive implementation that will loop forever and ever, to infinity and beyond.
