## Introductions

**JOHN:**

Hi (Uncle) Bob! You and I have each written books on software design.
We agree on some things, but there are some pretty big differences of
opinion between my recent book *A Philosophy of Software Design*
(herafter "APOSD") and your classic book *Clean Code*. Thanks for
agreeing to discuss those differences here.

**UB:**

My pleasure John.  Before we begin let me say that I've carefully read through your book and I found it very enjoyable, and full of valuable insights.  There are some things I disagree with you on, such as TDD, and Abstraction-First incrementalism, but overall I enjoyed it a lot.

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

I agree that dividing up code into relatively small units ("modular design")
is one of the most important ways to reduce the amount of information a
programmer has to keep in their mind at once. The idea, of course, is to take a
complex chunk of functionality and encapsulate it in a separate method
with a simple interface. Developers can then harness the methodality
of the method (or read code that invokes the method) without learning
the details of how the method is implemented; they only need to learn its
interface. The best methods are those that provide a lot of functionality
but have a very simple interface: they replace a large cognitive load
(reading the detailed implementation) with a much smaller
cognitive load (learning the interface). I call these methods "deep".

However, like most ideas in software design, decomposition can be taken too far.
As methods get smaller and smaller there is less and less
benefit to further subdivision.
The amount of functionality hidden behind each interface
drops, while the interfaces often become more complex.
I call these interfaces "shallow": they don't help much in terms of
reducing what the programmer needs to know. Eventually, the point is
reached where someone using the method needs
to understand every aspect of its implementation. Such methods
are usually pointless.

Another problem with decomposing too far is that it tends to
result in *entanglement*. Two methods
are entangled (or "conjoined" in APOSD terminology) if, in order to
understand how one of them works internally, you also need to read the
code of the other. If you've ever found yourself flipping back and forth
between the implementations of two methods as you read code, that's a
red flag that the methods might be entangled. Entangled methods
are hard to read because the information you need to have in your head
at once isn't all in the same place. Entangled methods can usually
be improved by combining them so that all the code is in one place.

The advice in *Clean Code* on method length is so extreme that it encourages
programmers to create teeny-tiny methods that suffer from both shallow
interfaces and entanglement.  Setting arbitrary numerical limits such
as 2-4 lines in a method and a single line in the body of an
`if` or `while` statement exacerbates this problem.

**UB:**

While I do strongly recommend very short functions, I don't think it's fair to say that the book sets arbitrary numerical limits. The 2-4 line functions that you referred to on page 34 were part of the _Sparkle_ applet that Kent Beck and I wrote together in 1999 as an exercise for learning TDD. I thought it was remarkable that most of the functions in that applet were 2-4 lines long because it was Swing program; and Swing programs tend to have very long methods.  

As for setting limits, on page 13 I make clear that although the recommendations in the book have worked well for me and the other authors, they might not work for everyone.  I claimed no final authority, nor even any absolute "rightness". They are offered for consideration.

**JOHN:**

I think these problems will be easiest to understand if we look at
specific code examples. But before we do that, let me ask you, Bob:
do you believe that it's possible for code to be over-decomposed, or
is smaller always better? And, if you believe that over-decomposition
is possible, how do you recognize when it has occurred?

**UB:**

It is certainly possible to over-decompose code.  Here's an example:

	void doSomething() {doTheThing()} // over-decomposed.

In general I like your narrow/deep, wide/shallow dichotomy.  However, I don't think extracting small methods inevitably drives a module towards the wide/shallow end.  This is because the methods I extract are private.  The root method from which they were extracted remains public, and is part of the interface of the module.  The narrow/deep nature of the module is preserved.

**JOHN:**

It sounds like you are saying that only a class's external interfaces need
to be deep, and it's OK for internal interfaces to be shallow and complex.
I disagree. I think that *all* interfaces should be deep, including
lower-level interfaces for methods within a class and higher-level interfaces
for subsystems, micro-services, etc. Clean interfaces provide benefits
everywhere.  

**UB:**
No, I'm saying something different.  Every function should have a narrow interface and an implementation that is deeper than the interface. The interface should be the abstraction that the function implements.

To me, depth does not mean lots of lines; rather it means deeper in the details that the interface is trying to hide. 

My fear with longish functions is that they often contain extractable elements. The interface between two extractable elements of code is implicit and anonymous.  An anonymous implicit interface is wide because the reader must deduce it from the entire context of the module.  Extracting those elements creates named and narrow interfaces without sacrificing any depth.

The practice of extracting elements and giving them named narrow interfaces creates a hierachy of named functions starting at the public interface of the module and descending, one step at a time, into the detailed implementation of that module.

This allows the reader to understand the flow of the module without having to read all the way down to the details.  This is very much like reading the first few paragraphs of a news article and then stopping because you got the gist of what the article was saying.

I call this kind of structure: _Polite_.

The strategy that I use for deciding how far to take extraction is the old rule that a method should do "*One Thing*".  If I can *meaningfully* extract one method from another, then the original method did more than one thing.  "Meaningfully" means that the extracted functionality can be given a descriptive name; and that it does less than the original method.

**JOHN:**

Unfortunately the One Thing approach will lead to over-decompositon:

 1. The term "one thing" is vague and easy to abuse. For example, if a method has two lines of code, isn't it doing two things?

 2. You haven't provided any useful guardrails to prevent over-decomposition. The example you gave is too extreme to be useful, and the "can it be named" qualification doesn't help: anything can be named.

 3. The One Thing approach is simply wrong in many cases. If two things are closely related, it might well make sense to implement them in a single method. For example, any thread-safe method will first have to acquire a lock, then carry out its function. These are two "things", but they belong in the same method.

**UB:**

I don't consider ease of abuse to be a concern. `If` statements are easy to abuse.  `Switch` statements are easy to abuse.  Assignment statements are easy to abuse.  The fact that something is easy to abuse does not mean that it should be avoided or suppressed.  It simply means people should take appropriate care. There will always be this thing called: _judgement_. 

So when faced with this snippet of code in a larger method:

	...
	amountOwed=0;
	totalPoints=0;
	...

It would be poor judgement to extract them as follows, because the extraction is not meaningful.  The implementation is not more deeply detailed than the interface.

	void clearAmountOwed() {
	  amountOwed=0;
	}

	void clearTotalPoints() {
	  totalPoints=0;
	}

However it may be good judgement to extract them as follows because the interface is abstract, and the implemention has deeper detail.

	void clearTotals() {
		amountOwed=0;
		totalPoints=0;
	}

The latter has a nice descriptive name that is abstract enough to be meaningful without being redundant.  And the two lines together are strongly related so as to qualify for doing _one thing_: initialization.

**JOHN:**

Without seeing more context, I'm skeptical that the `clearTotals`
method makes sense. If the meaning of those two lines isn't already clear
from context, why not just add a short comment before them, rather
than creating a separate method (which, by the way, is shallow)?
This example seems pretty trivial to me: is it really worth a lot
of thought over whether to pull 2 lines of code into a separate method?

