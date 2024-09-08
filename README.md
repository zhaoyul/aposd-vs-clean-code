## Introductions

**JOHN:**

Hi (Uncle) Bob! You and I have each written books on software design.
We agree on some things, but there are some pretty big differences of
opinion between my recent book *A Philosophy of Software Design*
(herafter "APOSD") and your classic book *Clean Code*. Thanks for
agreeing to discuss those differences here.

**UB:**

My pleasure John.  Before we begin let me say that I've read through your book and I found it very enjoyable, and full of valuable insights.  There are some things I disagree with you on, such as TDD, and Abstraction-First incrementalism, but overall I enjoyed it a lot.

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

* How much information must a developer have in their head in order to carry out a task?
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
  
The advice in *Clean Code* is too extreme, encouraging developers to
create teeny-tiny methods that suffer from both of these problems.
	
**UB:** 

Let me stop you there with a challenge.  Is it true that extracting small function adds complexity?  Or, rather, do the extracted small functions make the already existing complexity explict and give it names and a defined interfaces?  Doesn't such extraction give the reader the opportunity to understand the the code in small pieces, and to ignore those pieces that the interfaces make obvious.

Generally, when I extract small functions from larger functions, those smaller functions are private.  They do not add to the overall interface of the containing class.  That class remains narrow and deep.  No outside user sees those small private functions.

By the same token, those small private functions partition and organize the internal structure with names to guide the reader.  That internal structure is particularly helpful if it describes the top-down functional decomposition of the problem. 

Extracting small functions allows me to separate higher level code from lower level code so that readers of the code can read the high level without being forced to see the details they don't want or need.  The names given to the small extracted functions tell the reader what they need to know.

I call this: *being polite*.  

**JOHN:**

Yes, each extracted small function adds complexity, in the form of its
interface. As I'm reading the code, I see a function call rather than
code. I now have to learn how this
function works in general (its interface) and think about whether that
interface actually works in the current context. Without the function
call I just read the code directly and decide whether it works; I don't
go through the intermediate step of visualizing an "interface" for
that code (e.g., I don't have to match actual parameters to formal
parameters or think about how it would work in a more general case).

>=====you may want to reword the following paragraph.

I disagree with your comment about exposing complexity: exposing
complexity is bad! One of the most important goals of software design
is to *hide* complexity, so that most people don't have to be aware of
it most of the time.

That said, one of the best ways to hide complexity
is to encapsulate it in a method with a simple interface (callers only
need to understand the interface, not the messy details). If done
right, splitting up functions will hide complexity, but this only
works if the new interface I have to learn is simpler (ideally,
a *lot* simpler) than the function's code, which I no longer have to
learn.

If you split too far, you end up with functions whose interfaces
are just as complex as their implementations, so there is no
benefit. In the worst case, in order to use the function I end up
having to read all its code; in this case there is no complexity hidden,
plus I have to learn a new interface: my cognitive load
went up, not down.

**UB:**

Before a function is extracted it exists as a snippet of code in a larger function.  That snippet has an interface both at it's entry point and it exit point -- but that interface is not explicit. It exists within the current scope, has access to all the variables of that scope, and is "called" by the flow of control.  By extracting the function that snippet turns into a black box with a restricted access, and an explicit call to a named and defined interface.

I don't think we disagree in principle.  We both agree that splitting up functions is a good way to manage complexity.  It seems to me that all we are quibbling about is the degree.  I believe in the principle of many small named functions.  Perhaps you believe in the principle of a few moderately sized functions.  To each his own.

The strategy that I use for deciding how far to take extraction is the old rule that a function should do "*One Thing*".  If I can *meaninfully* extract one function from another, then the original function did more than one thing.  "Meaningfully" means that the extracted functionality can be named; and that it does less than the original function.

An example of a meaningless extraction is one in which you extract the entire body of a function, reducing that function to a single call to the extracted function.  That new function does not do less than the original, and it cannot be given a name that differs in meaning from the original.

