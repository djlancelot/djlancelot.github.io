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

The generalised algorithm introduces two new input parameters to the algorithm: 
- an initial value
- a weight function.

The calculation of the first row and first column of the $D$ matrix is really important, because during the dynamic programming algorithm the rest of the values depend on it.
In the original Levenshtein distance, the calculation 
