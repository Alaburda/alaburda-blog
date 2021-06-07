---
title: How to Chi-squared test
author: Paulius Alaburda
date: '2020-05-16'
slug: []
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2020-05-16T15:00:12+03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

As I am working through [Statistical Rethinking](https://xcelab.net/rm/statistical-rethinking/) and picking up new tools, I decided to rework my knowledge of null hypothesis testing. This is definitely not the worst tutorial on the Chi-squared test but this is more of a write up to wrap my head around it. I think this will of interest[^1] for first year undergraduates or students interested in competing in biology olympiads.

[^1]: Not necessarily interesting, haha

## Premise of the Chi-squared test 

Kearl Pearson created the test in 1900 to answer the question whether two or more categorical variables are associated with one another. It is built on the idea that the sum of squares of *independent* normal distributions is distributed as a Chi-squared distribution. If the distributions were *dependent*, then their sum of squares would *not* follow a Chi-squared distribution.

Now, we usually cannot know the whole distribution and instead we collect samples via surveys and experiments (as in, we usually don't know the exact gender, political affiliation or age breakdown of the whole population we are studying). However, we do know what a Chi-squared distribution should like under the null hypothesis of there being no association. So by using the test we get a single chi-squared value and check where that value falls in the Chi-squared distribution. Through that, we can calculate the probability of getting a value equal to or greater than the value we calculated, i.e. the p value.

## Step by step of the Chi-squared test

That sounds a little too much for me and probably for a lot of students out there as well. What usually happens is that a workflow is taught instead: gather categorical data, shove that into the Chi-squared test formula and stick your Chi-squared value and your p value into the paper[^2].

[^2]: This would make for a great "HAHA CHI-SQUARED TEST GOES BRRR" meme.

In my case, I was taught how to perform the Chi-squared test by hand. That's actually great because it lets us breakdown the Chi-squared test formula[^3]:

[^3]: I am leaving out some nuance here by using the goodness of fit formula when I could also show the Test for Independence formula which looks like this: `\(X^2=\sum\limits_{i=1}^I \sum\limits_{j=1}^J\dfrac{(O_{ij}-E_{ij})^2}{E_{ij}}\)` Although it is more correct, it also takes more time to read and understand for people not used to indices.

`$$\tilde{\chi}^2=\sum_{k=1}^{n} \frac{(O_k - E_k)^2}{E_k}$$`

Here `\(O_k\)` is the observed count of a category, `\(E_k\)` is the expected count.

Let's say you are given a contingency table of two variables (in this case, the number of cylinders and the number of gears from the `mtcars` dataset).

1) First, create a contingency table. This shows our observed values `\(O_k\)` for each combination of gear and cylinder:


```r
library(tidyverse)

observed <- table(mtcars$cyl, mtcars$gear)

observed %>% knitr::kable()
```



|   |  3|  4|  5|
|:--|--:|--:|--:|
|4  |  1|  8|  2|
|6  |  2|  4|  1|
|8  | 12|  0|  2|

2) Then, find the expected values `\(E_k\)` as though there was no association between the two. If you were forced to do this with pen and paper, you would have to calculate the proportion of each type of gear and each type of cylinder. Using R, here is the proportion of each type of cylinder:


```r
cyl_summary <- mtcars %>% count(cyl) %>% mutate(Proportion = prop.table(n))

cyl_summary %>% knitr::kable()
```



| cyl|  n| Proportion|
|---:|--:|----------:|
|   4| 11|    0.34375|
|   6|  7|    0.21875|
|   8| 14|    0.43750|

And each type of gear:


```r
gear_summary <- mtcars %>% count(gear) %>% mutate(Proportion = prop.table(n)) 

gear_summary %>% knitr::kable()
```



| gear|  n| Proportion|
|----:|--:|----------:|
|    3| 15|    0.46875|
|    4| 12|    0.37500|
|    5|  5|    0.15625|

If there was no association between gear and cylinder, we would expect that the probability of a 4 cylinder car with 3 gears would be the probability of a 4 cylinder car times the probability of 3 gear car. You can have `R` this for you for each combination:


```r
tbl_prop <- cyl_summary$Proportion %o% gear_summary$Proportion
rownames(tbl_prop) <- c(4,6,8)
colnames(tbl_prop) <- c(3,4,5)

tbl_prop %>% knitr::kable()
```



|   |         3|         4|         5|
|:--|---------:|---------:|---------:|
|4  | 0.1611328| 0.1289062| 0.0537109|
|6  | 0.1025391| 0.0820312| 0.0341797|
|8  | 0.2050781| 0.1640625| 0.0683594|

