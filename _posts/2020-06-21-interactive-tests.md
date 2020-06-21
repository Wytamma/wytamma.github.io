---
title: "Interactive diagnostic test statistics"
date: 2020-06-21T00:00:01-00:00
excerpt_separator: "<!--more-->"
header:
  image: assets/images/interactive-tests.png
categories:
  - blog
  - project
tags:
  - p5js
  - diagnostics
  - learning
  - webapp
---
I used [P5.js](https://p5js.org/) to make an interactive application to understand diagnostic test statistics. I've been watching a lot of [coding train](https://www.youtube.com/channel/UCvjgXvBlbQiydffZU7m1_aw), and it has got me keen to do something with P5.js. My housemate has been learning about diagnostics as part of her veterinary degree, so I decided to combine the two and make an interactive app for understanding diagnostics.
<!--more-->

<iframe width="600" height="405" src="https://wytamma.github.io/interactive-diagnostic-test-statistics/index.html" frameborder="0" allowfullscreen></iframe>

Red circles are diseased individuals, and the plus indicates a positive test result. Full code can be found on [github](https://github.com/Wytamma/interactive-diagnostic-test-statistics).

## Diagnostics 
Sensitivity and specificity are statistical measures of the performance of a binary classification test. Sensitivity and specificity are intrinsic measures of performance and described how the results of a test compare to reality. The portion of results that are correctly identified as positives and negative are the sensitivity (True Positive Rate) and specificity (False Positive Rate), respectively. One of the critical concepts of diagnostics is positive (PPV) and negative (NPV) predictive values. These values describe how believable the results of a test are. Unlike sensitivity and specificity, PPV and NPV change with prevalence. 

A test that has low sensitivity (i.e. incorrectly identifies positive individuals as negatives) will have a higher NPV (i.e. you can be confident in the negative results) when the prevalence is low, as the majority of individuals are negative anyway. When testing a high prevalence population with a low sensitivity assay, the NPV is low because the low sensitivity test will miss a higher number of positives. The opposite is true for specificity and PPV, i.e. when specificity and prevalence are low so is PPV, as a low specificity incorrectly identifies negative as individuals positive it is hard to believe a positive result when there are few true positives in the population. Try playing with the sliders in the app above to get a feeling of how this works.

This [wiki page](https://en.wikipedia.org/wiki/Sensitivity_and_specificity#Confusion_matrix) has more information on how these values are calculated and relate.

## P5.js
> p5.js is a JavaScript library for creative coding, with a focus on making coding accessible and inclusive for artists, designers, educators, beginners, and anyone else! p5.js is free and open-source because we believe software, and the tools to learn it, should be accessible to everyone. <cite><a href="https://p5js.org/">P5js website</a></cite>

This was my first time using P5.js, but I really like. It's a simple yet powerful visualisation library. There's a free web editor so you can get started right away. I wrote the above application in the web editor by forking [this project of bouncing balls](https://editor.p5js.org/cdaein/sketches/HJdF8TL6-). 