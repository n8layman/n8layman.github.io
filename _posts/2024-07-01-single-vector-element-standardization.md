---  
layout: post  
title: Single Vector Element Standardization  
subtitle: Finding harmony within oneself  
image: /assets/img/inner-harmony.png  
tags: [data, R, tricks]  
---  

Standardizing data vectors is an important step in data science. For
example, different data sources often represent the same entity, such as
country names, in different ways or with different spellings. In
addition, small errors or formatting inconsistencies can complicate
joins and lead to difficulties in downstream analysis. While excellent
resources do exist to solve this problem for individual use cases, such
as the [countries](https://github.com/fbellelli/countries) package in R,
it can be useful to understand general methods to self-standardize
categorical vectors. There are three approaches that I’ve used to
standardize character vectors. I’ve written scripts below to illustrate
each method.

## The vector to standardize

Note: this data is fictional!

| disease_name       | outbreaks |
|:-------------------|----------:|
| Influenza          |        15 |
| Inflenza           |        10 |
| COVID-19           |        95 |
| sars-covid-19      |        90 |
| Malaria            |        20 |
| Maleria            |        22 |
| malaria            |        20 |
| Diabetes           |        30 |
| Diabetis           |        28 |
| HIV/AIDS           |        75 |
| HIV                |        70 |
| AIDS               |        65 |
| Tuberculosis       |        40 |
| Tuberclosis        |        38 |
| Alzheimers         |        23 |
| Alzheimers Disease |        27 |
| Heart Disease      |        60 |
| Heart Diease       |        58 |

1.  Standardization by string distance
2.  Clustering
3.  Natural Language Processing (NLP)