To get the expected values `\(E_k\)`, we can multiply the probabilities by the number of total observations (in this case, 32):


```r
expected <- tbl_prop*sum(observed)

expected %>% knitr::kable()
```



|   |       3|     4|       5|
|:--|-------:|-----:|-------:|
|4  | 5.15625| 4.125| 1.71875|
|6  | 3.28125| 2.625| 1.09375|
|8  | 6.56250| 5.250| 2.18750|

3) Then calculate the difference between the observed and the expected values. This gives you a table of differences between `\(O_k\)` and `\(E_k\)`: 


```r
(observed-expected) %>% knitr::kable()
```



|   |        3|      4|        5|
|:--|--------:|------:|--------:|
|4  | -4.15625|  3.875|  0.28125|
|6  | -1.28125|  1.375| -0.09375|
|8  |  5.43750| -5.250| -0.18750|

For example, we did not observe any cars with 8 cylinders and 4 gears but based on the how frequent 8 cylinder cars and 4 gear cars are, we expected to see 5.25 cars.

Now what's interesting is that the sum of all those differences will always sum to 0. That is inconvenient, so what Pearson did in 1900 is square the differences, giving us this:


```r
(observed-expected)^2 %>% knitr::kable()
```



|   |         3|         4|         5|
|:--|---------:|---------:|---------:|
|4  | 17.274414| 15.015625| 0.0791016|
|6  |  1.641602|  1.890625| 0.0087891|
|8  | 29.566406| 27.562500| 0.0351562|

4) Divide by the expected value - without this step `\((O_k-E_k)^2\)` is just proportional to the number of counts, so comparing, for example, a sample of a 1000 counts and 100 counts would be impossible. That is why you need to normalise the values by dividing by the expected value[^4]. 

[^4]: A really good technical explanation can be found [here](https://math.stackexchange.com/questions/2074029/why-do-we-divide-by-the-expected-value-in-the-chi-squared-test). Essentially `\(O_k\)` is a count therefore it can be modeled as a Poisson distribution ($O_k ∼ Poisson(E_k,E_k)$) and with high enough counts the distribution becomes similar enough to the normal distribution with mean and variance `\(E_k\)`.


```r
((observed-expected)^2/expected) %>% knitr::kable()
```



|   |         3|         4|         5|
|:--|---------:|---------:|---------:|
|4  | 3.3501894| 3.6401515| 0.0460227|
|6  | 0.5002976| 0.7202381| 0.0080357|
|8  | 4.5053571| 5.2500000| 0.0160714|

5) Sum the resulting values:


```r
chisq_rs <- sum((observed-expected)^2/expected)
print(chisq_rs)
```

```
## [1] 18.03636
```

