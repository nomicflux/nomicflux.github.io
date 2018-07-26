---
title: P-adic Visualizations
---

How do we calculate how far apart numbers are?

This would seem to be an easy question; just subtract them.  5 and 2 are 5 - 2 = 3 units away, whatever we're counting.  And
most of the time, it's the best answer.

But this a distance that relies mainly on addition and subtraction.  There's another sort of distance which is more
closely related to multiplication, division, and cycles.

To give a taste of this sort of distance, let's take hours on a (12-hour) clock.  How far apart are 1 and 2?  1 hour.
This is the same whether we're talking about 1 hour of real time passing, 13 hours, or 145 - on the clock, we wrap
around every 12 hours.  Cycles are a common part of our life; additive and subtractive distances assume that everything can grow
infinitely apart, but those aren't the only sort of distances in our experience.

This sort of clock arithmetic, "modular arithmetic", is an interesting subject in itself.  But sometimes in nature,
things aren't quite cyclical; they look almost like cycles, but then if you zoom in, you'll see additional cycles
hanging out on those cycles.  P-adic distances take modular distances a step farther: we can have our cycles, then cycles on cycles, then cycles on cycles on cycles.  (Cue a "Yo Dawg" meme
here.)  We want to be able to _refine_ our analysis at every level.  So let's begin our descent.