**UB:**

Actually, that's an example from Martin Fowler's _Refactoring_.  The original code from that example looks like this:

	public String statement() {
	  double totalAmount = 0;
	  int frequentRenterPoints = 0;
	  Enumeration rentals = this.rentals.elements();
	  String result = "Rental Record for " + getName() + "\n";

	  while (rentals.hasMoreElements()) {
	    double thisAmount = 0;
	    Rental each = (Rental) rentals.nextElement();

	    // determines the amount for each line
	    switch (each.getMovie().getPriceCode()) {
	      case Movie.REGULAR:
	        thisAmount += 2;
	        if (each.getDaysRented() > 2)
	          thisAmount += (each.getDaysRented() - 2) * 1.5;
	        break;
	      case Movie.NEW_RELEASE:
	        thisAmount += each.getDaysRented() * 3;
	        break;
	      case Movie.CHILDRENS:
	        thisAmount += 1.5;
	        if (each.getDaysRented() > 3)
	          thisAmount += (each.getDaysRented() - 3) * 1.5;
	        break;
	    }

	    frequentRenterPoints++;

	    if (each.getMovie().getPriceCode() == Movie.NEW_RELEASE
	      && each.getDaysRented() > 1)
	      frequentRenterPoints++;

	    result += "\t" + each.getMovie().getTitle() + "\t"
	      + String.valueOf(thisAmount) + "\n";
	    totalAmount += thisAmount;
	  }

	  result += "You owed " + String.valueOf(totalAmount) + "\n";
	  result += "You earned " + String.valueOf(frequentRenterPoints) + " frequent renter points\n";

	  return result;
	}
	
And after a good refactoring looks like this:

	public String makeStatement() {
	  clearTotals();
	  return makeHeader() + makeRentalDetails() + makeFooter();
	}

	private void clearTotals() {
	  totalAmount = 0;
	  frequentRenterPoints = 0;
	}

	private String makeHeader() {
	  return "Rental Record for " + getCustomerName() + "\n";
	}

	private String makeRentalDetails() {
	  String rentalDetails = "";
	  for (Rental rental : rentals)
	    rentalDetails += makeRentalDetail(rental);

	  return rentalDetails;
	}

	private String makeRentalDetail(Rental rental) {
	  double amount = rental.determineAmount();
	  frequentRenterPoints += rental.determinePoints();
	  totalAmount += amount;
	  return formatRentalDetail(rental, amount);
	}

	private String formatRentalDetail(Rental rental, double amount) {
	  return "\t" + rental.getTitle() + "\t" + amount + "\n";
	}

	private String makeFooter() {
	  return "You owed " + totalAmount + "\n" +
	    "You earned " + frequentRenterPoints + " frequent renter points\n";
	}
	
BTW, that's _my_ refactoring, not Martin's.  Look at that `makeStatement` function.  It is about as clear and concise is you can get.  It communicates the high level policy of the module.  It would be a shame, IMHO, to pollute it with a couple of detailed assignment statements.  

Notice also the hierarchy of functions.  Each function calls those that are just below it.  And as you read downward you also descend into deeper detail. The reader can stop reading as soon as they get the gist of the module.  Again, that's _polite_.

**JOHN:**

I agree that it isn't possible to provide precise recipes for software
design, and judgment will inevitably be involved. But judgment depends
on principles and guidance. The
*Clean Code* arguments about decomposition, including the One Thing
principle, are one-sided. They give strong, concrete, quantitative
advice about when to chop things up, with virtually no guidance for
how to tell you've gone too far. All I could find is a 2-sentence
example on page 36 about Listing 3-3 (which is pretty trivial),
buried in the middle of exhortations
to "chop, chop, chop". Is there something more comprehensive that
I missed?

**UB:**

You make a good point that I don't talk much, in the book, about the how to make the judgement call.  Back in 2008 my concern was breaking the habit of the very large functions that were common in those early days of the web.  I'll have to consider being a bit more balanced in the 2d ed.  

Still, if I must err, I'd rather err on the side of extraction.  Extractions can always be inlined if they we judge them to be too decomposed. 

**JOHN:**

One of the reasons I use the deep/shallow characterization is that it
captures both sides of the tradeoff; it will tell you when a decomposition
is good and also when decomposition makes things worse.

I think it will be easier to clarify our differences if we consider
a specific code example. Let's look at the `PrimeGenerator` class from
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

Of course this code was never meant for production because it is a pedagogical exercise.  It appears in a chapter named *Classes*.  I used it as a demonstration for how to partition a large and messy legacy function into a set of smaller classes and methods.

**JOHN:**

It sounds like we may disagree not just on software design, but also
on teaching pedagogy. In my experience, when students see example code they
assume it is "good" in every way, and they will emulate it (in every way).

**UB:**

That's a fair point.  However, there are plenty of other examples in the book that go into substantially more depth.  For example, there is another implementation of a prime number generator on page 73 (Listing 4-8) that I think you'd find more acceptable.  So I'm not overly concerned that this one will lead anyone astray.

**JOHN:**

If we want to
teach people how to write production code, then our examples should appear
as close as possible to the way we think production code should appear.

**UB:**

Again, fair point.

**JOHN:**

In any case, can you clarify (a) in what ways you would like readers to
emulate this example in production and (b) in what ways they should *not*
emulate the example (if there are any)? 

**UB:**

The lesson of the chapter is that a large function can contain many different sections of code that are better extracted into independent classes.  So I would like my readers to attempt such extractions when they see large functions; and they should *not* allow such unextracted functions to persist.

BTW, that is also the lesson of Martin Fowler's video store example above.

In the chapter I extracted three classes from that function: `PrimePrinter`, `RowColumnPagePrinter` and `PrimeGenerator`.

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

**JOHN**:

I don't think you have represented Knuth's work fairly. 

**UB:**

In fact, on page 141 I say: "To be fair to Knuth, this is not the program as he wrote it but rather as it was output by his WEB tool.  I'm using it because it makes a great starting place for breaking up a big function into many smaller functions and classes."

**JOHN:**

If I understand
the Literate Programming paper, the idea was to intermix extensive documentation
with the program code in order to make the code easier to understand.
I don't necessarily agree with Knuth's approach, but
you seem to have extracted the code while dropping all
of the documentation that Knuth intended to accompany it. It's no surprise
that the extracted code is hard to read.

**UB**:

My goal was not to describe how to generate prime numbers.  I wanted my readers to see how large methods, that violate the Single Responsibility Principle, can be broken down into a few smaller well-named classes containing a few smaller well-named methods.

I think it's somewhat better than where it started.

It was _not_ my intent, in this chapter, to teach my readers how to write the most optimal prime number generator.  That was the last thing on my mind, and the last thing I wanted on theirs.

