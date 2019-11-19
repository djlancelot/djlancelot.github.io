---
layout: post
title:  "Weighted Levenshtein distance"
date:   2019-11-10 23:43:06 -0800
categories: datascience 
---
In this post I'll introduce a two new variants for the Damerau–Levenshtein distance calculation &mdash; specifically for an extended version of the Wagner–Fischer algorithm &mdash; to dynamically change the cost of the edit step based on the position of the changes.

## Summary of the Damerau-Levenshtein distance

The Wagner–Fischer algorithm calculates the edit disance between two strings. It is a dynamic programming algorithm that uses an m by n matrix to calculate the edit distance in between two words $w_{1}$ and $w_{2}$. For the original Levenshtein distance there are 3 kinds of operations available for two characters in the string:
- **Deletion** (Transformation from main to man by removing the *i* character)
- **Insertion** (Transformation from man to main by inserting the *i* character) 
- **Sustitution** (Transformation from main to gain by modifying the leading *m* character to *g*)

The Damerau–Levenshtein variant introduces one more operation, which is the **transposition** of the characters. The changes presented in this post were implemented on the Damerau–Levenshtein variant, but can be implemented for the original Levenshtein distance metric by removing the additional operation from the code.

The Damerau–Levenshtein distance provides the edit distance between two strings based on the 4 operations mentioned above. For example the edit distance between the words *Martha* and *Marha* is 1, because with the removal of the *t* character (or the addition of it) the other string can be generated. The edit distance for *Main* and *Gain* is also 1 as the *M* can be substituted by a *G* or vice versa to generate one word from the other. This metric is used for correcting typing errors in texts. The typed word is matched against a vocabulary and the word with the lowest Levenshtein distance is suggested as a correction for the word.

The distance can be normalized between 0 and 1 by dividing the distance by the length of the longest character. In the $D$ m by n matrix determined by the Wagner-Fischer algorithm this value is 

$norm = 
\cases{
    d_{m,0} & $\text{if } m \ge n$ \cr
    d_{0,n} & $\text{if } m \lt n$ 
}
$

In some use cases, like similar company name detection the end of the string is less important than the beginning. Especially in company names, some article mentions use the business type abbreviation or other suffixes with the name while others omit the abbreviation or write alternatives. For example variants like *Lucky Ltd*, *Lucky Limited* or *Lucky* might still refer to the same company. 
In this article 2 algorithms will be shown to overcome this problem and add different weights for the changes towards the end of the strings than the beginning of the string.

## Weighted Generalised Levenshtein distance

The generalised algorithm introduces three new input parameters to the algorithm: 
- an initial value
- a weight function
- a decision function

Based on the first two parameters a weight vector is created ( $w_{n}$ ) with the size of the longer string.
This weight vector will be shared with the other string.

$w_0 = w_{init}$

$w_{i \in n} = f_w(w_{i-1})$

In the original algorithm the cost of each modification is 1. In the proposed algorithm instead of the constant value, 
one of the weights are used at the position.
- The minimum of the two weights should be picked in case of a monotone decreasing weight function
- The maximum of the two weights should be picked in case of a monotone increasing weight function  

$c_{ij} = 
\cases{
\text{min} ( w_{i} , w_{j} ) & $f_w \text{ monotone decreasing}$ \cr
\text{max} ( w_{i} , w_{j} ) & $f_w \text{ monotone increasing}$
} 
$

Using the above decision functions will make sure that the algorithm can be executed in a greedy fashion, 
the cost of making one change at two positions will be the sum of the cost of a change at one of the positions and 
the cost of the change for the other position.

The first column and first row shows the cost of changing the string represented by the row or the column into a 0 length string (no string at all).
This is a cumulative sum of the step cost over the length of the string. 
As this case covers the possible outcome that one of the strings are missing the weight of the existing string is used instead of the minimum.

$d_{0,0}= 0$

$d_{i,0} = w_i + d_{i-1, 0}  \quad i \in (1,m)$

$d_{0, j} = w_j + d_{0, j-1} \quad j \in (1,n)$ 

Given the first row, column and the cost function the calculation of the rest of the matrix is similar to the original algorithm.
In case no change is necessary the cost will be zero, otherwise the cost will be calculated as shown above.
The same cost is used for deletion or insertion as the distance between the surrounding cells is de defined as equal, just like in the original algorithm.

If the weight function is monotone decreasing then the weights of the last characters would be less than the ones in the beginning.
Such function can be the $f_w(x) = 0.9 \cdot x$.

If the weight function is monotone increasing then the weights of the first characters would be the least.
Such function would be the $f_w(x) = x + 1$. 

This algorithm has a limitation that the weight of the last characters can't be set to decrease by making addition or subtraction operations, 
as at some point the weight would become negative. 

## Inverse Weighted Levenshtein distance

As the original purpose of the modification is to decrease the weights for the last characters a slight modification was made to the algorithm.
Although the purpose can be fulfilled by having a monotone decreasing weight function which does not fall below 0, the general algorithm can 
be easily modified to support the addition operation by taking the inverse of the weights at a given position.
The only condition is that the weight vector should not equal to zero.
This change  enables smoother weight functions. The weight vectors are calculated the same way as before, 
the calculation of the $D$ matrix is changed as the following: 

$d_{0,0}= 0$

$d_{i,0} = {\frac {1}{w_i} } + d_{i-1, 0}  \quad i \in (1,m)$

$d_{0, j} = {\frac {1}{w_j} } + d_{0, j-1} \quad j \in (1,n)$

The decision functions are swapped for this case.

$c_{ij} = 
\cases{
\text{min} ( w_{i} , w_{j} ) & $f_w \text{ monotone increasing}$ \cr
\text{max} ( w_{i} , w_{j} ) & $f_w \text{ monotone decreasing}$
} 
$