(Specifically, p-adic numbers use *prime* cycles, hence the p-.  So we'll have 3-adic, 5-adic,
7-adic, but not 6-adic numbers.  But the reasons why won't be relevant to the discussion here.)

### Cyclical Refinements

Let's work with 3-adic numbers:

![3-adic numbers](/images/3-adic.png)

As one might expect, everything is in threes.  The main idea is this: first, we take a number (say 29) and calculate
what it is modulo 3 (that is, what is the remainder when divided by 3).  This is 2.  This first step will take us a
distance of 1 from the center, angled toward the top; a number like 10 which is 1 modulo 3 would be angled toward the bottom
triangle, and any clean multiple of 3 toward the rightmost one.  (The precise angles are unimportant, as long as we
can space them in thirds around the center.)

But unlike modular arithmetic, we don't stop there.  That's just the information we got from the first cycle of 3s; what
about the next one?  That is, how do these numbers compare with regard to 3*3, and 3*3*3, and so on?

We start by representing each number in base-3.  So 0 is 0, 1 is 1, 2 is 2 ... then 3 is 10.  10 is 101, 29 is 1002.  
Normally, we represent numbers in base-10, so the furthest digit right
is the 1s place (10^0), then the 10s place (10^1), then 100s (10^2), and so on.  Other bases work the same way,
building off of powers of the base.  So in base-3, we have a 1s place, a 3s place, a 9s place, a 27s place, and so on.

So why is 29 written as 1002?  Well, 2 is the 1s place, then we have 0 for the 3s and 9s places, followed by 1 in the
27s place.  So, 2*1 (+ 0*3 + 0*9) + 1*27 = 29.

The digit furthest right gives us the first triangle destination: 

![3-adic: 0 to 2](/images/3-adic-3.png)

then the next digit gives us the triangle inside that triangle: 

![3-adic: 0 to 8](/images/3-adic-9.png)

then the triangle inside the triangle inside the triangle: 

![3-adic: 0 to 26](/images/3-adic-27.png)

as far as we want to go.

Representing numbers in different bases isn't all that unusual, however.  That would leave these the same old numbers as
always, just with funny costumes.  What we'll be doing differently is that, with each digit left we traverse, we'll
make our steps *smaller*.  Normally, we think of changing the 10s place as being much more of an event than changing the
1s place, with the 100s place even more so.  And with our normal additive and subtractive distances, that is true.  But
we're looking at cycles here.  The first cycle, given by our number modulo 3, is the biggest sorter of numbers.  After
that, we're making continual *refinements* - 6 and 18 are both multiples of 3, so are more like each other than either
is to 7 in that regard, but 18 is also a multiple of 9, which makes it a little three-ier than 6.  It not only can come
back to the start if we wrap it around a clock with 3 hours, but also with 3 of these 3-handed clocks.  54 can manage 3 cycles of 3 cycles of 3.  

This also leads to a principle of *coherence*.  Let's take our second level of triangles again:

![3-adic: Focus on 5](/images/3-adic-9-2.png)

5 is 12 in base-3 notation (2 in the 1s place, 1 in the 3s place, to give 2*1 + 1*3 = 5).  Now, we add another
layer:

![3-adic: Focus on 5, another layer deep](/images/3-adic-27-2.png)

Take a look at the dots surrounding 5 (12).  We just prefix each one by a different digit; 5 is 012 (putting a zero in
front doesn't change the number), 14 is 112, and 23 is 212.  As we zoom in, we add our new digits to the left, but
everything in that spot will share the same base digits.  You can check this out with the triangles starting with 2 and
8 as well.

So at each step, we tweak our numbers a little more.  We take our triangles, and separate them out into triangles, and
separate those out into triangles.  The farther left we go in our digits, the *smaller* the distances become, and the
closer our new triads of numbers will be in 3-adic distances.  This is what will create new numbers for us.

(We can also view these numbers around a circle, given by their distance from 0:

![3-adic norm around a circle](/images/3-adic-circle.png)

To try out different norms and to watch them animate as numbers flow from one to the next, go to:
[https://nomicflux.github.io/PadicVisualization/](https://nomicflux.github.io/PadicVisualization/).)

### New Numbers

Normally with numbers, we can create fractions.  Then we "fill the gaps" between those fractions; that is how we get
the irrational numbers, like pi.  We can find the cracks in between any two fractions, and go one level deeper; if we
follow all these paths left in between the cracks, we can end up with a *completion* of the rational numbers, the
numbers represented by fractions.

P-adic numbers have their fractions too, which I won't get into here (in the visualization, think about being able to
zoom out one extra set of triangles at a time; we can do that any number of times, but this
doesn't change the overall picture much.  We've just added another level of triangles in a stack that's already
triangles all the way down).  Remember, sizes and distances are given by how close the numbers are concerning cycles of
3 (or 5, or 7, or other numbers we might be using for comparison).  So when we *complete* this system by filling in the
cracks, we won't be working our way between fractions in the same way.  We'll be finding numbers yet three-ier
(five-ier...) than have been found before, charting new paths within:

![3-adic: Making a path](/images/3-adic-27-path.png)

The distance between, say, 4 and 2 is 1 (to start with a baseline); they are not even in the same starting triangle.  This is also true
of the distance between 100 and 2.  5 and 2 are in the same
starting triangle, though; 5 - 2 is 3, but the distance will be closer than 1.  So we'll say it's 1/3 (details on why
would require a more in-depth study; for now, the important part is just that it must be smaller than 1 so that the
numbers are closer, more three-ly together).  11 - 2 is 9, which being 3*3 must give an even smaller distance, which
we'll say is 1/9; this means that 2 and 11 are not only in the same starting triangle, but the same second-level
triangle.  The distance between 20 and 2 is also 1/9; 20 - 2 = 18 which similarly places them in the same second-level triangle, since 18
is a multiple of 9.  

So remember that as odd as this new distance seems, it translates literally into how close the numbers are in the
visualization:

![3-adic: 0 to 26, redux](/images/3-adic-27.png)

To fill out our paths, we'll need to extend the numbers outward to the left, exploring numbers which in regards to three-iness
are always going one triangle deeper in than what we have.  We can extend this to infinity, just like we would keep
going to the right with decimals, such as with 3.14159... to get pi.

When we simply wrote our numbers in base-3, they were the same numbers donning tri-corner hats.  When we changed the
way we calculated distances between them, they started getting a little odd, perhaps imbibing too much at the math
social.  Now, however, we have created an entirely new set of numbers, by tracing paths deeper and deeper into the
triangle abyss.  Our tipsy digits have launched an expedition to the Pacific and found the ruins of R'lyeh with all it
contains.  These new numbers simply *do not exist* in the normal number system.  They cannot be created in it, cannot be
represented in it.

(I will not explore the following point more in this article, but in addition to creating eldritch numerosities, we have
created a number of numbers that we already know.  For example, in 3-adic numbers written in base-3, what would be ....22222 + 1?
Carrying the 1 out to infinity, we get: 0.  ....22222 is then -1.  Starting from nonnegative integers, we have gotten
negatives for free.  If you play around a bit, you can find that we also have numbers we would have regarded as
fractions in our normal numbers, but constructible as these infinite integers.  See if you can find a number x such tha
5*x = 1 in 3-adic numbers; this makes x 1/5.)

### Continuous Analysis

This completion of a p-adic number system allows for something quite unusual: we have created a *continuous* number
system (of course, mathematically, that was what we were going for; a system where all Cauchy sequences converge).  
We can do calculus with these new numbers, using most of the rules we may have learned and lov... well, learned once
upon a time.

The reason this is so weird, however, is that our p-adic number system is entirely based on how divisible numbers are.
We have taken the concept of breaking down numbers into discrete chunks, entirely concerned with the whole blocks we can
make from counting and use to make numbers, and made it analyzable by techniques entirely concerned with infinitesimal
but continuous changes and their impact.  P-adic numbers unite these disparate mathematical disciplines, granting
us access to a new toolbox which shouldn't work, but does.  It's like finding a way to turn a philosphy degree into a
paying, non-academic job.  (Before the hatemail arrives, this is a point taken from personal experience.)

I will give one example before closing out this post.  Around Calculus II in the US curriculum, we start learning about
series.  In particular, there is the Taylor series of a function (wikipedia refresher
[here](https://en.wikipedia.org/wiki/Taylor_series)): we can take a function and analyze it at a single point.  We first
take the value of the function at that point, then we take the tangent line at that point using the first derivative.
We keep adding on derivatives to further nuance our approximation.  It is like being an ant on the function, trying to
reconstruct the entire curve based just on the nuances available right around you.  Some functions lend themselves well
to this analysis; others will give very different approximations based on how they look from different points.

The key takeaway is that we can analyse functions locally to get a picture of the whole from that point.  P-adic numbers
are effectively doing the same thing with numbers; instead of derivatives, we are viewing number modulo higher and
higher powers of p.  (This is not just a metaphor; this insight is behind the creation of the p-adic numbers).

Most works on the p-adic numbers will dive in deeper starting with this insight.  My point here is to have generated some of
the intuitive, visual appreciation for these oddities, and to provide a launching pad for further study.