6) Using a reference table (for example, [this one](https://people.smp.uq.edu.au/YoniNazarathy/stat_models_B_course_spring_07/distributions/chisqtab.pdf) one), find the critical p value (in this case, for df = 2). You do that by finding the largest Chi-squared value that is smaller than the one we calculated (in this case, smaller than 18.04). In this case, 18.04 for df = 2 is larger than the critical value of 9.21 so we can say that the p value is less than 0.01.

To be exact, let's see where this value falls if we were to build a Chi-squared distribution with df = 2:


```r
chisq_sample <- rchisq(10000, 2)

qplot(chisq_sample) + geom_vline(xintercept = chisq_rs, color = "red") + theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

```r
chisq_ecdf <- ecdf(chisq_sample)
chisq_percentile <- chisq_ecdf(chisq_rs)
```

Or, if we built a distribution function out of our samples, our value would fall into the 0.9998 percentile. Since that is definitely larger than 0.95, we can conclude that there is an association between the number of gears and the number of cylinders. Alternatively, this is what the Chi-squared test would have returned:


```r
rs <- chisq.test(observed)
```

```
## Warning in chisq.test(observed): Chi-squared approximation may be incorrect
```

```r
rs
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  observed
## X-squared = 18.036, df = 4, p-value = 0.001214
```

Which we could have reported as such (while also describing the count data):

> A chi-square test was conducted whether there is an association between the number of gears and the number of cylinders across different cars. The results were significant (χ^2^(4) = 18.04, p = 0.001)

## But why does it work?

I'll be honest, it takes me a while to grok distributions beyond the Binomial and the Gaussian. A good explanation how the Chi-squared distribution is derived can be found [here](https://www.rdatagen.net/post/a-little-intuition-and-simulation-behind-the-chi-square-test-of-independence/). 

The chi-square ($\chi^2$) distribution, as we have mentioned earlier, is a distribution of the sum of squares of two or more normal distributions. In other words, if you would square values from one normal distribution and add it to another, the result should approximate the Chi-square distribution:

`$$U∼N(0,1)$$`
`$$V∼N(0,1)$$`
`$$U^2+V^2∼\chi_2^2$$`

That subscript denotes that the sum of squares follows a `\(\chi^2\)` distribution with 2 degrees of freedom. Alternatively, a square of just one normal distribution follows a `\(\chi_1^2\)` distribution and for any number of degrees:

\[\sum_{j=1}^{k} \chi_j^2∼\chi_k^2\]

Let's plot these distributions!


```r
df <- tibble(`Normal` = rnorm(10000, mean = 0, sd = 1),
             `Normal squared` = Normal^2,
             `Actual Chi-squared` = rchisq(10000,2),
             `Chi-squared with df = 2` = `Normal squared` + rnorm(10000, mean = 0, sd = 1)^2) %>%
  gather(key = "distribution", value = "value") %>% 
  mutate(distribution = factor(distribution, levels = c("Normal","Normal squared","Chi-squared with df = 2","Actual Chi-squared")))

ggplot(df, aes(x = value, fill = distribution)) + 
  geom_histogram() + 
  facet_grid(~distribution, scales = "free_x") + 
  guides(fill = FALSE) + 
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="672" />

So... Now what? How does any of this relate back to the Chi-squared test formula? Here's where the central limit theorem comes in: even though any count data (whether it's counting blood cells, people, adverse events) is distributed along a poisson distribution, *the sampling distribution* (as in, the result of collecting samples a lot of times) is similar to the normal distribution.

To understand this, I've simulated two Poisson distributions (as in, count data). If we tried to build a Chi-squared distribution out of this data, what percentage of values would be higher than the 95th percentile of an actual Chi-squared distribution? 


```r
df <- tibble(
  pois_1 = rpois(10000, 10),
  pois_2 = rpois(10000, 10),
  chisq_real = rchisq(10000, df = 2)
  ) %>% 
  mutate(pois_1 = (pois_1-mean(pois_1))/sd(pois_1),
         pois_2 = (pois_2-mean(pois_2))/sd(pois_2)) %>% 
  mutate(chisq_11 = pois_1^2+pois_1^2,
         chisq_12 = pois_1^2+pois_2^2) %>% 
  select(contains("chisq_")) %>% 
  gather(key = "distribution", value = "value") %>%
  mutate(over_95 = value > qchisq(.95, df=2))

ggplot(df, aes(x = value, fill = over_95)) + 
  geom_histogram(alpha = 0.5, binwidth = 1) + 
  facet_grid(~distribution) + 
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="672" />

`chisq_12` is built via two independent Poisson distributions, while `chisq_11` is just to identical samples added together (making the distributions dependent). From the chart it seems like it works and if you look at the percentage of values higher than the 95th percentile:


```r
df %>% 
  group_by(distribution) %>% 
  summarise(over_95 = mean(over_95)) %>% 
  knitr::kable()
```



|distribution | over_95|
|:------------|-------:|
|chisq_11     |  0.0774|
|chisq_12     |  0.0512|
|chisq_real   |  0.0517|

So `chisq_12` and `chisq_real` makes sense - around 5 per cent of values are going to be higher than the 95th percentile. For `chisq_11`, we see that is not the case and the percentage is higher!

We can see that the Chi-squared distribution requires that the added distributions be independent. The final kicker comes in when we look back at the Chi-squared test formula:

`$$\tilde{\chi}^2=\sum_{k=1}^{n} \frac{(O_k - E_k)^2}{E_k}$$`
If the difference between observed and expected values is small, then `\(\chi^2\)` is small. If the difference is large, then `\(\chi^2\)` is large. The observed values follow a normal distribution and the expected values also follow a normal distribution, therefore the squared difference can be modeled as a Chi-squared distribution. And since it can be modeled it can be turned into a probability of observing this or more extreme[^5] data!

[^5]: I am putting this clunky wording here just to be correct because the probability of observing an exact Chi-squared test value is always zero since the distribution is continuous.

To me this makes enough sense to trust it. In summary, Pearson created the formula that returns a value. This particular value increases or decreases based on the difference of observed and expected count data. The test statistic by itself does not tell us much, but we know that the formula should generate values that are distributed along a chi-squared distribution and we can translate the value into a probability by checking where Chi-squared test values fall in the Chi-squared test distribution. 

Even though the test is pretty simple, it packs a lot of assumptions and nuance. For a really good explanation, I would recommend checking out Danielle Navarro's book [Learning with R](https://learningstatisticswithr.com/lsr-0.6.pdf).


