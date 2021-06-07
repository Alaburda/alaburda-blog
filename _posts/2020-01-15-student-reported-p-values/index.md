---
title: Student reported p values
author: Paulius Alaburda
date: '2020-01-15'
slug: student-reported-p-values
categories:
  - r
  - statistics
  - education
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2020-05-16T21:25:33+03:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

I have been delaying this post for 2 years but I am glad to finally publish it. This post is an example of incentives driving the students to publish more abstracts with more p values in them and I think the charts tell pretty much the whole story.


```r
knitr::opts_chunk$set(echo = FALSE, warning = FALSE, message = FALSE)

library(tidyverse)
library(lme4)
library(stats4)
library(knitr)

abstract_data <- read_csv("https://raw.githubusercontent.com/Alaburda/thesisExtractr/master/abstract_data.csv")

pvalue_data <- read_csv("https://raw.githubusercontent.com/Alaburda/thesisExtractr/master/p_vals.csv")


# Additional wrangling
## Create rounded variable
pvalue_data$rounded <- 0

## Being rounded and being truncated is mutually exclusive, so take untruncated values and see if they are rounded
pvalue_data$rounded[pvalue_data$truncated == 0] <- ifelse(round(pvalue_data$pval[pvalue_data$truncated == 0],2) == pvalue_data$pval[pvalue_data$truncated == 0],1,0)

# In cases such as "p < 0.00", the p value is reported incorrectly. The p value in that case is actually rounded and not truncated (even though it is reported in a truncated manner).
pvalue_data[pvalue_data$truncated == 1 & pvalue_data$pval == 0, ] <- 
pvalue_data %>% filter(truncated == 1, pval == 0) %>% mutate(truncated = 0,
                                                             rounded = 1)

# Mean imputation for p values?

# Live to check papers with confidence intervals but no p values
#anti_join(abstract_data,pvalue_data,by = c("ID","Year")) %>% filter(grepl("95%",Results)) %>% select(-Methods) %>%  View()
```

In 2014, Jeff Leek and Leah R. Jager published a [paper](https://academic.oup.com/biostatistics/article/15/1/1/244509) about the rate of false positive findings in research papers. The idea was to scrape multiple journal paper abstracts and estimate the number of false positive results. The paper was really interesting to read and inspired me to try to do something similar with a corpus of abstracts from my university. Over here, we have an annual local conference for medical students to submit their research abstracts. Students are incentivised to submit because a printed abstract goes towards increasing your rating when entering residency. Luckily, [the printed abstract books](https://smd.lt/publikacijos/) follow a similar pattern of introduction-methods-results-conclusion throughout the years, giving us an opportunity to show publishing trends over time.

First things first, the code to parse abstracts and extract p values can be found in a separate [repo here] (https://github.com/Alaburda/thesisExtractr). The code may not be perfectly tidy due to differences in abstract books, but should be able to run and recreate the datasets. In principle, it is very similar to what Jeff Leek and Leah R. Jager had done but I had to make a few language-specific additions. First of all, the code parses the abstract book into chunks of text based on the words "Vadovas" (eng. supervisor), "Autoriai" (eng. authors), "Metodai" (eng. Methods) and "Rezultatai" (eng. Results). Separating methods from the results was crucial because most methods mention using p < 0.05 as their threshold for significance, giving us a correct estimate of the number of p values reported in each abstract. 

The results section was parsed for three types of p values. One group are the truncated and significant p values (e.g. p < 0.05). P values reported as p0.05 were also included in this group on the assumption that the special symbol ≤ was somehow lost during the submission process. The second group of p values are truncated but insignificant p values (p > 0.05). Finally, the final group are exact untruncated p values (e.g. p = 0.04).

The final data has two datasets. One is the list of abstracts from each year with its corresponding year, ID, methods and results. The other datasets contains the parsed p values with the corresponding year, ID and information whether the p value is significant, truncated and rounded.

Here's what the data looks like: 


| ID| Year| truncated|   pval| significant| rounded|
|--:|----:|---------:|------:|-----------:|-------:|
|  6| 2004|         0| 0.0002|           1|       0|
|  6| 2004|         0| 0.0007|           1|       0|
|  6| 2004|         0| 0.5100|           0|       1|
|  6| 2004|         0| 0.1370|           0|       0|
|  6| 2004|         0| 0.4600|           0|       1|
|  6| 2004|         0| 0.0070|           1|       0|

## Abstract results

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

The number of abstracts increased drastically in 2012 to a whooping 559 abstracts. The increase persisted until 2016 - this was when an international conference had started being organised. The way the system works is that you are awarded more points for submitting to international conferences. My guess is, students skipped the local conference to present at the international one instead.

## P value results

The plot below shows the percentage of significant p values reported each year:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

The percentage of positive p values increased to around 50 percent and plateaued between 43 and 50 per cent. It's also interesting to note that 2013 through 2015 had the most abstracts, perhaps diluting positive results.

How many abstracts reported p values in the first place though?

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

Now that's beautiful - if I recall correctly from my student years, at some point guidelines started requiring reporting statistical results (in other words, p values) rather than outlining general findings. This would explain the tendency for students to almost uniformly start reporting p values in their papers.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Not only has the number of p values increased, the average abstract also started reporting more p values in an abstract. I am not sure what is going on in 2019 but then again - an average of over 6 p values in an abstract is pretty crazy. We could double check whether this is due to a larger sample of outliers:

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

The distributions are somewhat consistent across the years, suggesting that generally students may have started overreporting p values. Given that, you could suspect that people would report truncated values instead of exact ones.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

However, we see that instead of reporting truncated p values, students have instead started reporting exact values more often. Despite the overreliance on p values, it is great to see that students have adopted a better standard to report them!

Overall, this shows that statistical education in medicine has been getting better but there are still some ways to go. Students rely on p values (and rely on reporting a lot of them) to legitimise their work. Change, however, requires both institutional and student involvement. With conference policy requiring statistical tests in abstracts, I don't see this tendency going away any time soon and I will be sure to give an update once the 2020 conference hits.

## Is it possible to estimate a false discovery rate from p values?

Well, it's complicated and it needs some explaining to do. First of all, if we want to figure out the proportion of false positive results (FDR) from the array of reported p values, we need to know the underlying distribution of p values and we would ideally like to know the effect sizes of the studies. As from this [blogpost](http://daniellakens.blogspot.com/2017/10/science-wise-false-discovery-rate-does.html), as the effect size decreases, the FDR tends to increase with the same p values. So without knowing the effect size reported in the abstracts, we cannot make fair judgments. This problem is further complicated by the fact that effect sizes tend to be larger when the results are significant. This makes sense because for a large effect size to be statistically significant, there is more tolerance for larger errors.

Secondly, what is the underlying distribution of p values we are trying to analyse? We usually expect the distribution of p values to be uniform but students are biased to publish positive results rather than negative results. Furthermore, it is not uncommon to consciously or unconsciously p-hack your way to statistical significance. Remember, there is pressure to ensure the abstracts are accepted.

Because of these reasons, I decided to skip publication bias for now and focus on the exploratory side of things. Maybe in another blog post! Also, if you would like to read more, be sure to read some of the sources below.

Sources:

1. https://academic.oup.com/biostatistics/article/15/1/1/244509
2. https://github.com/jtleek/tidypvals
3. http://varianceexplained.org/statistics/interpreting-pvalue-histogram/
4. http://jtleek.com/genstats_site/lecture_notes/03_12_P-values.pdf
5. https://www.nature.com/news/statistics-p-values-are-just-the-tip-of-the-iceberg-1.17412

