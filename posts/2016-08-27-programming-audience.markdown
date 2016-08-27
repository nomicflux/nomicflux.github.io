---
title: Programming a Dialogue
---

We program in languages.  Languages are used for expression, for talking, for dialogue.  But who do we write programming languages *to*?

I propose that the three main audiences of a programming language are the Computer, the Idea, and the Human.

My point here is not to put one of these approaches above the others; in point of fact, I think that a professional programmer should be able to think from all three points of view, and balance them as needed for a given task.  An operating system should focus on the Computer, while scripts used by people from various backgrounds should prioritize the Human.  Most projects would benefit from thinking through all three angles on a deeper level.

I also do not take this to be a clean-cut, rigorous partitioning of programming languages; it's merely a distinction which I find helpful in guiding my thinking about the craft.  The "cons" in particular are not criticisms, but merely meant to be an assessment of likely pain points for different approaches.  YMMV.

## The Computer

#### The Gist

The first dialogue partner in programming is, of course, the computer.  On this level, the programmer has to think about how to express her thoughts to this Von Neumann machine, which understands little more than how to move bits around and combine them.  The programming language (one can think of FORTRAN or C, or assembly language if one prefers) is a thin abstraction over this bit shuffling, taking things from one region of memory into another.

#### The Code

Here's an in-place QuickSort algorithm, designed for efficiency and lacking any wasted space:
````c
void quickSort(int* arr, int left, int right) {
   int mid;

   if( left < right ) {
       mid = partition( arr, left, right);
       quickSort( arr, left, mid-1);
       quickSort( arr, mid+1, right);
   }
}

int partition( int* arr, int left, int right) {
   int pivot, i, j, temp;
   pivot = arr[left];
   i = left;
   j = right+1;

   while(i < j) {
   	do ++i; while( arr[i] <= pivot && i <= right );
   	do --j; while( aarr[j] > pivot );
    if(i < j) {
   	    temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
   }
   temp = arr[left];
   arr[left] = arr[j];
   arr[j] = temp;
   return j;
}
````

#### The Pros

For the present time, programming is done on computers.  It all comes down to combining and moving things in memory; reasoning about efficiency of an algorithm or about what is actually possible to do on a computer.  So understanding this aspect is essential to knowing what a program will, in fact, do.  Fancier paradigms will come and go, but every piece of code will need to address this audience at some point.

#### The Cons

No one needs to be told that low-level programming can be hard to read, hard to understand, and prone to all sorts of memory errors.  And while using bit flippin' magic can be fun and clever, it is seldom good code.

In addition, code which focuses on the Computer can obsess overmuch about solving one particular issue, leading to non-reusable code and to constant hacks to keep code updated.  Saving time on processing information may not be worth the cost of additional human hours spent writing, mainting, and debugging the code.

## The Idea

#### The Gist

Other partners include the ideas themselves.  Functional languages capture this conversation partner the best, by endeavoring to be expressions of the lambda calculus and logic itself.  In such a language, the programmer can more or less jot down the overall blueprint for a project, then write up that blueprint as code.

#### The Code

Here's a Haskell version of the QuickSort algorithm, showing the Idea-oriented approach.  It's not meant to be a direct comparison to the C version above; one could write an in-place QuickSort in Haskell, but it would be less demonstrative of the point at hand.
````haskell
quicksort :: [Int] -> [Int]
quicksort [] = []
quicksort (x : xs) = let left = filter (< x) xs
                         right = filter (>= x) xs
                     in (quicksort left) ++ [x] ++ (quicksort right)
````

#### The Pros

Idea-based programming is excellent at carving reality at its joints.  It finds how exactly to put together and take apart components of code, leading to reuse, composability, and separation of concerns.  Since it is the closest to the project blueprint, it can also lead to the quickest prototype for a complex solution - jot down the basic datatypes, scribble out the main data transformations necessary, and fill out the details one small piece at a time in a cohesive manner.

#### The Cons

As Randall Monroe has put it, ["Functional programming combines the flexibility and power of abstract mathematics with the intuitive clarity of abstract mathematics."](https://www.xkcd.com/1270/).  Idea-based programming is powerful, but it can require a steep learning curve, and does lend itself to terseness of thought to the point of unapproachability and to the exploration of difficult concepts for their own sake.  There is a tendency toward complexity couched as concision, which impacts the understandibility of the code.

In addition, not every project needs a full-out logical analysis.  Some projects are more exploratory by nature, discovering their logical joints as they go rather than starting with their conception.

## The Human

#### The Gist

Finally, though perhaps most importantly, a program is written for humans.  Human beings still need to read the code and write the code, at some level.  We are moving beyond this is some applications - for example, computers can generate HTML, so it is not necessarily important for such HTML to be written for the sake of human beings.  But the code generating the HTML still needs to be read and maintained.  Languages which focus on this approach tend to be dynamic scripting languages, such as Python or Ruby.

#### The Code

A simple algorithm like QuickSort doesn't really show the Human-readable aspect of Python; the Haskell version would be more readable.  Ultimately, the different approaches are not tied to one language or another.  But the Human-oriented approach is best seen in taking a complex task and reducing it to a couple quick imports and simple method calls, so I'm switching to data analysis here to demonstrate that:
````python
import pandas as pd
from sklearn.ensemble import GradientBoostingClassifier

with open(filename, 'r') as handle:
    csvfile = pd.read_csv(handle)

csvfile.loc[csvfile["Sex"] == "male", "Sex"] = 0
csvfile.loc[csvfile["Sex"] == "female", "Sex"] = 1
csvfile["Age"] = csvfile["Age"].fillna(base_age)
base_fare = csvfile["Fare"].median()
csvfile["Fare"] = csvfile["Fare"].fillna(base_fare)
train_data = csvfile["Sex", "Age", "Fare"]
train_labels = csvfile["Survived"]

alg = RandomForestClassifier(
          n_estimators=10,
          max_depth=3,
          min_samples_split=params.get('min_samples_split', 1))

alg.fit(train_data, train_labels)

predictions = alg.predict(train_data)
print("Predictions:\n{}".format(predictions))
````

#### The Pros

One nice thing about Human-oriented programming is that it tends to lead toward simplicity of expression and content, and not merely in the elegant-but-dense concision of the Idea-oriented approach or the clever-but-incrutable tricks of the Computer-oriented approach.  Simplicity of this sort has a side-effect of creating cleaner code with fewer potentials for bugs, and for ease of getting a project up and running.  While one might want C for implementing the innards of a deep neural network, or Haskell for automating the process of running a data experiment, a language like Python or R can parse through a CSV and generate basic exploratory graphs in the time that one is still importing libraries and setting up data structures in other languages.

#### The Cons

The simplicity of Human-oriented programming and the attendent ease of initial understanding has the potential to trade long-term quality for short-term gains.  It is possible (and quite easy) to write a program where every individual step is easy to read and test, but where integration of all the steps into an application is a nightmare.  Many problems of this sort reveal themselves only after considerable time has been spent on a project.

Simplicity can also be a lack of precision.  Both the Computer-oriented and Idea-oriented approaches fixate on one sort of precision, sometimes at the cost of missing the forest for the trees; Human-oriented approaches can sometimes dismiss added precision out of hand when it would lead to better-architected and even better-understood programs in the long run.
