---
layout: post
title:  "Weighted Levenshtein distance"
date:   2019-11-10 23:43:06 -0800
categories: datascience 
---
In this post I'll introduce a two new variants for the Damerau–Levenshtein distance calculation, specifically for an extended version of the Wagner–Fischer algorithm to dynamically change the cost of the edit step based on the position of the changes.

The Wagner–Fischer algorithm calculates the edit disance between two strings. It is a dynamic programming algorithm that uses an m by n matrix to calculate the edit distance in between two words $w_{1}$ and $w_{2}$. For the original Levenshtein distance there are 3 kinds of operations available for two characters in the string:
- **Deletion** (Transformation from main to man by removing the *i* character)
- **Insertion** (Transformation from man to main by inserting the *i* character) 
- **Sustitution** (Transformation from main to gain by modifying the leading *m* character to *g*)

The Damerau–Levenshtein variant introduces one more operation, which is the **transposition** of the characters. The changes presented in this post were implemented on the Damerau–Levenshtein variant, but can be implemented for the original Levenshtein distance metric.