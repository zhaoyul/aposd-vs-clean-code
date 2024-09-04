**JOHN:** Hi Bob! You and I have each written books on software design.
We agree on some things, but there are some pretty big differences of
opinion between my recent book *A Philosophy of Software Design*
(herafter "APOSD") and your classic book *Clean Code*. Thanks for
agreeing to discuss those differences here.

Let's start by comparing overall philosophies. When you hear about a
new idea related to software design, how do you decide whether or not
to endorse that idea?

I'll go first. For me, the fundamental goal of software design is
to make it easy to understand and modify the system. I use the term
"complexity" to refer to things that make it hard to understand and
modify a system. The most important contributors
to complexity relate to information:
* How much information must a developer have in their head in order to
  carry out a task?
* How accessible and obvious is the information that the developer needs?

The more information a developer needs to have, the harder it will be
for them to work on the system. Things get even worse if the required
information isn't obvious. The worst case is when there is a crucial
piece of information hidden in some far-away piece of code
that the developer has never heard of.

When I'm evaluating an idea related to software design, I ask whether
it will reduce complexity. This usually means either reducing the amount
of information a developer has to know, or making the required information
more obvious.

Now over to you: are there general principles that you use when deciding
which ideas to endorse?

**BOB:** *Please fill in*

**JOHN:** Our first area of disagreement is method length.
On page 34 of *Clean Code* you say "The first rule of functions is that
they should be small. The second rule of functions is that
*they should be smaller than that*." Later on, you say "Functions
should hardly ever be 20 lines long" and suggest that functions
should be "just two, three, or four lines long". On page 35, you
say "Blocks within `if` statements, `else` statements, `while` statements,
and so on should be one line long. Probably that line should be a function
call." I couldn't find anything in *Clean Code* to suggest that a function
could ever be too short.

I agree that making methods smaller can often reduce complexity, but it's
important to recognize that there are limits. If methods are chopped up
too much, it actually makes the code harder to understand:
* The methods become *entangled*: you can't understand one method
  without simultaneously understanding other methods.
* Each new method introduces a new interface, and each interface adds
  complexity. Interfaces are good, but more interfaces are not better!

The advice in *Clean Code* is too extreme, encouraging developers to
create teeny-tiny methods that suffer from both of these problems.

To illustrate my concern, let's consider a piece of code that follows
the *Clean Code* advice about making methods super-short.
It's the PrimeGenerator class on pages 145-146 of *Clean Code*, written
in Java, which generates the first N prime numbers:
```
// This code is a copy of Listing 10-8 on pp. 145-146 of "Clean Code",
// by Robert C. Martin

package literatePrimes;

import java.util.ArrayList;

public class PrimeGenerator {
    private static int[] primes;
    private static ArrayList<Integer> multiplesOfPrimeFactors;

    protected static int[] generate(int n) {
        primes = new int[n];
        multiplesOfPrimeFactors = new ArrayList<Integer>();
        set2AsFirstPrime();
        checkOddNumbersForSubsequentPrimes();
        return primes;
    }

    private static void set2AsFirstPrime() {
        primes[0] = 2;
        multiplesOfPrimeFactors.add(2);
    }

    private static void checkOddNumbersForSubsequentPrimes() {
        int primeIndex = 1;
        for (int candidate = 3;
             primeIndex < primes.length;
             candidate += 2) {
            if (isPrime(candidate))
                primes[primeIndex++] = candidate;
        }
    }

    private static boolean isPrime(int candidate) {
        if (isLeastRelevantMultipleOfLargerPrimeFactor(candidate)) {
            multiplesOfPrimeFactors.add(candidate);
            return false;
        }
        return isNotMultipleOfAnyPreviousPrimeFactor(candidate);
    }

    private static boolean
    isLeastRelevantMultipleOfLargerPrimeFactor(int candidate) {
        int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
        int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor;
        return candidate == leastRelevantMultiple;
    }

    private static boolean
    isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
        for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
            if (isMultipleOfNthPrimeFactor(candidate, n))
                return false;
        }
        return true;
    }

    private static boolean
    isMultipleOfNthPrimeFactor(int candidate, int n) {
        return candidate ==
                smallestOddNthMultipleNotLessThanCandidate(candidate, n);
    }

    private static int
    smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
        int multiple = multiplesOfPrimeFactors.get(n);
        while (multiple < candidate)
            multiple += 2 * primes[n];
        multiplesOfPrimeFactors.set(n, multiple);
        return multiple;
    }
}
```

I'd encourage everyone to read over this code and draw your own conclusions
about it before continuing with the discussion. Did you find the code
easy to understand? If so, why? If not, what makes it complex?

I use this code for an in-class exercise in a software design course
that I teach at Stanford. I give students about 30 minutes to read
over the code and discuss it in pairs. Invariably, the students
can't get comfortable with
the code in that time; they are unable to understand how the
algorithm works or why. I also struggled to understand this code.
In the class, we discuss why the code is so impenetrable, then students
rewrite it to simplify it.

