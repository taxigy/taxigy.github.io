---
layout: post
title: "How I learned Clojure while solving easy coding challenges"
date: 2015-08-03
categories: learning
crossposted: logdown
---

I decided to learn Clojure (and therefore ClojureScript, since the two intersect significantly) by doing. Who'd ever blame me with that?

I chose HackerRank as a platform for practice. There are other choices, of course, but I love HackerRank for two reasons:

* it offers a REPL-style flow: I write code, run it against some tests, get results instantly,
* before the code is submitted, I can test it against whatever cases I may imagine for as many times I need.

So I started from [Introduction section within Functional Programming domain](https://www.hackerrank.com/domains/fp/intro).

The first thing I encountered was that I didn't exactly know how to read from STDIN in Clojure. I also didn't know how to parse input string or stream, pass it into a function and split into substrings in particular.

The solution to this simple problem is, well, simple:

```clojure
(let [in (slurp *in*)]
    (println (clojure.string/split in #"\s")))
```

For input

```
2
4 3
-1 -3 4 2
4 2
0 -1 2 1
```

it returns

```
[2 4 3 -1 -3 4 2 4 2 0 -1 2 1]
```

which means pretty much what I need.

Or, in a less readable and more laconic form,

```clojure
(let [in (clojure.string/split (slurp *in*) #"\s")]
    (println in))
```

Typical challenge on HackerRank defines a sequence that, in its head, has a number that represents the number of test cases (in aforementioned example, there are 2 test cases). Knowing that, it's typically useful to extract this value into a separate variable. Good thing `let` evaluates its first parameter sequentially, so this is valid code:

```clojure
(let
    [in (clojure.string/split (slurp *in*) #"\s")
    tests (first in)
    input-data (rest in)]
    (print tests input-data))
```

It now outputs

```
2 (4 3 -1 -3 4 2 4 2 0 -1 2 1)
```

# HackerRank challenges

So here go some interesting (at the beginner level) challenges that I'd been puzzling over for relative long time, mostly because of lack of knowledge of `clojure.core` API.

## List replication
The problem is to output values from the input list `N` times each, `N` given as the first element of the input:

> Given a list repeat each element of the list n times. The input and output portions will be handled automatically by the grader. You need to write a function with the recommended method signature.

Sample input:

```
3
1
2
3
4
```

and sample output:

```
1
1
1
2
2
2
3
3
3
4
4
4
```

The most dumb problem in the world. And I've spent about 40 minutes on it.

The final code was

```clojure
(fn [num lst]
    (loop [head (first lst) tail (rest lst)]
        (if-not (nil? head)
            (do
                (dotimes [i num] (println head))
                (recur (first tail) (rest tail))))))
```

It didn't look clear or beautiful to me but I just couldn't do it better. Hopefully, things will change over time, positively.

## Filter array
The approach was very similar to the previous problem. This one is defined as:

> Filter a given array of integers and leave only that values which are less than a specified value X. The integers in the output should be in the same sequence as they were in the input. You need to write a function with the recommended method signature for the languages mentioned below. For rest of language you have to write complete code.

Sample input:

```
3
10
9
8
2
7
5
1
3
0
```

and sample expected output:

```
2
1
0
```

So, the problem can be decomposed into simple tasks:

* loop through the whole list `lst` and by the way
* check if current element is less than `delim`,
* and if true, print it,
* then recur with the rest of the list `lst`.

The code:

```clojure
(fn [delim lst]
    (loop [head (first lst) tail (rest lst)]
        (if-not (nil? head)
            (do
                (if (< head delim)
                    (println head))
                (recur (first tail) (rest tail))))))

```

## Filter positions in a list
The problem is simple, the program should filter out every second item of the list (one with an odd index):

> For a given list with N integers, return a new list removing the elements at odd positions.

Sample input:

```
2
5
3
4
6
7
9
8
```

and output:

```
5
4
7
8
```

The code:

```clojure
(fn [lst]
    (loop [odd false lst lst]
        (if-not (nil? (first lst))
            (do
                (if odd (println (first lst)))
                (recur (not odd) (rest lst))))))
```

For this problem, it took me about 5 minutes to reach the working solution. However, I was feeling that not using `let` makes the code a lot less comprehensible than it could be.

## Reverse a list

This one happened to be tricky for me, since I don't know much of Clojure's standard library. The `clojure.string` namespace seems to be deprecated for HackerRank FP challenges so I had to implement my own `join`, too.

The problem is to reverse a list without using `reverse` function, and then print it out each element on a new line.

Sample input: 

```
19
22
3
28
26
17
18
4
28
0
```

and output:

```
0
28
4
18
17
26
28
3
22
19
```

Now, my solution is a lot like freshman's would look like, but considering I'm a Clojure novice, I don't feel bad about it (maybe about 1/4 of bad):

```clojure
(fn [lst]
    (loop [in lst out (list)]
        (if (seq in)
            (recur (rest in) (conj out (first in)))
            (print
                (reduce str
                    (interpose "\n" out))))))
```

## Sum of odd elements

The challenge was simply to return the sum of all elements of an input list that are odd. The solution is so short I felt proud of myself. However, thanks to [Clojure by Example](https://kimh.github.io/clojure-by-example/) reference:

```clojure
(fn [lst] 
    (reduce + (filter odd? lst)))
```

## List Length

The challenge was to calculate the length of a list the hard way. Simple `reduce` got the job done:

```clojure
(fn [lst]
    (reduce (fn [n t] (inc n)) 0 lst))
```

## Update List

The caption isn't very clear, the problem statement is:

> Update the values of a list with their absolute values.

Sample input:

```
2
-4
3
-1
23
-4
-54
```

and output:

```
2
4
3
1
23
4
54
```

At first, I thought of `Math.abs`. So I tried

```clojure
(fn [lst]
    (map Math/abs lst))
```

but that threw with

```
Exception in thread "main" java.lang.RuntimeException: Unable to find static field: abs in class java.lang.Math
```

I'm not exactly sure why this happened, but, in a short search, I
discovered the solution:

```clojure
(fn [lst]
    (map #(Math/abs %) lst))
```

*upd: it was FIXME but then I found [an explanation](http://stackoverflow.com/questions/21753243/absolute-value-of-a-number-in-clojure#comment41125257_21753385)*

## Evaluating e^x

There's a function `e^x` that is defined as a series:

```
e^x = x^0 + x^1/1! + x^2/2! + x^3/3! + ... + x^n/n!
```

The goal is to find a sum of first 10 elements of `e^x`.

The problem perfectly decomposes into a set of simply solvable tasks:

* factorial of `n`,
* `x` to the power of `n`, and
* sum of series, actually of a fixed number of elements.

At first, it was a bit difficult for me to get away from for-loop imperative approach and rather switch to more mathematical methods. So I googled a bit and found great solution for both [factorial](http://stackoverflow.com/a/1663053/1287643):

```clojure
(defn factorial [n]
    (reduce * (range 1 (inc n))))
```

and [exponentiation](http://stackoverflow.com/a/5058544/1287643):

```clojure
(defn exp [x n]
    (reduce * (repeat n x)))
```

Next, the sum of the series is also a simple `reduce`. So the overall solution looks like this:

```clojure
(let
    [in (clojure.string/split (slurp *in*) #"\s")
    tests (read-string (first in))
    input-data (rest in)]
    (loop [input-data input-data]
        (when (not-empty input-data)
            (println 
                (reduce +
                    (map
                        (fn [l r]
                            (/
                                (reduce * (repeat r l))
                                (reduce * (range 1 (inc r)))))
                        (repeat 10 (read-string (first input-data)))
                        (iterate inc 0))))
            (recur (rest input-data)))))

```

It was accepted by HackerRank and got 100% score (20.00 pts), although its run-time is a bit too long for such a simple solution.

By the way, there are really great solutions submitted by other participants, [one in Scala](https://codepair.hackerrank.com/paper/k0vG9BVx?b=eyJyb2xlIjoiY2FuZGlkYXRlIiwibmFtZSI6InJpc2hhdCIsImVtYWlsIjoicmlzaGF0bXVoYW1ldHNoaW5AZ21haWwuY29tIn0%3D) and [one in Clojure](https://codepair.hackerrank.com/paper/TMZwdChN?b=eyJyb2xlIjoiY2FuZGlkYXRlIiwibmFtZSI6InJpc2hhdCIsImVtYWlsIjoicmlzaGF0bXVoYW1ldHNoaW5AZ21haWwuY29tIn0%3D), the latter deserves its place here:

```clojure
(doseq [i (range (Integer/parseInt (read-line)))]
  (println
    ((fn [x]
      (reduce +
        (map first
          (take 10
            (iterate (fn [[r k]] [(/ (* r x) k) (inc k)]) [1.0 1])))))
              (Double/parseDouble (read-line)))))
```

## Area Under Curves and Volume of Revolving a Curve

The [problem](https://www.hackerrank.com/challenges/area-under-curves-and-volume-of-revolving-a-curv) was, given a function, to calculate its definite integral.

For this problem, few links to related information were provided:

* [the limit definition of a definite integral](https://www.math.ucdavis.edu/~kouba/CalcTwoDIRECTORY/defintdirectory/), although a bit more mathematical and less computational,
* [areas and volume computation](http://tutorial.math.lamar.edu/Classes/CalcI/Area_Volume_Formulas.aspx), and an integral, in context of this problem, is a computation of an area.

However, I found more, and those were more useful:

* [formulating abstractions with higher-order functions](http://ecmendenhall.github.io/sicpclojure/pages/12.html#sec_1.3), a bit of useful Clojure-related info on integrals,
* [numerical integration](http://jeremykun.com/2012/01/08/numerical-integration/), a *very good* guide on different computational maths regarding to function integrals, with some visual perks,
* [a bit more comprehensible explanation of integrating circles formed by rotation](http://tutorial.math.lamar.edu/Classes/CalcI/VolumeWithRings.aspx),
* and even Wolfram|Alpha's [area calculator](http://www.wolframalpha.com/widgets/view.jsp?id=a0946d7385a92c63af1fba1f83cb2238).

So, it took me 2 pomodoros to do research and reach a half-working solution:

```clojure
(defn exp [x e]
    (reduce * (repeat e x)))

(defn build-function [multipliers powers]
    (fn [x]
        (reduce +
            (map
                #(* %1 (exp x %2))
                multipliers
                powers))))

(defn sum [term a next-term b]
    (if (> a b)
        0
        (+
            (term a)
            (sum term (next-term a) next-term b))))

(defn integral [f a b dx]
    (defn +dx [x] (+ x dx))
    (*
        (sum f (+ a (/ dx 2)) +dx b)
        dx))

(defn solve-flat [multipliers powers a b]
    (integral
        (build-function multipliers powers)
        a
        b
        0.001))

(let
    [multipliers
        (vec (map read-string (clojure.string/split (read-line) #"\s")))
    powers (vec (map read-string (clojure.string/split (read-line) #"\s")))
    ab (vec (map read-string (clojure.string/split (read-line) #"\s")))
    a (nth ab 0)
    b (nth ab 1)]
    (println
        (solve-flat multipliers powers a b)))
```

For the sample input 

```
1 2
0 1
2 20
```

it has output

```
413.99999999997846
```

which was close to the answer, but only for the first part of two. Because the problem was:

> The first Line will contain the area between the curve and the x-axis, bound between the specified limits. The second Line will contain the volume of the solid obtained by rotating the curve around the x-axis, between the specified limits.

So, the expected output was:

```
414.0
36024.1
```

However, this code was unstable, too, because for

```
1 2 3 4 5
6 7 8 9 10
1 4  
```

it returned

```
2432035.87874474
```

instead of expected

```
2435300.3
```

So I had to add volume calculation. A very brief googling got me to the [calculus explanation](http://tutorial.math.lamar.edu/Classes/CalcI/Area_Volume_Formulas.aspx) of calculation of the volume of a shape formed by rotation of a given function around x-axis.

The code then was:

```clojure
(defn exp [x e]
    (reduce * (repeat e x)))

(defn build-function [multipliers powers]
    (fn [x]
        (reduce +
            (map
                #(* %1 (exp x %2))
                multipliers
                powers))))

(defn build-circle-area-function [f]
    (fn [x]
        (*
            Math/PI
            (exp
                (f x)
                2))))

(defn sum [term a next-term b]
    (if (> a b)
        0
        (+
            (term a)
            (sum term (next-term a) next-term b))))

(defn integral [f a b dx]
    (defn +dx [x] (+ x dx))
    (*
        (sum f (+ a (/ dx 2)) +dx b)
        dx))

(defn solve-flat [multipliers powers a b]
    (integral
        (build-function multipliers powers)
        a
        b
        0.001))

(defn solve-volume [multipliers powers a b]
    (integral
        (build-circle-area-function (build-function multipliers powers))
        a
        b
        0.001))

(let
    [multipliers
        (vec (map read-string (clojure.string/split (read-line) #"\s")))
    powers (vec (map read-string (clojure.string/split (read-line) #"\s")))
    ab (vec (map read-string (clojure.string/split (read-line) #"\s")))
    a (nth ab 0)
    b (nth ab 1)]
    (println
        (solve-flat multipliers powers a b)
        (solve-volume multipliers powers a b)))
```

But it got me only [18 points](https://www.hackerrank.com/challenges/area-under-curves-and-volume-of-revolving-a-curv/submissions/code/12927134) out of possible 30. Kinda sad. 2 out of 5 tests failed.

It's a shame that I haven't come up with the right conclusion so I bought input and output samples for one of the tests, however only input was enough. The point was, I made up exponentiation function for only positive powers, e.g. `x^e` for `e >= 0`. So the next step was to enhance the exponentiation function to allow it calculate values for negative powers:

```clojure
(defn exp [x e]
    (if (> e 0)
        (reduce * (repeat e x))
        (reduce / 1 (repeat (- e) x))))
```

This worked perfectly. [20 out of 20 points](https://www.hackerrank.com/challenges/area-under-curves-and-volume-of-revolving-a-curv/submissions/code/12936641).