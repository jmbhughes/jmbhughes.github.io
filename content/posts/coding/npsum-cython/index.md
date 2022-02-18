---
title: Comparison of speed of np.sum in Cython
date: 2021-05-11
description: comparing np.sum speeds
menu:
  sidebar:
    name: np.sum speeds in Cython
    identifier: np.sums
    weight: 101
    parent: Coding
tags: ["Basic"]
categories: ["Basic"]
draft: false
---

I was curious how much the overhead of `np.sum` impacted Cython code.
That is, should you write your own sum method that loops or use `np.sum`?
So, I wrote up a little test definition of the two approaches, as shown below:

{{< highlight cython >}}

import numpy as np
cimport numpy as np

cpdef double sum_np(np.ndarray[double] a):
    return np.sum(a)

cpdef double sum_loop(double[:] a) nogil:
    cdef size_t i, I
    cdef double total = 0.0
    I = a.shape[0]
    for i in range(I):
        total += a[i]
    return total
{{< /highlight >}}

`sum_np` just uses `np.sum` directly. `sum_loop` instead manually sums using a for loop.

I then ran this many times to determine a runtime for each loop on a length 20,000 `numpy` array of numbers. On my computer `sum_loop` is 1.8 times faster than `sum_np`. For a smaller length 20 `numpy` array, `sum_loop` is 6 times faster than `sum_np`. This makes sense because the overhead of calling a `numpy` function is a greater fraction of the operation time.

Thus, if you're summing inside of a loop it might actually make sense to manually do the sums yourself, but if you're just summing once it might be easier to just use `np.sum` directly (unless you're looking for the most performant code).