I think there are many design problems with this code, but for now I'll
focus on method length (I'll come back to this code later when we
discuss comments). This code is chopped up so much (8 teeny-tiny methods)
that it's difficult to read. For starters, consider the
`isNotMultipleOfAnyPreviousPrimeFactor` method. This method invokes
`isMultipleOfNthPrimeFactor`, which invokes
`smallestOddNthMultipleNotLessThanCandidate`. These methods are entangled:
in order to understand
`isNot...` you have to read the other two
methods and load all of that code into your mind at once. For example,
`isNot...` has side effects (it modifies `multiplesOfPrimeFactors`), but
you can't see that unless you read all three methods.

Going back to my introductory remarks about complexity, splitting up
`isNot...` into three methods doesn't reduce the amount of information
you have to have. It just splits it up and spreads it out, so it isn't as
obvious that you need to read all three methods together. And, it's harder
to see the overall structure of the code because it's split up: readers have
to flip back and forth between the methods, effectively reconstructing a
monolithic version in their minds. Thus, this code will be easiest to
understand if it's all together in one place.

Not only is `isNot...` entangled with the methods it calls, it's also
entangled with its callers. `isNot...` only works if it is invoked in
a loop where `candidate` increases monotonically. To convince yourself
that it works, you have to find the code that invokes `isNot...` and
make sure that `candidate` never decreases from one call to the next.
Separating `isNot...` from the loop that invokes it makes it harder
for readers to convince themselves that it works.

To me, all of the methods in the class are entangled: in order to
understand the class I had to load all of them into my mind
at once. I was constantly flipping back and forth between the methods
as I read the code. This is a red flag indicating
that the code has been over-decomposed.

Bob, can you help me understand why you divided the code into such
tiny methods?
Is there some benefit to having so many methods that I have missed?

**BOB:** *please fill in*

**JOHN:** Let's consider another example: `isMultipleOfNthPrimeFactor`.
This method contains a single statement that calls `smallestOdd...`; the only
functionality this method contributes is a `==` comparison.
Why does it make sense for this to be a separate method rather than
just invoking `smallestOdd...` directly from `isNot...`?

**BOB:** *please fill in*

**JOHN:** Let's move on to the second area of disagreement: comments.
Here is what *Clean Code* says about comments (page 54):

> The proper use of comments is to compensate for our failure to express
  ourselves in code. Note that I use the word failure. I meant it.
  Comments are always failures. We must have them because we cannot always
  figure out how to express ourselves without them, but their use is not
  a cause for celebration... Every time you write a comment, you should
  grimace and feel the failure of your ability of expression.

I have to be honest: I was horrified when I first read this text, and it
still makes me cringe. This stigmatizes writing comments. Junior developers
will think "if I write comments, people may think I've failed, so the
safest thing is to write no comments." Comments are not the problem;
they are the solution. The problem is that there is a lot of important
information that simply cannot be expressed in code. When a programmer
writes comments, they have not failed to express themselves; they have
succeeded in providing important information that makes code easier to
understand.

Nowhere near enough comments are being written today, and *Clean Code*
is one of the major reasons why.
The result is unnecessarily cryptic code all over the world, which drives
up development costs.

Let's consider the PrimeGenerator class. There is not a single comment
in this code other than the one I added at the top; presumably you
think this is appropriate? There are two major reasons why comments
are needed.

First, there is important infomation that cannot be represented in the code.
For example:
* One of the key ideas behind this code is to improve performance by
  avoiding divisions, which are relatively expensive. Readers will wonder
  "why didn't you just use `mod` instead of all this complexity with
  prime multiples?"
* The first multiple for each new prime number is computed by squaring
  the prime, rather than multiplying it by 3. This is mysterious:
  why is it safe to skip the intervening odd multiples?

There is no way to explain either of these things in the code; without
comments, readers are left to figure them out on their own. The students
in my class are generally unable to figure out either of these in the
30 minutes I give them, but comments would have
allowed them to understand in a minute or two. Going back to my
introductory remarks, this is an example where information is important,
but it isn't made available.

The second reason for comments is abstraction. Simply put, without
comments there is no way to have abstraction or modularity. Abstraction
is one of the most important components of good software design.
I define an abstraction as "a simplified way of thinking about something
that omits unimportant details." The most obvious example of an abstraction
is a method: it should be possible to use a method without reading its code.
The way we achieve this is by writing a header comment that describes
the method's *interface* (all the information someone needs in order
to invoke the method). If the method is well designed, the interface will be
much simpler than the code of the method (it omits implementation details),
so the comments reduce the amount of information people must have in
their heads.

Consider the `isNot...` method in PrimeGenerator. This method has no header
comment, so readers are forced to read the method's code to figure out how
to use it. This means that readers have to load more information into their
heads, which makes it harder to work in the code.
Here are a couple of things that would need to be described in
the header comment for `isNot...`:
* Its side effects (modifying `multiplesOfPrimeFactors`); without comments,
  the entire tree of methods underneath `isNot...` must be read to
  discover the side effects.
* The fact that it works only if invoked with ascending argument values (this
  constraint is unlikely to be obvious even after reading the code).

Bob, can you explain why you chose not to include any comments in the PrimeGenerator
class? How is code better when it has no comments?

**BOB:** *please fill in*