**JOHN:**

Agreed; for this discussion I assume we will take the algorithm as a given,
and focus on the cleanest possible way to implement that algorithm.

I think there are many design problems with `PrimeGenerator`, but for now I'll
focus on method length (I'll come back to this code later when we
discuss comments). The code is chopped up so much (8 teeny-tiny methods)
that it's difficult to read. For starters, consider the
`isNotMultipleOfAnyPreviousPrimeFactor` method. This method invokes
`isMultipleOfNthPrimeFactor`, which invokes
`smallestOddNthMultipleNotLessThanCandidate`. These methods are shallow
and entangled:
in order to understand
`isNot...` you have to read the other two
methods and load all of that code into your mind at once. For example,
`isNot...` has side effects (it modifies `multiplesOfPrimeFactors`), but
you can't see that unless you read all three methods.

**UB:**

I agree.  Though I would not have agreed eighteen years ago when I was in the throes of refactoring this.  At the time the names and structure made perfect sense to me.  They make sense to me now, too -- but that's because I once again understand the algorithm.  When I returned to the algorithm a few days ago, I also struggled with the names and structure.  Once you understand the algorithm the names make perfect sense, and that understanding breaks the entaglement.

**JOHN:**

I think those names are highly problematic; we'll talk about that a bit later,
when discussing comments.

**UB**:

So, a good critique of those names is that they depend, to some extent, upon gaining some understanding of the algorithm.

**JOHN:**

Going back to my introductory remarks about complexity, splitting up
`isNot...` into three methods doesn't reduce the amount of information
you have to have. It just splits it up and spreads it out, so it isn't as
obvious that you need to read all three methods together. And, it's harder
to see the overall structure of the code because it's split up: readers have
to flip back and forth between the methods, effectively reconstructing a
monolithic version in their minds. Because the pieces are all related,
this code will be easiest to understand if it's all together in one place.

**UB:**

I disagree.  Here is `isNotMultipleOfAnyPreviousPrimeFactor`.

	  private static boolean
	  isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
	    for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
	      if (isMultipleOfNthPrimeFactor(candidate, n))
	        return false;
	    }
	    return true;
	  }

If you trust the `isMultipleOfNthPrimeFactor` method, then this method stands alone quite nicely.  I mean we loop through all n previous primes and see if the candidate is a multiple.  That's pretty straight forward.

Now it would be fair to ask the question how we determine whether the candidate is a multiple, and in that case you'd want to inspect the `isMultiple...` method.

**JOHN:**

I agree that it would be easy for someone reading `isNot...` to think they
understood it completely without needing to read `isMultiple...` or
`smallestOdd...`. Unfortunately they would be mistaken. For example, they
would not realize that `isNot...` has side effects and that its
`candidate` argument must be monotonically non-decreasing from invocation
to invocation. To understand these important behaviors, you have to
read both `isMultiple...` and `smallestOdd...`. The current decomposition
hides this important information from the reader by pulling it out of
`isNot...`.

If there is one thing more likely to result in bugs than not understanding code,
it's thinking you understand it when you don't.

**UB:**

That's a valid concern.  However, it is tempered by the fact that the functions are presented in the order they are called.  Thus we can expect that the reader has already seen the main loop and understands that candidate increases by two each iteration.

The side effect buried down in `smallestOddNth...` is a bit more problematic. Now that you've pointed it out I don't like it much.  Still, that side effect should not confound the basic understanding of `isNot...`.   

In general, if you trust the names of the methods being called then understanding the caller does not require understanding the callee.  For example:

	for (Employee e : employees)
	  if (e.shouldPayToday())
		  e.pay();

This would not be made more understandable if we replaced those two method calls with the their implementations.  Such a replacement would simply obscure the intent.

**JOHN:**

This example works because the called methods are relatively independent of
the parent. Unfortunately that is not the case for `isNot...`.

In fact, `isNot...` is not only entangled with the methods it calls, it's also
entangled with its callers. `isNot...` only works if it is invoked in
a loop where `candidate` increases monotonically. To convince yourself
that it works, you have to find the code that invokes `isNot...` and
make sure that `candidate` never decreases from one call to the next.
Separating `isNot...` from the loop that invokes it makes it harder
for readers to convince themselves that it works.

**UB:**

Which, as I said before, is why the methods are ordered the way they are.  I expect that by the time you get to `isNot...` you've already read `checkOddNumbersForSubsequentPrimes` and know that `candidate` increases by twos.

**JOHN:**

I'm glad you brought this up, because it's another area where I
disagree with *Clean Code*. If methods are entangled, there is no
clever ordering of the method definitions that will fix the problem.

In this particular situation two other methods intervene between the
loop in `checkOdd...` and `isNot...`, so readers will have forgotten
the loop context before they get to `isNot...`. Furthermore, the actual
code that creates a dependency on the loop isn't in `isNot...`: it's in
`smallestOdd...`, which is even farther away from `checkOdd...`.

**UB:**

I sincerely doubt anyone is going to forget that `candidate` is being increased by twos.  It's a pretty obvious way to avoid waste.

**JOHN:**

In my opening remarks I talked about how it's important to reduce the
amount of information people have to keep in their minds at once.
In this situation, readers have to remember that loop while they read
four intervening methods that are mostly unrelated. But it's even worse than
that. There is no indication which parts of `checkOdd...` will be important
later on, so the only safe approach is to remember *everything*, from *every*
method, until you have encountered every other method that could possibly
descend from it. This can't possibly work.

If two pieces of code are tightly related, the solution is to bring
them together. Separating the pieces, even in adjacent methods,
makes the code harder to understand.

To me, all of the methods in `PrimeGenerator` are entangled: in order to
understand the class I had to load all of them into my mind
at once. I was constantly flipping back and forth between the methods
as I read the code. This is a red flag indicating
that the code has been over-decomposed.

Bob, can you help me understand why you divided the code into such
tiny methods?
Is there some benefit to having so many methods that I have missed?

**UB:**

I think you and I are just going to disagree on this.  In general I believe in the principle of small well-named methods and the separation of concerns. Generally speaking if you can break a large method into several well-named smaller methods with different concerns, and by doing so expose their interfaces, and the high level functional decomposition, then that's a good thing.

* Looping over the odd numbers is one concern.
* Determining primality is another.
* Marking off the multiples of primes is yet another.

It seems to me that separating and naming those concerns helps to expose the way the algorithm works.

In your solution, below, you break the algorithm up in a similar way.  However, instead of separating the concerns into functions, you separate them into sections with comments above them.

You mentioned that in my solution readers will have to keep the loop context in mind while reading the other functions.  I suggest that in your solution, readers will have to keep the loop context in mind while reading your explanatory comments.  They may have to "flip back and forth" between the sections in order to establish their understanding.

