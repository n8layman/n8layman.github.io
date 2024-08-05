---
layout: post
title: What is Machine Learning?
image: /assets/img/overfit.svg
cover-img: /assets/img/overfit.svg
share-img: /assets/img/overfit.svg
tags: [machine learning]
---

While I was visiting with my aunt the other day she asked me, “what is
machine learning?” I was caught off guard because the term is so common
place in my industry that I haven’t really ever had to sit down and
define it from first principles before. I normally think about Machine
Learning (ML) in terms of model tuning and the variance-bias trade-off,
neither of which was of any use when talking with my aunt. Perhaps my
favorite explanation is from [xkcd](https://xkcd.com/1838/):

<p style="text-align:center;">
<img src="https://imgs.xkcd.com/comics/machine_learning_2x.png" width="40%">
</p>

## The variance-bias trade-off

Still the bias-variance trade-off is a good place to start. It’s an
important concept that describes the balance between two sources of
error that affect model performance: bias and variance. Bias refers to
the error introduced by simplifying assumptions made by the model,
leading to systematic inaccuracies or underfitting. Variance, on the
other hand, represents the model’s sensitivity to fluctuations in the
training data, causing overfitting.

<p style="text-align:center;">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/9f/Bias_and_variance_contributing_to_total_error.svg/1920px-Bias_and_variance_contributing_to_total_error.svg.png" width="50%">
</p>

An optimal model minimizes the total error by striking a balance between
bias and variance. Too much bias results in an overly simple model that
misses relevant patterns, while too much variance creates an overly
complex model that captures noise rather than the underlying data
structure. Finding this balance is important for both classification and
regression based problems.

<p style="text-align:center;">
<img src="https://365datascience.com/resources/blog/x671k7dla1f-overfitting-vs-underfitting-classification-examples.png" width="100%">
</p>

Unfortunately the ‘sweet-spot’ between bias and variances **depends on
how much data you have.**

But how do we control where we land on the variance-bias trade-off? In
short, by changing the flexibility of the model. This is often done by
tuning the levers, switches, and dials in our model fitting method to
ramp up or ramp down model complexity in different way hunting for the
perfect balance between bias and variance. For tree-based methods like
boosted regression trees this could, in example, involve controlling the
maximum tree depth. The way we identify the optimal tree depth is by
holding back a part of our data to tune against.

## The train-validate-test split

Using a train-validate-test split ensures that models generalize well to
new data. By splitting the data into three sets, we can effectively
train the model on the training set, tune hyper-parameters and make
decisions on model performance using the validation set, and finally,
evaluate the model’s performance on a hold-out test set. The splits look
like this in tidymodels:

<p style="text-align:center;">
<img src="https://www.tidymodels.org/start/case-study/img/validation-split.svg" width="50%">
</p>

A downside is that we set aside parts of the data twice, reducing the
amount of data available for training. Techniques like Cross-Validation
help to avoid reducing the amount of data too much by allowing all data
points to be used for both training and validation. Instead of splitting
off a single validation set, cross-validation partitions the data into
multiple subsets and trains the model on different combinations, each
time validating on the remaining subset. This method ensures that every
data point is utilized for validation without significantly reducing the
training data size and, as a bonus, prevents outlier datapoints from
unduly influencing hyper-parameter selection.

## My definition

And that’s it. Those are the things that make up Machine Learning. Not
all methods tune hyper-parameters but ultimately I think of Machine
Learning as **any computational process that can improve both it’s
answers *and it’s method* over time or with the addition of new data**.
In the xkcd comic above not only do the answers that come out of the box
become refined (hopefully!) when you pour more data in but the
organization of the pile improves as well. ML is a pile that stirs
itself. And a good sign you’re dealing with an ML based approach is if
you see that train-validate-test split.

But why would you want to use ML anyway? Well, when is it okay to over
fit? Maybe if you’re interested in the noise or you just care about
describing your particular dataset and not predicting outside the range
of your data.

I would love to hear your thoughts! What are your general definitions of
Machine Learning for a non-expert audience? When should you use, or
avoid using, ML methods?

Be well.

## Resources

1.  [Tidymodels example](https://www.tidymodels.org/start/case-study/)
2.  [Coursera Machine Learning
    Course](https://www.coursera.org/learn/machine-learning)
3.  [Kaggle](https://www.kaggle.com/)
4.  [Google Developers Machine Learning Crash
    Course](https://developers.google.com/machine-learning/crash-course)
5.  [Towards data sciece on Medium](https://towardsdatascience.com/)
