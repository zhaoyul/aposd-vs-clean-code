>I thought it would be best to create a new file for my proposed TDD segment.

**UB:**

Let's turn to TDD.  As I said at the start I have carefully read _A Philosophy of Software Design_. I found it to be full of worthwhile insights, and I strongly agree with most of the points you make.  

So I was surprised to find, on page 157, that you wrote a very short, dismissive, pejorative, and inaccurate section on _Test Driven Development_.  Sorry for all the adjectives, but I that's a fair characterization.  So my goal, here, is to correct the misconception you have that led you to write that section. 

In that section you deride TDD as "tactical programming".  This is correct.  TDD is a tactical discipline.  I'm sure you agree that the act of coding is tactical, and that discipline is essential to tactical efficiency.  Indeed a quick thumb through the sections of your book reveals many tactical disciplines.  

So perhaps the first misapprehension you have is that TDD is viewed as something more than a tactical discipline.  You may fear that programmers who follow TDD follow nothing else.  In particular you may fear that they eschew considerations of design and architecture, or of what Rich Hickey calls _Hammock Driven Design_.  

So let me disabuse you of that fear.  As a 25 year adherent to the discipline of TDD, I am a strong believer in forethought, planning, design, and architecture.  As I told Jim Coplien, over twenty years ago, TDD without architecture is horseshit.

For the moment, let's focus on the tactic; then we;ll back off and look at how design and architecture dovetale with that tactic.  Perhaps the best way to describe the tactic is with an analogy.

Accountants and programmers have a very similar problem.  Both create arcane documents that only those within their profession can properly understand.  Both must be meticulously accurate because even one misplaced symbol can lead to disastrous results.  

Accountants address this problem using the tactical discipline of _Double Entry Bookkeeping_ (DEB).  One at a time they enter every transaction _twice_.  The two entries are complementary, and are entered in very different accounts.  They follow separate mathematical pathways until they meet on the balance sheet with a subtraction that must yield zero.

_Test Driven Development_ is the same.  One at a time programmers enter every desired behavior twice.  The two entries are complimentary, and excute together to yield a zero -- zero tests failed.

The rationale and the discipline are the same.  They differ only at the level of the symbols being entered, and the structure of the source code vs. the accounts.

As I am sure you know, the design of accounts for a large enterprise is a significant task that takes into account and fascilitates the discipline of DEB.  So too the design of source code modules is a signficant task that takes into account and fascilitates the discipline of TDD. As such both DEB and TDD have an impact upon the design of their corresponding accounts and modules.  Or, in other words, the tactic informs the strategy.

Perhaps another analogy would help.  Two thousand years ago the Romans learned that if their soldiers formed a shield wall confronting their enemies, and if each soldier attacked the enemy to his left, instead of the one in front of him, their killing was made much more effcient.  That was a tactic, driven by a strategy.  

So then, what is the tactic of TDD.  It exists in two layers.  The first layer is comprised of three laws:

1. You are not allowed to write the desired production code until you have first written a test that fails due to its absence.
2. You are not allowed to write more of a test that is sufficient to fail, and failing to compile counts as a failure.
3. YOu are not allowed to write more production code than is sufficient to pass the currently failing test.

A little thought will convince you that these three laws will lock you into a cycle that is just a few seconds long.  You'll write a line or two of a test that will fail, you'll write a line or two of production code that will pass, around and around every few seconds.  Tactical indeed!  And very similar to DEB.

Many programmers are disturbed, at first, by the shortness of that cycle.  They fear it will interrupt their flow and prevent them from thinking through their code.  They fear it will slow them down.  Experience, however, suggests that the opposite happens.  

The second layer is the Red-Green-Refactor loop.  This loop is several minutes long.  It is comprised of a few cycles of the three laws, followed by a period of reflection and refactoring.  During that reflection we pull back from the intimacy of the quick cycle and look at the design of the code we've just written.  Is it clean.  Is it well structured.  Is there a better approach.  Does it match the design we are pursuing?  If not, should it?  

At this point you may well ask "where is the overall design?".  We, of the Agile persuation follow your advice of incrementalism -- and that includes incrementalism of design and architecture.  But that doesn't mean we don't think ahead.

Generally, at the start of a project, there is a brief period of brainstorming over the design and architecture of the system.  This period might be a day or even a week.  The output is very rough but is enough to set us moving in a direction.  Then, with each sprint (I like 1 week sprints, some people like 2 week sprints), we begin with a re-evaluation and refinement of that design and architecture that informs the development effort during that sprint.

OK, I think I've said enough to start with. Given my brief and extremely high level description.  And given your own studies of the disciplines in question, let's move on to the discussion at hand.

What say you?
  