Now perhaps you are concerned that in my solution the "flipping" is a longer distance (in lines) than in yours.  I'm not sure that's a significant point since they all fit on the same screen (at least they do on my screen) and the landmarks are pretty obvious.

**JOHN:**

I agree that separation of concerns is a good thing, but only if the
concerns are really independent. The problem with `PrimeGenerator` is
that it separates things that are not independent.

If these two approaches still seem to you like they create equal
cognitive load, then we will have to agree to disagree. I invite
readers to decide for themselves.

I agree that names are important, but I worry that you focus too narrowly
on names without considering other factors that are even more important.
Breaking a large method up only makes
sense if the smaller methods have clean interfaces and are relatively
independent. A method's name almost never captures the full complexity of its
interface (we will discuss this later) and it doesn't say anything about
independence vs. entanglement.

**UB:**

We will have to disagree on that.  I think that if a method is terse enough, then the name should be sufficient to describe it in most cases.  

In this case I think I improved the structure and naming of the original quite a bit.  Perhaps I did not improve it enough but I think it serves to present the strategy for breaking up large methods into smaller classes and methods.  

**JOHN:** >>reword

I'm comfortable with your goal. However (and I'm sorry to be blunt) `PrimeGenerator` is a
really bad decomposition. I hope readers will not emulate it. Furthermore, I
don't think it's bad because of an accident; it ended up way over-decomposed
and hard to read because you followed the advice of *Clean Code*.

It sounds like we'll have to disagree here. Readers can decide for
themselves: when deciding how to decompose code, is the algorithm
relevant? And, when breaking up functions is it OK to produce a decomposition
that makes it hard to understand how the algorithm works?

**UB:**

I don't mind people being blunt.  As for your contention that the `PrimeGenerator` was over decomposed, I decomposed it into 8 small functions.  You decomposed your solution (below) into seven commented sections.  So I'm not sure your argument holds a lot of water.  ;-)

**JOHN:**

We already discussed this (it isn't just how many pieces there are, but
how they are arranged); 

**UB:**

I like the way I arranged it because it follows the principle of functional decomposition:  Higher level policies should be above lower level details.  

## Comments

**JOHN:**

Let's move on to the second area of disagreement: comments. In my opinion,
the *Clean Code* approach to commenting results in code with
inadequate documentation, which increases the cost of software development.
I'm sure you disagree, so let's discuss.

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
safest thing is to write no comments."

**UB:**

That chapter begins with these words:
>*Nothing can be quite so helpful as a well placed comment.*

It goes on to say that comments are a *necessary* evil.

The only way a reader could infer that they should write no comments is if they hadn't actually read the chapter.  The chapter walks through a series of comments, some bad, some good.

**JOHN:**

*Clean Code* focuses a lot more on the "evil" aspects of comments than the
"necessary" aspects. The sentence you quoted above is followed by two
sentences criticizing comments. Chapter 4 spends 4 pages talking about good
comments, followed by 15 pages talking about bad comments. There are snubs
like "the only truly good comment is the comment you found a way
not to write". And "Comments are always failures" is so catchy
that it's the one thing readers are most likely to remember from the
chapter.

**UB:**

The difference in page count is because there are just a few ways to write good comments, and so many more ways to write bad ones.

You and I likely both survived through a time when comments were absolutely necessary.  In the '70s and '80s I was an assembly language programmer.  I also wrote a bit of FORTRAN. Programs in those languages that had no comments were impenetrable. 

As a result it became conventional wisdom to write comments by default.  And, indeed, computer science students were taught to write comments uncritically.  Comments became _pure good_.

In the book I decided to fight that mindset.  Comments can be _really bad_ as well as good.


**JOHN:**

I don't agree that comments are evil, and I don't agree that comments are
any less necessary today than they were 40 years ago.

Comments are not the problem, they are the solution.
The problem is that there is a lot of important
information that simply cannot be expressed in code. When a programmer
writes comments, they have not failed to express themselves; they have
succeeded in providing important information that makes code easier to
understand.

**UB:**

It's very true that there is important information that is not, or cannot be, expresssed in code.  That's a failure.  A failure of our languages, or of our ability to use them to express ourselves.  In every case a comment is a failure of our ability to use our languages to express our intent.

And we fail at that very frequently, and so comments are a necessary evil -- or, if you prefer, _an unfortunate necessity_.  If we had the perfect programming language (TM) we would never write another comment.

**JOHN:**

I don't agree that a perfect programming language would
eliminate the need for comments. Comments and code serve very different
purposes, so it's not obvious to me that we should use the same
language for both. In my experience, English works quite well
as a language for comments.
Why do you feel that information about a program should
be expressed entirely in code, rather than using a combination of code
and English?

**UB:**

I bemoan the fact that we must sometimes use a human language instead of a programming language.  Human languages are imprecise and full of ambiguities. Using a human language to describe something as precise as a program is very hard, and fraught with many opportunities for error and inadvertent misinformation.

**JOHN:**

I agree that English isn't as precise as code, but it can be used in
in fairly precise ways and comments typically don't need the same
degree of precision as code.
Comments often contain qualitative information such
as *why* something is being done, or the overall idea of something.
English works better for these than code; overall, it is a more
expressive language.

Are you concerned that comments will be incorrect or
misleading and that this will slow down software development?
I often hear people complain about stale comments, but
I have not found them be a significant problem
over my career. Incorrect comments do happen, but I don't encounter them
very often and when I do, they rarely cost me much time. In contrast, I waste
*enormous* amounts of time because of inadequate documentation; it's not
unusual for me to spend 50-80% of my development time wading through
code to figure out things that would be obvious if the code was properly
commented.

**UB:**

You and I have had very different experiences.  ;-)

**JOHN:**

I invite everyone reading this article to ask yourself the following questions:

* How much does your software development speed suffer because of
  incorrect comments?
* How much does your software development speed suffer because of
  missing comments?

For me the cost of missing comments is easily 10-100x the cost of incorrect
comments. That is why I cringe when I see things in *Clean Code* that
discourage people from writing comments.

Let's consider the `PrimeGenerator` class. There is not a single comment
in this code other than the one I added at the top; do you
think this is appropriate?

**UB:**

I think it was appropriate for the purpose for which I wrote it. It was created to make the point that large methods can be broken down into smaller classes containing smaller methods. Adding lots of explanatory comments would have detracted from that point.

I agree, however, that the commenting style in Listing 4-8 is generally more appropriate.

**JOHN:**

I disagree that adding comments would have distracted from your point,
but let's not argue that. Instead, let's discuss what comments this code
*should* have if it were used in production. I will make some suggestions
and you can agree or disagree.

For starters, let's discuss your use of megasyllabic names like
`isLeastRelevantMultipleOfLargerPrimeFactor`.  My understanding is that
you advocate using names like this instead of using shorter names
augmented with descriptive comments: you're effectively moving the
comments into code. To me, this approach is problematic:

* Long names are awkward. Developers effectively have to retype
  the documentation for a method every time they invoke it, and the long
  names waste horizontal space and trigger line wraps in the code. The names are
  also awkward to read: my mind wants to parse every syllable every time
  I read it, which slows me down. Notice that both you and I resorted to
  abbreviating names in this discussion: that's an indication that
  the long names are awkward and unhepful.
* The names are hard to parse and don't convey information as effectively
  as a comment.
  When students read `PrimeGenerator` one of the first things they
  complain about is the long names (students can't make sense of them).
  For example, the name above is
  vague and cryptic: what does "least relevant" mean, and what is a
  "larger prime factor"? Even with a complete understanding of the code in
  the method, it's hard for me to make sense of the name.  If this name
  is going to eliminate the need for a comment, it needs to be even longer.

In my opinion, the traditional approach of using shorter names with
descriptive comments is more convenient and conveys the required information
more effectively. What advantage is there in the approach you advocate?

**UB:**

"_Megasyllabic_": Great word! 

I like my method names to be sentence fragments that fit nicely with keywords and assignment statements.  It makes the code a bit more natural to read.

	if (isTooHot)
	  cooler.turnOn();

I also follow a simple rule about the length of names.  The larger the scope of a function, the shorter its name should be.  The private functions I extracted in this case live in very small scopes, and so have longish names.  Functions like this are typically called from only one place, so there is no burden on the programmer to remember a long name for another call.

As for being hard to parse, that's a matter of practice.  Code is full things that take practice to get used to.

As for the meaning of "leastRelevant", that's a much larger problem that you and I will encounter shortly.  It has to do with the intimacy that the author has with the solution, and the reader's lack of that intimacy.

**JOHN:**

Now that we've discussed the specific issue of comments vs. long method
names, let's talk about comments in general. There are two major reasons
why comments are needed. The first reason for comments is abstraction.
Simply put, without comments there is no way to have abstraction or modularity.

Abstraction is one of the most important components of good software design.
I define an abstraction as "a simplified way of thinking about something
that omits unimportant details." The most obvious example of an abstraction
is a method. It should be possible to use a method without reading its code.
The way we achieve this is by writing a header comment that describes
the method's *interface* (all the information someone needs in order
to invoke the method). If the method is well designed, the interface will be
much simpler than the code of the method (it omits implementation details),
so the comments reduce the amount of information people must have in
their heads.

**UB:**

Long ago, in a 1995 book, I defined abstraction as:
>*The amplification of the essential and the elimination of the irrelevant.*

I certainly agree that abstraction is of importance to good software design.  I also agree that well placed comments can enhance the ability of readers to understand the abstractions we are attempting to employ.  I disagree that comments are the _only_, or even the _best_, way to understand those abstractions.  But sometimes they are the only option.

But consider:

	addSongToLibrary(String title, String[] authors, int durationInSeconds);

This seems like a very nice abstraction to me, and I cannot imagine how a comment might improve it.

**JOHN:**

Our definitions of abstraction are very similar; that's good to see.
However, the `addSongToLibrary` declaration is not (yet) a good abstraction
because it omits information
that is essential. In order to use `addSongToLibrary`, developers
will need answers to the following questions:

* Is there any expected format for an author string, such as "LastName, FirstName"?
* Are the authors expected to be in alphabetical order? If not, is the order
  significant in some other way?
* What happens if there is already a song in the library with the given title
  but different authors? Is it replaced with the new one, or will the library
  keep multiple songs with the same title?
* How is the library stored (e.g. is it entirely in memory? saved on disk?)?
  If this information is documented somewhere else, such as the
  overall class documentation, then it need not be repeated here.

Sometimes the signature of a method (names and types of the method, its
arguments, and its return value) contains all the information
needed to use it, but this is pretty rare. Just skim through the documentation
for your favorite library package: in how many cases could you understand how
to use a method with only its signature?

**UB:**

Yes, there are times when the signature of a method is an incomplete abstraction and a comment is required.  There are other times when the signature tells you everything you want to know.  It seems to me that we should try to create more of the later kind and avoid the former where possible.  

**JOHN:**

Let's consider an example from `PrimeGenerator`: the `isMultipleOfNthPrimeFactor`
method. When someone reading the code encounters the call to `isMultiple...`
in `isNot...` they need to understand enough about how `isMultiple...` works
in order to see how it fits into the code of `isNot...`.
The method name does not fully document the interface, so if there
is no header comment then readers will have to read the code of `isMultiple`.
This will force readers to load more information into their
heads, which makes it harder to work in the code.

Here is my first attempt at a header comment for `isMultiple`:

```
    /**
     * Returns true if candidate is a multiple of primes[n], false otherwise.
     * May modify multiplesOfPrimeFactors[n].
     * @param candidate
     *      Number being tested for primality; must be at least as
     *      large as any value passed to this method in the past.
     * @param n
     *      Selects a prime number to test against; must be
     *      <= multiplesOfPrimeFactors.size().
     */
```

What do you think of this?

**UB:**

I think it's accurate.  I wouldn't delete it if I encountered it.  I don't think it should be a javadoc.

The first sentence is redundant with the name `isMultipleOfNthPrimeFactor` and so could be deleted.  The warning of the side effect is useful.

The name `candidate` is synonymous with "Number being tested for primality".

The constraints on the arugments could be written as executable preconditions following Bertrand Meyer's idea of _Design by Contract_ used in the Eiffel language.

	require
		candidate >= last.candidate and
		n <- multipleOfPrimeFactors.size()

In Eiffel these could be turned off during production.

Eiffel is an example of a language that diminishes the need for comments like that.

In the end, however, all those words are just going to have to sit in my brain until I understand why they are there.  I'm also going to have to worry if they are accurate.  So I'm going to have to read the code to understand and validate the comment.

**JOHN:**

I mentioned earlier that there are two general reasons why comments are
needed. So far we've been discussing the first reason (abstraction).
The second general reason for comments is for important information
that is not obvious from the code. The algorithm in `PrimeGenerator`
is very non-obvious, so quite a few comments are needed to help readers
understand what is going on and why. Most of the complexity arises because
the algorithm is designed to compute primes efficiently:

* The algorithm goes out of its way to avoid divisions, which were quite
  expensive  when Knuth wrote his original version (they aren't that expensive
  nowadays).

* The first multiple for each new prime number is computed by squaring the
  prime, rather than multiplying it by 3. This is mysterious: why is it safe
  to skip the intervening odd multiples? Furthermore, it might seem that this
  optimization only has a small impact on performance, but in fact it makes an
  *enormous* difference (orders of magnitude). Using the square has the
  side-effect that when
  testing a candidate, only primes up to the square root of the
  candidate are tested. If 3x were used as the initial multiple, primes
  within a factor of 3 of the candidate would be tested; that's a *lot*
  more tests.
  This implication of using the square is so non-obvious that I only realized
  it while preparing material for this discussion; it never occurred to me in
  the many times I have discussed the code with students.

