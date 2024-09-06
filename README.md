## Introductions

**JOHN:**

Hi (Uncle) Bob! You and I have each written books on software design.
We agree on some things, but there are some pretty big differences of
opinion between my recent book *A Philosophy of Software Design*
(herafter "APOSD") and your classic book *Clean Code*. Thanks for
agreeing to discuss those differences here.

**UB:**

My pleasure John.  Before we begin let me say that I've read through your book and I found it very enjoyable, and full of valuable insights.  

**JOHN:**

I'd like to discuss three topics with you: method length, comments,
and test-driven development. But before getting into these,
let's start by comparing overall philosophies. When you hear about a
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

**UB:** 

I agree with your approach. A discipline or technique should make the job of programmers easier. I would add that the programmer we want to help most is not the author.  The programmer whose job we want to make easier is the programmer who must read and understand the code written by others (or by themself a week later).  Programmers spend far more hours reading code than writing code, so the activity we want to ease is that of reading.

**JOHN:**
I agree: ease of reading is much more important for code than ease of writing.

## Method Length

**JOHN:** 

Our first area of disagreement is method length.
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
	
**UB:** 

Let me stop you there with a challenge.  Is it true that each extracted small function adds complexity?  Or, rather, does ach extracted small function *expose* complexity that was previously hidden, and gives it a name.

Generally, when I extract small functions from larger functions, those smaller functions are private.  They do not add to the overall interface of the containing class.  That class remains narrow and deep.  No outside user sees those small private functions.

By the same token, those small private functions partition and organize the internal structure with names to guide the reader.  That internal structure is particularly helpful if it describes the top-down functional decomposition of the problem. 

Because of this I disagree with your following statement.

**JOHN:**

The advice in *Clean Code* is too extreme, encouraging developers to
create teeny-tiny methods that suffer from both of these problems.

To illustrate my concern, let's consider a piece of code that follows
the *Clean Code* advice about making methods super-short.

It's the PrimeGenerator class on pages 145-146 of *Clean Code*, written
in Java, which generates the first N prime numbers:

**UB:**

