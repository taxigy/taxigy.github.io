---
layout: post
title: "How I learned Clojure while solving easy coding challenges, part 2"
date: 2015-08-20
categories: coding
---

[Previously]({% post_url 2015-08-03-learning-clojure %}) I made a little note about several things I've learned about Clojure while solving easy tasks. Later, I set my hands to the next subset of FP problems on HackerRank.

The subdomain is called “[Recursion](https://www.hackerrank.com/domains/fp/recursion)”, so it's expected that one employs recursive calls and, possibly, higher-order functions.

## Computing the GCD

The problem statement is simple: given two numbers, find their GCD using Euclidean algorithm. The code is simple, too.

```clojure
(defn gcd [a b]
    (if (= b 0)
        a
        (recur b (mod a b))))

(let [f (fn [a b] (gcd a b) ) [m n] (map read-string (re-seq #"\d+" (read-line)))] (println (f m  n)))
```

I'm not quite sure if `recur` or function name at the tail of the function body makes any difference. There's also an [explanation](http://iloveponies.github.io/120-hour-epic-sax-marathon/looping-is-recursion.html) of `recur` and `loop` and why those form recursion, not just a traditional imperative loop.

## Fibonacci Numbers

This one was interesting. Simple recursion caused TLE, termination due to too long execution.

```clojure
(defn fib [n]
    (if (#{0 1 2} n)
        (max 0 (- n 1))
        (+
            (fib (- n 1))
            (fib (- n 2)))))

(let [n (Integer/parseInt (read-line))]
    (print (fib n)))
```

It's because the complexity of this solution is `O(N!)`, the worst ever imaginable.

The good thing is that Clojure provides [memoization technique implementation](https://github.com/clojure/core.memoize) out of the box with `memoize` function that returns a memoized function:

```clojure
(def fib
    (memoize
        (fn [n]
            (case n
                0 0
                1 0
                2 1
                (+ (fib (- n 1))
                    (fib (- n 2)))))))

(let [n (Integer/parseInt (read-line))]
    (print (fib n)))
```

This solution was *far more* efficient and faster at run-time.

## Pascal's Triangle

I've [explained this problem in detail earlier]({% post_url 2015-08-18-pascal-triangle %}) so I won't make a stop here.

## String Mingling

The problem statement is complicated, however the extract it is simple: given two strings, [interleave](https://clojuredocs.org/clojure.core/interleave) them. But the first solution I made had, too, terminated due to timeout:

```clojure
(let [fst (read-line) snd (read-line)]
    (print
        (reduce str
            (map str fst snd))))
```

On the discussion forum, [somebody asked](https://www.hackerrank.com/challenges/string-mingling/forum/comments/28852) if anybody else experienced problem with higher-order functions. I got the idea and replaced `reduce` with simple loop:

```clojure
(loop [fst (read-line) snd (read-line)]
    (when (first fst)
        (print (str (first fst) (first snd)))
        (recur (rest fst) (rest snd))))
```

It worked, and again, in shorter time than at the first take.

## String-o-Permute

Again, very simple problem. Given a string, swap each pair of its elements. The funny thing was that, at the third time, my solution at first had TLE:

```clojure
(loop [tests (read-string (read-line))]
    (when (> tests 0)
        (let [input (read-line)]
            (println
                (reduce str
                    (map
                        #(clojure.string/join (reverse %))
                        (partition 2 2 input)))))
        (recur (- tests 1))))
```

Seemed like `partition` was too slow so I had to implement its alternative, narrower and faster in execution:

```clojure
(defn swap-pairs
    ([input] (swap-pairs input []))
    ([input result]
        (if (empty? input)
            result
            (swap-pairs
                (drop 2 input) 
                (conj result (second input) (first input))))))

(loop [tests (read-string (read-line))]
    (when (> tests 0)
        (let [input (read-line)]
            (println
                (clojure.string/join "" (swap-pairs input))))
        (recur (- tests 1))))
```

Done.

By the way, that was the first time I've employed [multi-arity](http://theburningmonk.com/2013/09/clojure-multi-arity-and-variadic-functions/) function in Clojure. Due to my previous experience both with Java and then Javascript, I missed this feature in the latter. The syntax in Clojure is pretty clean:

```clojure
(defn function-name
    ([param1] ...)
    ([param1 param2] ...))
```

The number of overloads and the number of params in each may vary.

## String Compression

Classical problem when you have to somehow change the input “in stream way”, which means you don't have to return back thus getting it done in `O(N)`. My problem here was, however, that the code was ugly:

```clojure
(defn truthy? [exp]
    (if (and exp (pos? exp) (> exp 1))
        exp
        nil))

(defn compress
    ([input] (compress input "" nil 1))
    ([input result last-char last-count]
        (if (empty? input)
            (str result last-char (truthy? last-count))
            (if (= (first input) last-char)
                (compress
                    (rest input)
                    result
                    last-char
                    (+ last-count 1))
                (compress
                    (rest input)
                    (str result last-char (truthy? last-count))
                    (first input)
                    1)))))

(println (compress (read-line)))
```

Will think about its improvement later, though. I'm sure there's a better, better readable and cleaner solution.


## Prefix Compression

Given two strings, find common prefix (from 0th element), cut it away from both strings and output all the three, provided with each one's length.

```clojure
(defn cut-prefix
    ([left right] (cut-prefix left right ""))
    ([left right prefix]
        (let [l (first left) r (first right)]
            (if (and l r (= l r))
                (cut-prefix (rest left) (rest right) (str prefix l))
                [prefix (clojure.string/join left) (clojure.string/join right)]))))

(defn output [[prefix left right]]
    (println (count prefix) prefix)
    (println (count left) left)
    (println (count right) right))

(let [left (read-line) right (read-line)]
    (output (cut-prefix left right)))
```

Ugly, but works.

## String Reductions

Given a string, reduce number of each unique character to one occurence. Nothing could be simpler than this because strings are sequences, and there's a function that extracts unique elements from a list into a (shorter) list.

```clojure
(let [input (read-line)]
    (println
        (clojure.string/join
            (distinct input))))
```

It was so simple that, at first, I thought the TLE would happen again. It wouldn't.

## Filter Elements

This task is marked as *easy* but I'm sure it's a bit harder than other easy problems. So, given a list, output a (shorter) list of elements that occur no less that N times, order preserved. For example, given

```
5 4 3 2 1 1 2 3 4 5
```

and for N equal to 2, the expected output is

```
5 4 3 2 1
```

The first thing, and very straightforward solution was to apply hash-maps, keys are elements in the list, and values are number of occurrences.

Hence the code

```clojure
(defn pass-when-enough
    ([threshold lst] (pass-when-enough (count lst) threshold lst))
    ([elements-count threshold lst] (pass-when-enough elements-count threshold lst {}))
    ([elements-count threshold lst result]
        (if (zero? elements-count)
            (map first
                (filter
                    #(>= (second %) threshold)
                    result))
            (pass-when-enough
                (- elements-count 1)
                threshold
                (rest lst)
                (merge-with + result {(first lst) 1})))))
```

But it didn't work at some of the tests. I've spent about 20 minutes on hunting for the explanation and found out a very important thing: hash-maps don't preserve order when merged! Thus, the original list must be preserved. Thus, I've rewritten the solution, although no less ugly:

```clojure
(defn pass-when-enough
    ([lst] (pass-when-enough (first lst) (second lst) (drop 2 lst)))
    ([threshold lst] (pass-when-enough (count lst) threshold lst))
    ([elements-count threshold lst] (pass-when-enough elements-count threshold lst {} []))
    ([elements-count threshold lst result-map result-lst]
        (if (zero? elements-count)
            (distinct
                (filter
                    (into {}
                        (filter #(>= (second %) threshold) result-map))
                    result-lst))
            (pass-when-enough
                (- elements-count 1)
                threshold
                (rest lst)
                (merge-with + result-map {(first lst) 1})
                (conj result-lst (first lst))))))
```

And it worked. There has to be a shorter and more comprehensible solution, though. But since I'm at the “learning by doing” phase, it doesn't matter now.

----

Learning a new language by solving series of simple tasks is great. However, it doesn't give any industry-level experience. But what I'm completely certain about is that one doesn't simply learn a new thing by either reading or doing, but rather by moving in both directions simultaneously. It takes a lot more time but it pays.