Neither of these issues is obvious from the code; without
comments, readers are left to figure them out on their own. The students
in my class are generally unable to figure out either of them in the
30 minutes I give them, but comments would have
allowed them to understand in a few minutes. Going back to my
introductory remarks, this is an example where information is important,
so it needs to be made available.

Do you agree that there should be comments to explain each of these
two issues?

**UB:**

I agree that the algorithm is subtle.  Setting the first prime multiple as the square of the prime was deeply mysterious at first.  I had to go on an hour long bike ride to understand it.

Would a comment help?  Perhaps.  However, my guess is that no one who has been reading our conversation has been helped by it, because you and I are now too intimate with the solution.  You and I can talk about that solution using words that fit into that intimacy; but our readers likely do not yet enjoy that fit.

One solution is to paint a picture -- being worth a thousand words.  Here's my attempt.

	                                                                X
	                                                    1111111111111111111111111
	       1111122222333334444455555666667777788888999990000011111222223333344444
	   35791357913579135791357913579135791357913579135791357913579135791357913579
	   !!! !! !! !  !!  ! !! !  !  !!  ! !!  ! !  !   ! !! !! !      
	 3 |||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-
	 5 |||||||||||-||||-||||-||||-||||-||||-||||-||||-||||-||||-||||-
	 7 |||||||||||||||||||||||-||||||-||||||-||||||-||||||-||||||-||||||-
	11 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||-||||||||||-
	13 ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
	...
	113||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
	
I expect that our readers will have to stare at this for some time, and also look at the code.  But then there will be a _click_ in their brains and they'll say "Ohhh!  Yes!  I see it now!"


## John's Rewrite of PrimeGenerator

**JOHN:**

I mentioned that I ask the students in my software design class to rewrite
`PrimeGenerator` to fix all of its design problems. Here is my rewrite:

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

* There is only one method. I didn't subdivide it because I felt the method already divides naturally into pieces that are distinct and understandable. It didn't seem to me that pulling out methods would improve readability significantly. When students rewrite the code, they typically have 2 or 3 methods, and those are usually OK too.
* There are a *lot* of comments. It's extremely rare for me to write code with this density of comments. Most methods I write have no comments in the body, just a header comment describing the interface. But this code is subtle and tricky, so it needs a lot of comments to make the subtleties clear to readers. Even with all this additional explanatory material my version is a bit shorter than the original (65 lines vs. 70).

**UB:**

I presume this is a complete rewrite.  My guess is that you worked to understand the algorithm from Clean Code and then wrote this from scratch.  If that's so, then fair enough.

In _Clean Code_ I *refactored* Knuth's algorithm in order to give it a little structure.  That's not the same as a complete rewrite.

Having said that, your version is much better than either Knuth's or mine.  I could not have used it in the chapter I was writing *because* it is still in a single method, and I needed to show a more granular partioning.

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

I'm not opposed to Javadocs as a rule; but I write them only when absolutely necessary. I also have an aversion for descriptions and `@param` statements that are perfectly obvious from the method signature.

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