Ah, yes.  The `PrimeGenerator`.  This code comes from the 1982 paper on [*Literate Programming*](https://www.cs.tufts.edu/~nr/cs257/archive/literate-programming/01-knuth-lp.pdf) written by Donald Knuth.  The program was originally written in Pascal, by Knuth's WEB system.  I translated it to Java.  

The original code was a larger program that not only generated prime numbers but also printed them in columns and pages with page numbers.  It was one big monolithic function that I was using as an example of how to take large functions apart into several smaller classes.  The three new classes I created were `PrimePrinter`, `RowColumnPagePrinter` and `PrimeGenerator`.  

Once the classes were extracted, the `PrimeGenerator` looked like this: (which I did not publish in the book.)

	public class PrimeGenerator {
	  protected static int[] generate(int n) {
	    int[] p = new int[n];
	    ArrayList<Integer> mult = new ArrayList<Integer>();
	    p[0] = 2;
	    mult.add(2);
	    int k = 1;
	    for (int j = 3; k < p.length; j += 2) {
	      boolean jprime = false;
	      int ord = mult.size();
	      int square = p[ord] * p[ord];
	      if (j == square) {
	        mult.add(j);
	      } else {
	        jprime=true;
	        for (int mi = 1; mi < ord; mi++) {
	          int m = mult.get(mi);
	          while (m < j)
	            m += 2 * p[mi];
	          mult.set(mi, m);
	          if (j == m) {
	            jprime = false;
	            break;
	          }
	        }
	      }
	      if (jprime)
	        p[k++] = j;
	    }
	    return p;
	  }
	} 

18 years ago I refactored that wad of code in order to break it up into a few reasonbaly sized and reasonably named chunks so that my readers could see how large functions, that violate the Single Responsibility Principle, can be broken down into a few smaller well-named classes containing a few smaller well-named functions.  

I think it's somewhat better than where it started.

It was NOT my intent, in this chapter, to teach my readers how to write the most optimal prime number generator.  That was the last thing on my mind, and the last thing I wanted on theirs.

**JOHN:**

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

**JOHN:**

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

**UB:**

Thirty minutes?  Wow.  Maybe I'm just dense but it took me a lot longer than that to understand it.  I even had Knuth's original description and I still had to puzzle it out for a really long time. I even had the code that I expect you will show next, and since it has been eighteen years, it *still* took me a lot longer than 30 minutes to work it all out.  

**JOHN:**

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

**UB:**

I agree.  Though I would not have agreed eighteen years ago when I was in the throes of refactoring this.  At the time that name made perfect sense to me.  It makes perfect sense to me now, too -- but that's because I understand the algorithm.  When I returned to the algorithm a few days ago, I also struggled with those names.  Once you understand the algorithm the names make perfect sense, and that understanding breaks the entaglement.

So, a good critique of those names is that they depend upon some understanding of the algorithm.

But again, teaching this algorithm was not the intent of that chapter.  

**JOHN:**

Going back to my introductory remarks about complexity, splitting up
`isNot...` into three methods doesn't reduce the amount of information
you have to have. It just splits it up and spreads it out, so it isn't as
obvious that you need to read all three methods together. And, it's harder
to see the overall structure of the code because it's split up: readers have
to flip back and forth between the methods, effectively reconstructing a
monolithic version in their minds. Thus, this code will be easiest to
understand if it's all together in one place.

**UB:**

I disagree.  What you say may be true in this particular case; but it is not true in the general case.  For example:

	for (Employee e : employees)
	  if (e.shouldPayToday())
		  e.pay();

This would not be made more understandable if we replaced those two function calls with the their implementations.  Such a replacement would simply obscure the intent.

**JOHN:**

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

**UB:** 

In general I believe in the principle of small well-named functions.  I think it's fair to critique this particular example -- but there are many other examples in the book that I think you'd find more difficult to criticize. 

Generally speaking if you can break a large function into several well-named smaller function, and by doing so expose the high level functional decomposition, then that's a good thing. 

**JOHN:** 

Let's consider another example: `isMultipleOfNthPrimeFactor`.
This method contains a single statement that calls `smallestOdd...`; the only
functionality this method contributes is a `==` comparison.
Why does it make sense for this to be a separate method rather than
just invoking `smallestOdd...` directly from `isNot...`?

**UB:** 

Functions like this are trying to describe a logical equivalence that might help the reader understand things.  In this case it is saying that the smallest odd nth multiple not less than the candidate _is_ a multiple of the nth prime factor.  That's an interesting equivalence.  Or, at least, I found it to be interesting at the time I wrote it.

## Comments

**JOHN:** 

Let's move on to the second area of disagreement: comments.
Here is what *Clean Code* says about comments (page 54):

> The proper use of comments is to compensate for our failure to express
  ourselves in code. Note that I use the word failure. I meant it.
  Comments are always failures. We must have them because we cannot always
  figure out how to express ourselves without them, but their use is not
  a cause for celebration... Every time you write a comment, you should
  grimace and feel the failure of your ability of expression.

I have to be honest: I was horrified when I first read this text, and it
still makes me cringe. This stigmatizes writing comments. 

**UB:**

That chapter begins with these words:
>*Nothing can be quite so helpful as a well placed comment.*

**JOHN:** 

Junior developers
will think "if I write comments, people may think I've failed, so the
safest thing is to write no comments." 

**UB:** 

Well, perhaps, but only if they didn't read the chapter.  The chapter walks through a series of comments, some bad, some good.  

**JOHN:** 

Comments are not the problem;
they are the solution. The problem is that there is a lot of important
information that simply cannot be expressed in code. When a programmer
writes comments, they have not failed to express themselves; they have
succeeded in providing important information that makes code easier to
understand.

**UB:**

It's very true that there is important information that is not, or cannot, be expresssed in code.  That's a failure.  A failure of our languages, or of our ability to use them to express ourselves.  In every case a comment is a failure of our ability to use our languages to express our intent.

And we fail at that very frequently, and so comments are a necessary evil.  If we had the perfect programming language TM we would never write another comment.

**JOHN:** 

Nowhere near enough comments are being written today, and *Clean Code*
is one of the major reasons why.  The result is unnecessarily cryptic code all over the world, which drives
up development costs.

**UB:**

*Oh, baloney!*  Again, if you read the chapter you are not going to be led to write uncommented code.  Hopefully you will be led to strive to write fewer comments by using the code to express your intent.

**JOHN:** 

Let's consider the `PrimeGenerator` class. There is not a single comment
in this code other than the one I added at the top; presumably you
think this is appropriate? There are two major reasons why comments
are needed.

**UB:**

If I was putting that code into a library then some comments would be appropriate. But his code is not getting put into a library.  It was created to make the point that large functions can be broken down into smaller classes containing smaller functions. Adding lots of explanatory comments to this code would have detracted from that point.

**JOHN:** 

First, there is important infomation that cannot be represented in the code.
For example:

* One of the key ideas behind this code is to improve performance by
  avoiding divisions, which are relatively expensive. Readers will wonder
  "why didn't you just use `mod` instead of all this complexity with
  prime multiples?"
	
**UB:**

Why is that important to understanding the lesson of that chapter?

Consider my audience.  I was writing for software developers in 2008, and I was explaining how large functions can be split into several classes.  Why would I clutter up that explanatory code with a comment that they would have utterly no interest in?	
	
**JOHN:** 

* The first multiple for each new prime number is computed by squaring
  the prime, rather than multiplying it by 3. This is mysterious:
  why is it safe to skip the intervening odd multiples?

**UB:**

Why indeed?  This is not at all easy to understand.  I needed to go on an hour long bike ride to finally work it out.  Again, maybe I'm dense, but this is not an easy thing to understand.  

And again I was not teaching software developers the most optimal way to calculate prime numbers.  I was teaching them how and when to break large functions up into smaller classes and smaller functions.

**JOHN:** 

There is no way to explain either of these things in the code; without
comments, readers are left to figure them out on their own. The students
in my class are generally unable to figure out either of these in the
30 minutes I give them, but comments would have
allowed them to understand in a minute or two. Going back to my
introductory remarks, this is an example where information is important,
but it isn't made available.

**UB:**

That bike ride I took was *after* I had read the comment you put in your version of this function.  So -- again, maybe I'm dense, but that comment didn't help me at all -- it was just a jumble of numbers that I could not map to the problem.  Now that I understand the algorithm (again), I understand your comment.  But the reverse was not true.

**JOHN:** 

The second reason for comments is abstraction. Simply put, without
comments there is no way to have abstraction or modularity. 

**UB:**

Um...

**JOHN:** 

Abstraction
is one of the most important components of good software design.
I define an abstraction as "a simplified way of thinking about something
that omits unimportant details." 

**UB:**

Long ago, in a 1995 book, I defined it as:
>*The amplification of the essential and the elimination of the irrelevant.*

**JOHN:** 

The most obvious example of an abstraction
is a method: it should be possible to use a method without reading its code.
The way we achieve this is by writing a header comment that describes
the method's *interface* (all the information someone needs in order
to invoke the method). If the method is well designed, the interface will be
much simpler than the code of the method (it omits implementation details),
so the comments reduce the amount of information people must have in
their heads.

**UB:**

	addSongToLibrary(String title, String[] authors, int durationInSeconds);

This seems like a very nice abstraction to me, and I cannot imagine how a comment might improve it.

**JOHN:** 
	
Consider the `isNot...` method in `PrimeGenerator`. This method has no header
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

Bob, can you explain why you chose not to include any comments in the `PrimeGenerator`
class? How is code better when it has no comments?

**UB:** 

As I explained earlier, those kinds of comments in this code would have detracted from the lesson of the chapter, which was how and why to break large functions into smaller classes containing smaller functions, and not to understand some ancient prime generator algorithm. 
