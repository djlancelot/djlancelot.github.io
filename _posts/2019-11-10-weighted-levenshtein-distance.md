---
layout: post
title:  "Weighted Levenshtein distance"
date:   2019-11-10 23:43:06 -0800
categories: datascience 
---
In this post I'll introduce a two new variants for the Levenshtein distance calculation, specifically for the Wagner–Fischer algorithm to dynamically change the cost of the edit step based on the position of the changes.

The Wagner–Fischer algorithm calculates the edit disance between two strings. It is a dynamic programming algorithm that uses an m by n matrix to calculate the edit distance in between two words $w_{1}$ and $w_{2}$

<script type="text/javascript" id="MathJax-script" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.0.0/es5/tex-mml-chtml.js">
</script>