That doesn't make a lot of sense.  The code it's talking about is:

	            for (int i = 1; i <= lastMultiple; i++) {
	                while (multiples[i] < candidate) {

The `multiples` array, as we have now learned, is an array of *multiples* of prime numbers.  This loop is not testing the candidate against prime *factors*, it's testing it against the current prime _multiples_.

Fortunately for me the third of fourth time I read this comment I realized that you really meant to use the word "multiples".  But the only way for me to know that was to understand the algorithm.  And when I understand the algorithm, why do I need the comment?

That left me with one final question.  What the deuce was the reason behind:

	multiples[primesFound] = candidate*candidate;

Why the square?  That makes no sense.  So I changed it to:

	multiples[primesFound] = candidate;

And it worked just fine.  So this must be an optimization of some kind.  

Your comment to explain this is:

	            // Start with the prime's square here, rather than 3x the prime.
	            // This saves time and is safe because all of the intervening
	            // multiples will be detected by smaller prime numbers. As an
	            // example, consider the prime 7: the value in multiples will
	            // start at 49; 21 will be ruled out as a multiple of 3, and
	            // 35 will be ruled out as a multiple of 5, so 49 is the first
	            // multiple that won't be ruled out by a smaller prime.

The first few times I read this it made no sense to me at all.  It was just a jumble of numbers. 

I stared at the ceiling, and closed my eyes to visualize. I couldn't see it.  So I went on a long contemplative bike ride during which I realized that the prime multiples of 2 will at one point contain 2*3 and then 2*5.  So the `multiples` array will at some point contain multiples of primes *larger* than the prime they represent.  _And it clicked!_  

Suddenly it all made sense. I realized that the `multiples` array was the equivalent of the array of booleans we use in the *Sieve of Eratosthenes* -- but with a really interesting twist.  If you were to do the sieve on a whiteboard, you _could_ erase every number less than the candidate, and only cross out the numbers that were the next multiples of all the previous primes. 

That explanation makes perfect sense to me -- now, but I'd be willing to bet that those who are reading it are puzzling over it.  The idea is just hard to explain.

Finally I went back to your comment and could see what you were saying.

###A Tale of Two Programmers
The bottom line here is that you and I both fell into the same trap.  I refactored that old algorithm 18 years ago, and I thought all those method and variable names would make my intent clear -- *because I understood that algorithm*.

You wrote that code awhile back and decorated it with comments that you thought would explain your intent -- *because you understood that algorithm*.

But my names didn't help me 18 years later.  They didn't help you, or your students either.  And your comments didn't help me.

We were inside the box trying to communicate to those who stood outside and could not see what we saw.

## Bob's Rewrite of PrimeGenerator2

**UB:**

When I saw your solution, and after I gained a good understanding of it.  I refactored it just a bit.  I loaded it into my IDE, wrote some simple tests, and extracted a few simple methods.

I also got rid of that *awful* labeled `continue` statement.  And I added 3 to the primes list so that I could mark the first element as *irrelevant* and give it a value of -1.  (I think I was still reeling from the even/odd confusion.)

I like this because the implementation of the `generateFirstNPrimes` method describes the moving parts in a way that hints at what is going on.  It's easy to read that implementation and get a glimpse of the mechanism.  I'm not at all sure that the comment helps.

I think it is just the reality of this algorithm that the effort required to properly explain it, and the effort required for anyone else to read and understand that explanation is roughly equivalent to the effort needed to read the code and go on a bike ride.

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

## Test-Driven Development

**JOHN:**

Let's move on to our third area of disagreement, which is Test-Driven
Development. I am a huge fan of unit testing. I believe that unit tests are
an indispensable part of the software development process and pay for
themselves over and over. I think we agree on this.

However, I am not fan of Test-Driven Development (TDD), which dictates
that tests must be written before code and that code must be written
and tested in tiny increments. This approach has serious problems
without any compensating advantages that I have been able to identify.

*Clean Code* advocates for TDD, but I was not able to find any
explanation in *Clean Code* of *why* TDD is a good idea. Bob,
perhaps you can start the discussion off by explaining what benefits
accrue from writing tests before code?

**UB:**

Oh good!  Now I get to complain about your book.  ;-)

You briefly mention TDD on page 157 and ... well, to be kind, you get it pretty badly wrong. Allow me to quote:

>"Test-driven development is an approach to software development where programmers write unit tests before they write code.  When creating a new class, the develper first writes unit tests for the class, based on its expected behavior.  None of these tests pass, since there is no code for the class.  Then the developer works through the tests one at a time, writing enough code for that test to pass.  When all of the tests pass, the class is finished."

This is just wrong.  TDD is quite considerably different from what you describe.  I describe it using three laws.

 1. You are not allowed to write any production code until you have first written a unit test that fails because that code does not exist.

 2. You are not allowed to write more of a unit test than is sufficient to fail, and failing to compile is failing.

 3. You are not allowed to write more production code than is sufficient to make the currently failing test pass.

 **JOHN:**

 Oops! I plead "guilty as charged". I will fix this in the next revision of APOSD. That said,
 your description of TDD does not change my feelings about it.

 **UB:**

Following these three laws traps you into a loop that is, perhaps, 10 to 30 seconds long.  It's almost a line by line thing.

 * You write a line or two of a unit test, but it won't compile because the code it calls doesn't exist.

 * You write a line or two of production code that makes the test compile.

 * You write another line or two of the test, but it fails to compile because you haven't finished the production code.

 * You write another line or two of production code that makes the test compile.

 * You write another line or two of the test, and it compiles, but fails because the production code isn't working.

 * You write another line or two of the production code that makes the test pass.

This very quick cycle means that you are never more than a few seconds or minutes away from seeing _all_ your currently written tests pass.   And _that_ means you almost never need to debug anything.  After all, if you saw everything work a minute ago, and now it doesn't, it must be something you did within the last minute.

**JOHN:**

These claims seem pretty fantastical (and you've provided no supporting
evidence). You seem to be claiming that TDD makes it harder to create bugs;
I don't see why that would be true, and I will argue
just the opposite below. And, when a problem arises, it isn't necessarily
"something you did within the last minute". It could be a latent bug in
code you wrote months ago, which simply wasn't triggered until the latest
code was added. Adding a small amount of new code could
result in a long and painful debugging process. You seem to be assuming
that if code is added in tiny increments it isn't possible to create
big problems; I don't think any experienced programmer would find this
claim credible.

>>> I rewrote the text above, so you may want to revise your response below.

**UB:**

Asking for supporting evidence is a two-way street.  All through this document I could have been asking you to show evidence of your claims.  But that's not what this document is about.  This is about two very experienced practitioners describing recommendations based upon their experiences.

I have been practicing TDD for over 25 years.  So if I say something that sounds "fantastical" that's only because you and I have had very different experiences.  From my point of view, nothign I said was fantastical.

Or rather, it's not fantastical _now_.  It was 25 years ago when I first encountered this discipline.  Back then I thought it was a load of hooey.  But then I tried it -- and my mind was changed very quickly.  

But allow me to continue with the explanation.  The very intense cycle I just described is part of a larger cycle that we call RED-GREEN-REFACTOR.  First we make it fail, then we make it pass, then we clean it up and consider the design.

This follows the old maxim:

 * First make it work.
 * Then make it right.

**JOHN:**

I think this maxim is bad advice. This is what I call "tactical programming"
in APOSD, where the emphasis in development is on getting the next thing working,
rather than producing a good design. At each step it's easy to justify taking
a shortcut or making things a bit more complicated, because that's the
fastest way to get the next thing working. The result is a rapid accumulation
of complexity and technical debt.

Your suggestion that I (or anyone) might work in a perfectly straight
line without errors is silly, of
course, and I agree that finding bugs quickly is a good thing. But there's
one thing that's even better than finding bugs quickly, and that's
preventing bugs from occurring in the first place. That's what design
is for.

TDD discourages design, which leads to more complex code
with more bugs. For example, TDD discourages thinking ahead
("You are not allowed to write more production code than is sufficient
to make the currently failing test pass") but the whole idea of design
is to think ahead to avoid problems in the future.

Here is the kind of nightmare scenario I worry about with TDD:

* I write my first unit test and just enough production code to make
  it work. Within a couple of minutes, the test passes.
* Five minutes later, the next test is passing.
* I write the next unit test and start filling in its production code.
  However, the code I wrote for earlier tests doesn't exactly work for this
  new test. I could revise the earlier code to fix this, but the fastest
  way to make the new test pass is to make a couple of tweaks rather
  than a clean fix.
* Each test exposes more problems with the code structure that is evolving,
  but there's always a hack I can make to get the next test running.
  This seems consistent with TDD's instruction that I must write no more
  production code than is needed to make the next test pass.
* The code gets uglier and uglier, and the hacks get more and more complicated.
  Pretty soon, the hacks for one
  test start breaking other tests, so I need to try several hacks to
  find one that works. It's taking longer and longer to make each new test
  pass as I work through an ever-increasing cascade of bugs.
* At this point I realize that if I had spent a bit of time up-front
  coming up with an overall design and thinking about the big picture
  rather than just the next test, life would be a lot easier. Many
  of the bugs I've been chasing never would have happened in the first
  place. But I've written so much code that it would be painful to
  throw it out. So, I keep layering on Band-Aids.
* Eventually, I get everything working. It was painful and took
  longer than if I had done some design at the beginning. The
  code is in really bad shape.
* I know that I'm now supposed to "make it right", but that would
  require throwing out most of the code that I've written
  so far, and that would take a long time.
* I go to my boss to ask for advice. After I describe the situation,
  she says "Well, that project already took way longer than we
  thought it would, and I have a lot of other things I need you to work
  on. Since your code seems to work and you have extensive unit tests,
  let's leave it as is. Hopefully we won't need to go into that code to
  make changes, and maybe we can hire a summer intern to clean it up next
  year."

As this scenario shows, exposing a lot of bugs quickly isn't
necessarily a good sign...

**UB:**

(LOL)  That was a funny story -- if perhaps a bit speculative.  And like all speculation based upon inexperience, it's quite inaccurate. So let's correct a few things.

1. We certainly do think about the big picture and we certainly do spend time on working through an overall design.  The idea that we don't is an old and false meme that was used to criticize the Agile and Extreme Programming movements twenty years ago. If you've read any of my books you know that I talk about design and architecture a _lot_.  This is an activity I engage in very frequently at many different time scales: weekly, daily, hourly, and every few minutes.

1. Yes, of course the three laws are tactical -- they are a tactical discipline.  Writing code is tactical.  Every line of code you write is a tactical decision. In the best case those tactical decisions are guided by strategic thought at many different levels.

1. At the lowest level of strategy we write the tests first.  Therefore the production code must be designed to be testable. You cannot create a class that's hard to test if you write the tests first.  And a class that is easy to test _is_ easy to test because it is decoupled.

1. The REFACTOR phase of the RED-GREEN-REFACTOR loop identifies lower level strategic issues.  We look at the growing code, compare it to the overall plan for the current module, and clean it up as needed. This happens several times per hour.

1. Those of us who follow the Agile methods engage in daily reflections during which mid level strategic issues are identified.  We also engage in weekly or bi-weekly sessions where the highest level strategic issues arise.

1. The very first few days of an Agile project is generally spent working on an overall high level picture of the system.  This picture, though certainly wrong, informs the rest of the process.

1. We never show working code to our manager until after it has been cleaned -- because until it has been cleaned it is not done.  And we never delay cleaning to the end.  We engage in cleaning almost every time we get a test to pass.

**JOHN:**

I stand by my claims that TDD is tactical and discourages design.

**UB:**

Try it, Sam I Am.  And then tell me if you still think it discourages design.  
If you'd like some help you can I can pair on a few things.   Or if you'd like you can watch some of the videos I, or others, have done regarding this discipline.

**JOHN:**

I'm not saying it's impossible to do good design in a TDD environment,
but to do so you must develop additional processes to compensate for
the tactical pressure created by TDD. Perhaps the approaches you described
above, if diligently
executed, can produce an acceptable outcome. However, I fear that most
developers won't have the self-discipline to avoid the mess I described
earlier.

**UB:**

But you think that most developers have the self-discipline to write careful comments and work though, and then follow, up front designs?  

Discipline is discipline.  Either you've got it, or you don't.  And if you don't, you wind up with a mess.


>>At this point John you are speculating about things you don't know.  These are very old, and very tired, arguments that have been proven wrong over the last two and a half decades.  

>>The offer is open.  I'm happy to work with you to show you the discipline if you like. It is nothing at all like what you fear.

**JOHN:**

Even if it's possible to make TDD work, why use an approach that must be
compensated for? The fundamental problem with TDD is that it prioritizes
the wrong thing. Yes, tests are essential to software development.
But they are not the most important thing. The most important thing
in software is good design. Thus, the software development process
should be organized with design, not tests, as the highest priority.

Instead of TDD, I would argue that we should design and implement code
in larger chunks that encourage design thinking. Ideally, the unit of
coding will be a modest-size collection of related things with a coherent
mission, such as a class. Take a bit of
time up front to think about the sub-problems that will have to be
solved while implementing this abstraction. Look for simple components
that will help to solve those problems. Find commonalities, where
one piece of code solves multiple problems.

Then write the code for this chunk of functionality, probably in
stages, with unit tests. Personally, I prefer to write enough code
in each stage to (mostly?) solve a sub-problem so I can see how its
pieces work together (anywhere from 50-500 lines). Then I write the unit tests
for that sub-problem. I drive the unit tests from the code, working through
each method line-by-line, writing tests for a few lines of code at
a time (such as a loop or conditional). The structure of the tests
mirrors the structure of the code, so when I make code changes it's
easy to see which tests are affected.

That said, I don't object to other approaches to unit testing, as long
as they produce a comprehensive and high-quality suite of tests.
The thing I'm adamant about is that the development process
should be driven by *design*, not tests.

Also, to be clear, there is a limit to how much design can be done
before implementing anything; it's just too hard to visualize all of the
consequences of design decisions. And, even if you design up front, you
still won't get it right the first time. So plan on time to refactor
(it typically takes me three tries to get a new class right). But starting from
some design ideas will get you to good code a lot faster than starting
with spaghetti code.

Here are some questions for you:
* Do you agree that a good design is the most important goal of
  software development?
* Do you believe that TDD is more likely to produce a better design than other
  approaches that are design-centric? If so, please explain.
* If TDD produces a worse design, what advantages does TDD have that
  compensate for an inferior design?

You have argued that writing the tests first will lead to more testable
code ("You cannot create a class that's hard to test if you write the tests
first"). This is not at all obvious to me. Do you have evidence to support
this claim? Can you provide an illustrative example? I think what is more
likely is that with TDD developers simply won't write the hard tests. This
is easy to do when tests are created out of the blue, with no code to guide them.
On the other hand, if you write the code first, and if you are conscientious
about writing tests that cover all of your code, then you will be forced
to deal with the hard cases.

Now let's suppose that you are right, and that code produced via TDD is
easier to test than code that is ideal from a design standpoint. If
this happens, it means that the TDD code is not ideal from a
design standpoint. This sounds like a bad trade-off to me: it's
better to adopt the best possible design and spend a bit more time
writing tests. In my experience, though, this is mostly a non-problem:
hard-to-test code doesn't occur very often.  Every once in a while
I will encounter a situation where it is worth modifying the design
slightly (perhaps even making it a bit worse) in order to improve
testability. But again, this is very rare. So, why not focus on
what's most important (design) and only worry about testability
in the rare cases where it is a problem?

I'd like to raise an additional concern about TDD: I don't see how it
can produce good test coverage. If I write tests before I write code,
I can test the overall functionality of the code (effectively, its
interface) but I can't test how that functionality is implemented
(loop structure, details of special-case handling, etc.).
Don't I need to look at the code in order to determine the best
way to test it? For example, in `PrimeGenerator` some of the first
primes are generated with special-case code, which may require special
tests. How do I know that if I haven't seen the code?
It seems to me that TDD not only compromises design, but it also
compromises test quality. Am I missing something here?

**UB:**

Now I'm not going to tell you that TDD is sufficient.  I'm not going to tell you that we who pratice TDD don't also lay back in the hammock and consider design.  We do.  All the old wive's tales about TDD being the only design tool you need are baloney.  Of course we consider design and architecture.  We don't spend months, weeks, or even days on it; but we do think the problems through.

And then, of course, there comes the day when you realize you've chosen the wrong design.  This happens to everyone, even those of us who plan their designs for months.  This is a natural result of customers wanting to make changes that the developers could not predict.

The great thing about TDD is that the discipline produces a suite of tests that you can trust with your life.  And therefore you can easily refactor the design of the system in small increments without breaking anything.

**JOHN:**

This is indeed wonderful. But it is true of any approach that generates a
comprehensive unit test suite. It doesn't require the tests to be written
before the code.

**UB:**

I could go on.  I have gone on.  I've written several books on the topic.  The most complete one is _Clean Craftsmanship_.  You might want to give it a look.  ;-)

**JOHN:**

Um, how many of your books do I have to read before I can actually
write good code?  For example, I don't recall seeing anything in *Clean Code*
about ensuring a good design in spite of TDD (the word "design" doesn't
seem to appear in the chapter on testing); did I miss something?