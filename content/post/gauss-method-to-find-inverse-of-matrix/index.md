---
title: Gauss method to find inverse of matrix
subtitle: Python Code
date: 2020-09-09T08:33:37.525Z
draft: false
featured: true
authors:
  - Riade Benbaki
categories:
  - Algorithmics
image:
  filename: matrix-3109795_1280.jpg
  focal_point: Smart
  preview_only: false
---
During an algorithmic challenge, I had to inverse a matrix of rational numbers. Usually the go-to method for this is to use a liner algebra package like Numpy. However, surprisingly, it is not possible to [invert a matrix of rationals (or fractions) using pure Numpy](https://stackoverflow.com/questions/33437023/linear-system-solution-with-fractions-in-numpy), unless you're willing to have a results as a floating point value (with the inherent precision loss). The solution is therefore to write your own liner solver.

One possible approach to this is using Gauss elimination method. I couldn't found any implementation to this, so I thought that sharing it could be useful in case you need a quick way to to this.

The following code does not use Numpy at all (since the algorithmic challenge I used it in did not support it). However, if you have the the possibility of using it, then you can replace the elementary operations on rows with Numpy operations, resulting in a shorted (and possibly faster) code.  



```python
def inverse_m(M,n):
    I = [] #create identity matrix
    for i in range(n): #Fill identity matrix
        L = []
        for j in range(n):
            if i == j: L.append(Fraction(1,1))
            else :L.append(Fraction(0,1))
        I.append(L)

    for i in range(n):
        j = i
        while M[j][i] == 0: #Looking for the first row with a non null value to use a a pivot
            j += 1
        #swap line i and line j
        M[i],M[j] = M[j], M[i]
        I[i],I[j] = I[j], I[i] 
        const = M[i][i] #pivot (non null)
        for j in range(n): #divide by pivot to have 1 in diagonal
            I[i][j] = I[i][j] / const
            M[i][j] = M[i][j] / const

        for j in range(i+1,n): #Elementary opeation to have 0 in the ith column of all rows below ith
            const = M[j][i]
            for k in range(n):
                I[j][k] = I[j][k] - const * I[i][k] 
                M[j][k] = M[j][k] - const*M[i][k]
    #At this point we have an upper trianguler matrix in M with ones in the diagonal
    for i in range(n):
        for j in range(i):
            const = M[j][i]
            for k in range(n):
                I[j][k] = I[j][k] - const * I[i][k]
                M[j][k] = M[j][k] - const * M[i][k]
    return I
```