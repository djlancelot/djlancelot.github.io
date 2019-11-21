---
layout: post
title:  "Weighted Levenshtein distance"
date:   2019-11-10 23:43:06 -0800
categories: datascience 
---
In this post, I'll introduce two new variants for the [Damerau–Levenshtein distance](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance) calculation &mdash; specifically for an extended version of the Wagner–Fischer algorithm &mdash; to dynamically change the cost of the edit step based on the position of the changes.

## Summary of the Damerau-Levenshtein distance

The [Wagner–Fischer algorithm](https://en.wikipedia.org/wiki/Wagner%E2%80%93Fischer_algorithm) calculates the edit distance between two strings. It is a dynamic programming algorithm that uses an m by n matrix to calculate the edit distance between two words $w_{1}$ and $w_{2}$. For the original Levenshtein distance there are 3 kinds of operations available for two characters in the string:
- **Deletion** (Transformation from main to man by removing the *i* character)
- **Insertion** (Transformation from man to main by inserting the *i* character) 
- **Substitution** (Transformation from main to gain by modifying the leading *m* character to *g*)

The Damerau–Levenshtein variant introduces one more operation, which is the **transposition** of the characters. 
The changes presented in this post were implemented on the Damerau–Levenshtein variant, but can be implemented for the original [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) metric by removing the additional operation from the code.

The Damerau–Levenshtein distance provides the edit distance between two strings based on the 4 operations mentioned above. 
For example, the edit distance between the words *Martha* and *Marha* is 1, because with the removal of the *t* character (or the addition of it) the other string can be generated. The edit distance for *Main* and *Gain* is also 1 as the *M* can be substituted by a *G* or vice versa to generate one word from the other. This metric is used for correcting typing errors in texts. The typed word is matched against a vocabulary and the word with the lowest Levenshtein distance is suggested as a correction for the word.

The distance can be normalized between 0 and 1 by dividing the distance by the length of the longest character. In the $D$ m by n matrix determined by the Wagner-Fischer algorithm this value is 

$norm = 
\cases{
    d_{m,0} & $\text{if } m \ge n$ \cr
    d_{0,n} & $\text{if } m \lt n$ 
}
$

In some use cases, like similar company name detection, the end of the string is less important than the beginning. 
Especially in company names, some article mentions using the business type abbreviation or other suffixes with the name while others omit the abbreviation or write alternatives. 
For example variants like *Lucky Ltd*, *Lucky Limited* or *Lucky* might still refer to the same company. 
Other possible use-cases where varying weight would be advantageous are street addresses and news titles.
In street addresses using the abbreviation or the full street type is less important than the house number and the street name itself.
Some news sources make minor changes to the original article and re-publish them on their own site. 
Using a weighted distance can be beneficial when trying to deduplicate them.

  
In this article 2 algorithms will be shown to solve this problem and add different weights depending on the position of the changed characters.

## Jaro-Winkler distance, the alternative

The [Jaro-Winkler distance](https://en.wikipedia.org/wiki/Jaro%E2%80%93Winkler_distance) can be used for cases when the beginning of the string has higher importance than the end. 
The algorithm is based on the Jaro similarity and can boost the similarity based on a matching prefix of up to 4 characters.
If there is no matching prefix or the matching prefix is longer than 4 characters, 
the distance will stay the same as the Jaro similarity, which does not use any weights based on the position.
The proposed algorithms overcome these issues too.

## Weighted Generalised Levenshtein distance

The generalized algorithm introduces three new input parameters to the algorithm: 
- an initial value
- a weight function
- a decision function

Based on the first two parameters a weight vector is created ( $w_{n}$ ) with the size of the longer string.
This weight vector will be shared with the other string.

$w_0 = w_{init}$

$w_{i \in n} = f_w(w_{i-1})$

In the original algorithm, the cost of each modification is 1. In the proposed algorithm instead of the constant value, 
one of the weights is used at the position.
- The minimum of the two weights should be picked in case of a monotonically decreasing weight function
- The maximum of the two weights should be picked in case of a monotonically increasing weight function  

$c_{ij} = 
\cases{
\text{min} ( w_{i} , w_{j} ) & $f_w \text{ monotonically decreasing}$ \cr
\text{max} ( w_{i} , w_{j} ) & $f_w \text{ monotonically increasing}$
} 
$

Using the above decision functions will make sure that the algorithm can be executed in a greedy fashion, 
the cost of making one change at two positions will be the sum of the cost of a change at one of the positions and 
the cost of the change for the other position.

The first column and first row show the cost of changing the string represented by the row or the column into a 0 length string (no string at all).
This is a cumulative sum of the step cost over the length of the string. 
As this case covers the possible outcome that one of the strings is missing the weight of the existing string is used instead of the minimum.

$d_{0,0}= 0$

$d_{i,0} = w_i + d_{i-1, 0}  \quad i \in (1,m)$

$d_{0, j} = w_j + d_{0, j-1} \quad j \in (1,n)$ 

Given the first row, column and the cost function the calculation of the rest of the matrix is similar to the original algorithm.
In case no change is necessary the cost will be zero, otherwise, the cost will be calculated as shown above.
The same cost is used for deletion or insertion as the distance between the surrounding cells is de defined as equal, just like in the original algorithm.

If the weight function is monotonically decreasing then the weights of the last characters would be less than the ones in the beginning.
Such function can be the $f_w(x) = 0.9 \cdot x$.

If the weight function is monotonically increasing then the weights of the first characters would be the least.
Such function would be the $f_w(x) = x + 1$. 

Mixing up the decision function or using a function that changes its direction would
cause problems, like 
- incorrect normalization: values other than the first row or first column can become the maximum of the matrix
- inconsistent distances: the distance from "the" -> "ehe" and "the" -> "tre", would not equal to "the" -> "ere".

This algorithm has a limitation that the weight of the last characters can't be set to decrease by making addition or subtraction operations, 
as at some point the weight would become negative. 

A sample implementation of the algorithm is the following:

```python
import numpy as np
def wlevenshtein(s1, s2, normalize=False, w_init=1, w_func=lambda x: x, d_func=min):    
    l1, l2 = len(s1)+1,len(s2)+1
    ml = max(l1, l2)
    wv = np.zeros(ml)
    w = w_init
    for i in range(1,ml):
        wv[i] = w
        w = w_func(w)
    d = np.zeros((l1, l2))
    cost = 0
    norm = 1.0
    for i in range(1,l1):
        d[i, 0] = (wv[i]) + d[i-1, 0]
    for j in range(1,l2):
        d[0, j] = (wv[j]) + d[0, j-1]
    for i in range(1,l1):
        for j in range(1,l2):
            step_cost = d_func(wv[i],wv[j])
            cost = 0 if s1[i-1]==s2[j-1] else step_cost
            d[i,j]= min(
                d[i-1, j]+step_cost,
                d[i, j-1] + step_cost,
                d[i-1, j-1] + cost
            )
            if i > 1 and j > 1 and s1[i-1] == s2[j-2] and s1[i-2] == s2[j-1]:
                d[i, j]=min(d[i, j], d[i-2, j-2] + cost)
    if normalize:
        norm = d[l1 - 1, 0] if l1 > l2 else d[0, l2 - 1]
    return d[l1-1, l2-1]/norm
```

## Inverse Weighted Levenshtein distance

As the original purpose of the modification is to decrease the weights for the last characters a slight modification was made to the algorithm.
Although the purpose can be fulfilled by having a monotonically decreasing weight function which does not fall below 0, the general algorithm can 
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
\text{min} ( w_{i} , w_{j} ) & $f_w \text{ monotonically increasing}$ \cr
\text{max} ( w_{i} , w_{j} ) & $f_w \text{ monotonically decreasing}$
} 
$

Choosing the right decision function is equally important in this case too.

A sample implementation of the algorithm is the following:

```python
import numpy as np
def iwlevenshtein(s1, s2, normalize=False, w_init=1, w_func=lambda x: x, d_func=min):
    l1, l2 = len(s1)+1,len(s2)+1
    ml = max(l1,l2)
    wv = np.zeros(ml)
    w = w_init
    for i in range(1,ml):
        wv[i] = w
        w = w_func(w)
    d = np.zeros((l1, l2))
    cost = 0
    norm = 1.0
    for i in range(1, l1):
        d[i, 0]= (1/wv[i]) + d[i-1, 0]
    for j in range(1, l2):
        d[0, j]= (1/wv[j]) + d[0, j-1]
    for i in range(1, l1):
        for j in range(1, l2):
            step_cost = d_func(1/wv[i], 1/wv[j])
            cost = 0 if s1[i-1]==s2[j-1] else step_cost
            d[i, j]= min(
                d[i-1, j]+step_cost,
                d[i, j-1] + step_cost,
                d[i-1, j-1] + cost
            )
            if i > 1 and j > 1 and s1[i-1] == s2[j-2] and s1[i-2] == s2[j-1]:
                d[i, j]= min(d[i, j], d[i-2, j-2] + cost)
    if normalize:
        norm = d[l1 - 1, 0] if l1 > l2 else d[0, l2 - 1]
    return d[l1-1,l2-1]/norm
```


## Conclusion

Using the previously introduced algorithms two strings edit distance can be calculated in a way that the weight of each change 
varies depending on the position of the changed characters. This can be useful for comparing company names, postal addresses or news titles 
where a match at the start of the string is more important than the match at the end. 
Unlike the Jaro-Winkler distance, this calculation does not need a matching prefix of the two strings.
The inverse weighted distance is more convenient to use with decreasing weights.
