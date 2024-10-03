+++
title = 'Efficient Inclusion Check'
date = 2022-03-25
draft = false
math = true
+++

## Problem Statement

I recently stumbled upon a cute problem where you have three sets of numbers
$$A, B, C \subseteq \lbrace 0, \dots, n \rbrace.$$ You want to answer if there exists $$a \in A, b \in B \text{ such that } c = a + b \in C.$$ It's a conceptually simple problem: are there two numbers from the first two sets that sum to a number in the third set? The hard part is we want to solve in it O(n log n) time.

## Brute force search

The simplest solution is to define the C set as a set in Python so that we can do constant time inclusion checking.

```py
for a in A:
  for b in B:
    if a+b in C:
      return True
return False
```

Thus the above code is O(n²), not our goal yet. (Or if we don't have efficient checking of inclusion in C, it's O(n³).)

## Clever search

We can do better! The observation that we can represent the sets as polynomials is critical.
Thus let, $$A = \lbrace a_1, a_2, a_3, \dots, a_n \rbrace$$ be represented as $$p_A(x) = x^{a_1} + x^{a_2} + \dots + x^{a_n}.$$
Do the same for B and C. Then, consider $$p_A(x) \times p_B(x).$$

Let's run a little example to see why this matters.
$$ A = \lbrace 1, 4, 9 \rbrace, B = \lbrace 2, 10 \rbrace, C = \lbrace 12, 1000 \rbrace$$
Then, $$p_A(x) = x + x^4 + x^9$$ and $$p_B(x) = x^2 + x^{10}$$ and $$p_C(x) = x^{12} + x^{1000}.$$
Notice though that $$p_A(x) \times p_B(x) = x^3 + x^6 + 2 x^{11} + x^{14} + x^{19}.$$
We only have cofficients greater than zero for terms that are the sums of elements in A and B!
Thus, if we can efficiently calculate the polynomial product, we can just compare the coefficient list to C.

### Efficient polynomial multiplication

It turns out multiplying polynomials in O(n log n) time is possible if we do it in Fourier space.

```py
from numpy.fft import fft, ifft
from numpy import real, imag

def polynomial_multiply(a_coeff_list, b_coeff_list):
  """
  Return the coefficient list of the multiplication
  of the two polynomials
  """
    n, m = len(a_coeff_list), len(b_coeff_list)

    # we pad with zeros since c is size m + n - 2
    a_fft_coeffs = fft(a_coeff_list + [0.0]*(m - 1))
    b_fft_coeffs = fft(b_coeff_list + [0.0]*(n - 1))

    c_fft_coeffs = [a*b for a, b in zip(a_fft_coeffs, b_fft_coeffs)]
    c = [real(v) for v in ifft(c_fft_coeffs)]
    return c
```

### The final solution

```py
def check_sum_exists(a, b, c, n):
  """
  Inputs:
  sets a, b, c
  n is the maximum number in a, b, c together
  """
  # set up the polynomial coefficients
  a_coeffs = [0]*n
  b_coeffs = [0]*n
  for aa in a:
    a_coeffs[aa] = 1
  for bb in b:
    b_coeffs[bb] = 1

  # multiply them together
  c_coeffs = polynomial_multiply(a_coeffs, b_coeffs)

  # Make sure the coefficients are integers without floating point errors
  coeffs_copy = []
  for num in c_coeffs:
    if(abs(num-0) < abs(num-1)):
      coeffs_copy.append(0)
    elif(abs(num-1) < abs(num-2)):
      coeffs_copy.append(1)
    else:
      coeffs_copy.append(2)

  return any([coeffs_copy[cc] >= 1 for cc in c])
```

## Conclusion

This cute solution is an approach that I wouldn't arrived at initially.
Just because you don't see a more efficient solution doesn't mean there isn't one.