There are also times when a stretch of code is just so obvious that an extraction, albeit meaningful, would obscure more than it reveals.  That's a judgement call.  And it is in that judgement that the two of us likely differ.

**JO:** ===I've done some revising.  ##### So have I.

Given this, maybe we can revise the arguments above to focus on our
agreement and avoid uninteresting nitpicking? For example, maybe you
could say that you also agree that it's possible to take decomposition
too far, and explain how you decide when things have gone too far?
On the other hand, if you believe that introducing new functions
comes with no cost (in terms of the additional interfaces), that's an
important disagreement that needs to be highlighted.

**JOHN:**

I think it will be easier to clarify our differences if we consider
a specific code example. Let's look at the PrimeGenerator class from
*Clean Code*, which is Listing 10-8 on pages 145-146. This Java class
generates the first N prime numbers:

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

Before we dive into this code, I'd encourage everyone reading
this article to take time to read over the code and draw your own conclusions
about it. Did you find the code easy to understand? If so, why? If not, what
makes it complex?

Also, Bob, can you confirm that you stand by this code (i.e. the code
properly exemplifies the design philosophy of *Clean Code* and this
is the way you believe the code should appear if it were used in
production)?

**UB:**

Ah, yes.  The `PrimeGenerator`.  This code comes from the 1982 paper on [*Literate Programming*](https://www.cs.tufts.edu/~nr/cs257/archive/literate-programming/01-knuth-lp.pdf) written by Donald Knuth.  The program was originally written in Pascal, and was automatically generated by Knuth's WEB system.  I translated it to Java.  

The example appears in a chapter named *Classes*.  I used it as a demonstration for how to partition a large and messy legacy function into a set of smaller classes and functions.

The original code was a large-ish program that not only generated prime numbers but also printed them in columns and pages with page numbers.  It was one big monolithic function.  The three new classes I extracted from it were `PrimePrinter`, `RowColumnPagePrinter` and `PrimeGenerator`.  

Once the classes were extracted, the `PrimeGenerator` looked like this: (which I did not publish in the book.)  The variable names were Knuth's.

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

I refactored that wad of code in order to break it up into a few reasonbaly sized and reasonably named chunks so that my readers could see how large functions, that violate the Single Responsibility Principle, can be broken down into a few smaller well-named classes containing a few smaller well-named functions.  

I think it's somewhat better than where it started.

It was NOT my intent, in this chapter, to teach my readers how to write the most optimal prime number generator.  That was the last thing on my mind, and the last thing I wanted on theirs.

**JOHN:**

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

So, a good critique of those names is that they depend, to some extent, upon gaining some understanding of the algorithm.

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

In general I believe in the principle of small well-named functions. Generally speaking if you can break a large function into several well-named smaller function, and by doing so expose the high level functional decomposition, then that's a good thing. 

In this case I think I improved the structure and naming of the original quite a bit.  I think you can make the argumennt that I did not improve it enough.  However, the goal of this chapter, and the goal of this example, were not to show the best possible prime number generator.  Rather it was to present the strategy for breaking up large functions into smaller classes and functions.  In that regard I think the example was a success.

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

It goes on to say that comments are a *necessary* evil.

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

If I was putting that code into a library then some comments would be appropriate. But his code is not getting put into a library.  It was created to make the point that large functions can be broken down into smaller classes containing smaller functions. Adding lots of explanatory comments would have detracted from that point.

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

**UB:**

I agree.

**JOHN:**

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

As I explained earlier, those kinds of comments in this code would have detracted from the lesson of the chapter, which was how and why to break large functions into smaller classes containing smaller functions, and not to understand Knuth's ancient prime generator algorithm. 

## John's Rewrite of PrimeGenerator

**JOHN:**

I mentioned that I ask the students in my software design class to rewrite
PrimeGenerator to fix all of its design problems. Here is my rewrite:

	package literatePrimes;

	import java.util.ArrayList;

	public class PrimeGenerator2 {

	    /**
	     * Computes the first prime numbers; the return value contains the
	     * computed primes, in increasing order of size.
	     * @param n
	     *      How many prime numbers to compute.
	     */
	    public static int[] generate(int n) {
	        int[] primes = new int[n];

	        // Used to test efficiently (without division) whether a candidate
	        // is a multiple of a previously-encountered prime number. Each entry
	        // here contains an odd multiple of the corresponding entry in
	        // primes. Entries increase monotonically.
	        int[] multiples = new int[n];

	        // Index of the last value in multiples that we need to consider
	        // when testing candidates (all elements after this are greater
	        // than our current candidate, so they don't need to be considered).
	        int lastMultiple = 0;

	        // Number of valid entries in primes.
	        int primesFound = 1;

	        primes[0] = 2;
	        multiples[0] = 4;

	        // Each iteration through this loop considers one candidate; skip
	        // the even numbers, since they can't be prime.
	        candidates: for (int candidate = 3; primesFound < n; candidate += 2) {
	            if (candidate >= multiples[lastMultiple]) {
	                lastMultiple++;
	            }

	            // Each iteration of this loop tests the candidate against one
	            // potential prime factor. Skip the first factor (2) since we
	            // only consider odd candidates.
	            for (int i = 1; i <= lastMultiple; i++) {
	                while (multiples[i] < candidate) {
	                    multiples[i] += 2*primes[i];
	                }
	                if (multiples[i] == candidate) {
	                    continue candidates;
	                }
	            }
	            primes[primesFound] = candidate;

	            // Start with the prime's square here, rather than 3x the prime.
	            // This saves time and is safe because all of the intervening
	            // multiples will be detected by smaller prime numbers. As an
	            // example, consider the prime 7: the value in multiples will
	            // start at 49; 21 will be ruled out as a multiple of 3, and
	            // 35 will be ruled out as a multiple of 5, so 49 is the first
	            // multiple that won't be ruled out by a smaller prime.
	            multiples[primesFound] = candidate*candidate;
	            primesFound++;
	        }
	        return primes;
	    }
	}


Everyone can read this and decide for themselves whether they think
it is easier to understand than the original. I'd like to mention a
couple of overall things:
* There is only one method. I didn't subdivide it because I felt the
  method already divides naturally into pieces that are distinct
  and understandable. It didn't seem to me that pulling out
  methods would improve readability significantly. When students
  rewrite the code, they typically have 2 or 3 methods, and those are
  usually OK too.
* There are a *lot* of comments. It's extremely rare for me to
  write code with this density of comments. Most methods
  I write have no comments in the body, just a header comment
  describing the interface. But this code is subtle and tricky,
  so it needs a lot of comments to make the subtleties clear to
  readers. Even with all this additional explanatory material
  my version is a bit shorter than the original (65 lines vs. 70).

**UB:**

I presume this is a complete rewrite.  My guess is that you worked to understand the algorithm from Clean Code and then wrote this from scratch.  If that's so, then fair enough.

That's not what I did.  I *refactored* Knuth's algorithm in order to give it a little structure.  

Having said that, your version is much better than either Knuth's or mine.  I could not have used it in the chapter I was writing *because* it is still in a single function, and I needed to show the partioning.

I wrote that chapter 18 years ago, so it's been a long time since I saw and understood this algorithm.  When I first saw your challenge I thought: "Oh, I can figure out my own code!"  But, no.  I could see all the moving parts, but I could not figure out why those moving parts generated a list of prime numbers.  

So then I looked at your code.  I had the same problem.  I could see all the moving parts, all with comments, but I still could not figure out why those moving parts generated a list of prime numbers.

Figuring that out required a lot of staring at the ceiling, closing my eyes, visualizing, and riding my bike.

Among the problems I had were the comments you wrote.  Let's take them one at a time.

	    /**
	     * Computes the first prime numbers; the return value contains the
	     * computed primes, in increasing order of size.
	     * @param n
	     *      How many prime numbers to compute.
	     */
	    public static int[] generate(int n) {
				
It seems to me that this would be better as:

	public static int[] generateNPrimeNumbers(int n) {
		
or if you must:

	//Return the first n prime numbers
	public static int[] generate(int n) {

I'm not opposed to Javadocs as a rule; but I write them only when absolutely necessary. I also have an aversion for descriptions and `@param` statements that are perfectly obvious from the function signature.  

The next comment cost me a good 20 minutes of puzzling things out.

	        // Used to test efficiently (without division) whether a candidate
	        // is a multiple of a previously-encountered prime number. Each entry
	        // here contains an odd multiple of the corresponding entry in
	        // primes. Entries increase monotonically.

First of all I'm not sure why the "division" statement is necessary.  I'm old school so I expect that everyone knows to avoid division in inner loops if it can be avoided.  But maybe I'm wrong about that...

Also, the *Sieve of Eratosthenes* does not do division, and is a lot easier to understand *and explain* than this algorithm.  So why this particular algorithm?  I think Knuth was trying to save memory -- and in 1982 saving memory was important.  This algorithm uses a lot less memory than the sieve.

Then came the phrase: `Each entry here contains an odd multiple...`.  I looked at that, and then at the code, and I saw: `multiples[0] = 4;`.  

"That's not odd" I said to myself.  "So maybe he meant even."

So then I looked down and saw: `multiples[i] += 2*primes[i];`  

"That's adding an even number!" I said to myself.  "I'm pretty sure he meant to say 'even' instead of 'odd'."

I hadn't yet worked out what the `multiples` array was.  So I thought it was perfectly reasonable that it would have even numbers in it, and that your comment was simply an understandable word transposition.  After all, there's no compiler for comments so they suffer from the kinds of mistakes that humans often make with words.

It was only when I got to `multiples[primesFound] = candidate*candidate;` that I started to question things.  If the `candidate` is prime, shouldn't `prime*prime` be odd in every case beyond 2?  I had to do the math in my head to prove that.  (2n+1)(2n+1) = 4n^2+4n+1 ... Yeah, that's odd.

OK, so the `multiples` array is full of odd multiples, except for the first element, since it will be muliples of 2. 

So perhaps that comment should be:

	 // multiples of corresponding prime.  
	 
Or perhaps we should change the name of the array to something like `primeMultiples` and drop the comment altogether.

Moving on to the next comment:

	            // Each iteration of this loop tests the candidate against one
	            // potential prime factor. Skip the first factor (2) since we
	            // only consider odd candidates.
							
That doesn't  make a lot of sense.  The code it's talking about is:

	            for (int i = 1; i <= lastMultiple; i++) {
	                while (multiples[i] < candidate) {

The `multiples` array, as we have now learned, is an array of *multiples* of prime numbers.  This loop is not testing the candidate against prime *factors*, it's testing it against the current prime multiples.

Fortunately for me the third of fourth time I read this comment I realized that you really meant to use the word "multiples".  But the only way for me to know that was to understand the algorithm.  And when I understand the algorithm, why do I need the comment?

The last comment is:

	            // Start with the prime's square here, rather than 3x the prime.
	            // This saves time and is safe because all of the intervening
	            // multiples will be detected by smaller prime numbers. As an
	            // example, consider the prime 7: the value in multiples will
	            // start at 49; 21 will be ruled out as a multiple of 3, and
	            // 35 will be ruled out as a multiple of 5, so 49 is the first
	            // multiple that won't be ruled out by a smaller prime.
							
The first few times I read this it made no sense to me at all.  It was just a jumble of numbers.  So I ignored it.

As I started at the ceiling, and closed my eyes to visualize, I finally realized that the `multiples` array was the equivalent of the array of booleans we use in the *Sieve of Eratosthenes* -- but with a really interesting twist.  If you were to do the sieve on a whiteboard, you _could_ erase every number less than the candidate, and only cross out the numbers that were the next multiples of all the previous primes.

That explanation makes perfect sense to me -- now, but I'd be willing to be that those who are reading it are puzzling over it.  The idea is just hard to explain.

OK, so that left me with one final question.  What the deuce was the reason behind:

	multiples[primesFound] = candidate*candidate;

Why the square?  That makes no sense.  So I changed it to:

	multiples[primesFound] = candidate;

And it worked just fine.  So this must just be an optimization.  That whole long comment is about an optimization.  OK, so why?  Why can you start the multiple for a prime at prime^2?  

That's not an easy question to answer.  Eventually, after a long bike ride, I realized that the prime multiples of 2 will at one point contain 2*3 and then 2*5.  So the `multiples` array will at some point contain multiples of primes *larger* than the prime they represent.  !!!  And then it all made sense.

Then I could read your comment and see what you were saying.

###A Tale of Two Programmers
The bottom line here is that you and I both fell into the same trap.  I refactored that old algorithm 18 years ago, and I thought all those function and variable names would make my intent clear -- *because I understood that algorithm*.  

You wrote that code awhile back and decorated it with comments that you thought would explain your intent -- *because you understood that algorithm*.  

But my names didn't help me 18 years later.  They didn't help you, or your students either.  And your comments didn't help me.  

We were inside the box trying to communicate to those who stood outside and could not see what we saw.  

## Bob's Rewrite of PrimeGenerator2

**UB:**

When I saw your solution, and after I gained a good understanding of it.  I refactored it just a bit.  I loaded it into my IDE, wrote some simple tests, and extracted a few simple methods.  

I also got rid of that *awful* labeled `continue` statement.  And I added 3 to the primes list so that I could mark the first element as *irrelevant* and give it a value of -1.  (I think I was still reeling from the even/odd confusion.)

I like this because the implementation of the `generateFirstNPrimes` function describes the moving parts in a way that hints at what is going on.  It's easy to read that implementation and get a glimpse of the mechanism.  I'm not at all sure that the comment helps.

I think it is just the reality of this algorithm that the effort required to properly explain it, and the effort required for anyone else to read and understand that explanation is roughly equivalent to the effort needed to read the code and go on a bike ride.

```
package literatePrimes;

public class PrimeGenerator3 {
    private static int[] primes;
    private static int[] primeMultiples;
    private static int lastRelevantMultiple;
    private static int primesFound;
    private static int candidate;

    // Lovely little algorithm that finds primes by predicting
    // the next composite number and skipping over it. That prediction
    // consists of a set of prime multiples that are continuously
    // increased to keep pace with the candidate.

    public static int[] generateFirstNPrimes(int n) {
        initializeTheGenerator(n);

        for (candidate = 5; primesFound < n; candidate += 2) {
            increaseEachPrimeMultipleToOrBeyondCandidate();
            if (candidateIsNotOneOfThePrimeMultiples()) {
                registerTheCandiateAsPrime();
            }
        }
        return primes;
    }

    private static void initializeTheGenerator(int n) {
        primes = new int[n];
        primeMultiples = new int[n];
        lastRelevantMultiple = 1;

        // prime the pump. (Sorry, couldn't resist.)
        primesFound = 2;
        primes[0] = 2;
        primes[1] = 3;

        primeMultiples[0] = -1;// irrelevant
        primeMultiples[1] = 9;
    }

    private static void increaseEachPrimeMultipleToOrBeyondCandidate() {
        if (candidate >= primeMultiples[lastRelevantMultiple])
            lastRelevantMultiple++;

        for (int i = 1; i <= lastRelevantMultiple; i++)
            while (primeMultiples[i] < candidate)
                primeMultiples[i] += 2 * primes[i];
    }

    private static boolean candidateIsNotOneOfThePrimeMultiples() {
        for (int i = 1; i <= lastRelevantMultiple; i++)
            if (primeMultiples[i] == candidate)
                return false;
        return true;
    }

    private static void registerTheCandiateAsPrime() {
        primes[primesFound] = candidate;
        primeMultiples[primesFound] = candidate * candidate;
        primesFound++;
    }
}
```
