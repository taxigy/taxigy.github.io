---
layout: post
title: "Pascal's Triangle in Clojure"
date: 2015-08-18
categories: coding
---

[Pascal's triangle](https://en.wikipedia.org/wiki/Pascal%27s_triangle) is a great problem that can be solved with recursion. However, I've done it with a loop that only resembles tail recursion, mostly because calculation shold be started from 0 and up, not from the parameter value and down; and because `loop` with `recur` get optimized at run-time.

[tic;smtc](https://en.wikiquote.org/wiki/Linus_Torvalds):

```clojure
(defn pascal [n]
    (cond
        (= n 0)
            []
        (= n 1)
            [1]
        :else
            (loop [iteration 1 lst [1] result []]
                (if (<= iteration n)
                    (recur
                        (+ iteration 1)
                        (vec
                            (concat
                                [1]
                                (map
                                    #(apply + %)
                                    (partition 2 1 lst))
                                [1]))
                        (conj result lst))
                    result))))
```

It then outputs a vector of vectors that each has proper binomial coefficients:

```clojure
(pascal 0)
;;=> []

(pascal 1)
;;=> [1]

(pascal 2)
;;=> [[1] [1 1]]

(pascal 3)
;;=> [[1] [1 1] [1 2 1]]

(pascal 10)
;;=> [[1] [1 1] [1 2 1] [1 3 3 1] [1 4 6 4 1] [1 5 10 10 5 1] [1 6 15 20 15 6 1] [1 7 21 35 35 21 7 1] [1 8 28 56 70 56 28 8 1] [1 9 36 84 126 126 84 36 9 1]]

(pascal 25)
;;=> [[1] [1 1] [1 2 1] [1 3 3 1] [1 4 6 4 1] [1 5 10 10 5 1] [1 6 15 20 15 6 1] [1 7 21 35 35 21 7 1] [1 8 28 56 70 56 28 8 1] [1 9 36 84 126 126 84 36 9 1] [1 10 45 120 210 252 210 120 45 10 1] [1 11 55 165 330 462 462 330 165 55 11 1] [1 12 66 220 495 792 924 792 495 220 66 12 1] [1 13 78 286 715 1287 1716 1716 1287 715 286 78 13 1] [1 14 91 364 1001 2002 3003 3432 3003 2002 1001 364 91 14 1] [1 15 105 455 1365 3003 5005 6435 6435 5005 3003 1365 455 105 15 1] [1 16 120 560 1820 4368 8008 11440 12870 11440 8008 4368 1820 560 120 16 1] [1 17 136 680 2380 6188 12376 19448 24310 24310 19448 12376 6188 2380 680 136 17 1] [1 18 153 816 3060 8568 18564 31824 43758 48620 43758 31824 18564 8568 3060 816 153 18 1] [1 19 171 969 3876 11628 27132 50388 75582 92378 92378 75582 50388 27132 11628 3876 969 171 19 1] [1 20 190 1140 4845 15504 38760 77520 125970 167960 184756 167960 125970 77520 38760 15504 4845 1140 190 20 1] [1 21 210 1330 5985 20349 54264 116280 203490 293930 352716 352716 293930 203490 116280 54264 20349 5985 1330 210 21 1] [1 22 231 1540 7315 26334 74613 170544 319770 497420 646646 705432 646646 497420 319770 170544 74613 26334 7315 1540 231 22 1] [1 23 253 1771 8855 33649 100947 245157 490314 817190 1144066 1352078 1352078 1144066 817190 490314 245157 100947 33649 8855 1771 253 23 1] [1 24 276 2024 10626 42504 134596 346104 735471 1307504 1961256 2496144 2704156 2496144 1961256 1307504 735471 346104 134596 42504 10626 2024 276 24 1]]
```

A little note: `cond` demands even number of arguments, and for each `2k`th, if it's true, `2k+1`st is returned. There, `:otherwise` symbol means anything that is truthy. It in just the same way could be `:else` (which is a conventional and widespread way) or even `true`.

The code employs several cool features that Clojure provides:

* `cond` decision tree,
* `loop` + `recur` (of course), and
* `partition` along with `map` to slice vector and map function to the slices.

The first two aren't interesting. The latter is.

Partitioning is a great tool to manipulate slices of a list or vector. In its shortest form, it takes a number of partitioned elements and a sequence:

```clojure
(partition 1 (list 1 2 3 4 5 6 7))
;;=> ((1) (2) (3) (4) (5) (6) (7))
```

If three parameters are provided, the first defines the length of each partition, the second stands for *step*, and the third is list itself:

```clojure
(partition 2 1 (list 1 2 3 4 5 6 7))
;;=> ((1 2) (2 3) (3 4) (4 5) (5 6) (6 7))

(partition 3 3 (list 1 2 3 4 5 6 7))
;;=> ((1 2 3) (4 5 6))
```

See that elements that don't fit into the slice are dropped off.

Pascal's triangle is calculated the way that `i`th element is a sum of `i-1`th and `i`th elements of the sequence that was calculated at the previous step. This is why for every pair of elements in

```clojure
(partition 2 1 (list 1 3 4 3 1))
;;=> ((1 3) (3 4) (4 3) (3 1))
```

we need to calculate its sum. The easiest way is to map with `map`.

```clojure
(map + (partition 2 1 (list 1 3 4 3 1)))
;; ClassCastException Cannot cast clojure.lang.LazySeq to java.lang.Number
```

This happens because `+` is applied to sequence (say, `(1 3)`). To apply it to numeric values, we need to pick them from the initial sequence. But it's impossible to get (using `get`) an element from a lazy sequence by index. Before that, we need to cast it to vector:

```clojure
(map #(+ (get (vec %) 0) (get (vec %) 1)) (partition 2 1 (list 1 3 4 3 1)))
;;=> (4 7 7 4)
```

The problem is that it's impossible to map sequence to `+`, so here `apply` enters the battle:

```clojure
(map #(apply + %) (partition 2 1 (list 1 3 4 3 1)))
;;=> (4 7 7 4)
```

The last step is to ensure that first and last elements of the new sequence remain. Simply `concat` them:

```clojure
(concat [1] (map #(apply + %) (partition 2 1 (list 1 3 4 3 1))) [1])
;;=> (1 4 7 7 4 1)
```

For convenience and validity of `conj`, make up vector from it. To do it, embrace with `vec`. Done.