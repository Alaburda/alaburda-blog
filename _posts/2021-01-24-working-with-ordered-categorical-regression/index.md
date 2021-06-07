---
title: Working with Ordered Categorical Regression
author: Paulius Alaburda
date: '2021-01-24'
slug: []
categories:
  - r
tags:
  - R
subtitle: ''
summary: ''
authors: []
lastmod: '2021-01-24T21:13:43+02:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

During my time in the lab, I would usually use linear regression, sometimes logistic regression and, of course, a wide variety of statistical tests. Ordered categorical regression was always something I had heard of but never had the opportunity to apply it. Now that I'm trying to dip into statistics freelancing, the technique is fairly useful. A lot of datasets contain questionnaire data likert scale questions. That sort of data can be interpreted using ordered regression.

## How it works

## Simulating the data


```r
set.seed(1839)
n <- 150
x1 <- rnorm(n)
x2 <- runif(n)

b01 <- 1
b02 <- 0.05
b03 <- -0.05
b04 <- -1
b1 <- 0.8
b2 <- 0.5

logodds1 <- b01 + b1 * x1 + b2 * x2
logodds2 <- b02 + b1 * x1 + b2 * x2
logodds3 <- b03 + b1 * x1 + b2 * x2
logodds4 <- b04 + b1 * x1 + b2 * x2

inv_logit <- function(logit) exp(logit) / (1 + exp(logit))
prob_2to5 <- inv_logit(logodds1)
prob_3to5 <- inv_logit(logodds2)
prob_4to5 <- inv_logit(logodds3)
prob_5 <- inv_logit(logodds4)

prob_1 <- 1 - prob_2to5
prob_2 <- prob_2to5 - prob_3to5
prob_3 <- prob_3to5 - prob_4to5
prob_4 <- prob_4to5 - prob_5

y <- c()
for (i in 1:n) {
  y[i] <- sample(
    x = c(1:5), 
    size = 1, 
    prob = c(prob_1[i], prob_2[i], prob_3[i], prob_4[i], prob_5[i])
  )
}


dat <- data.frame(x1, x2, y = factor(y))
```

