---
title: Ranking problems
date: 2022-06-10 09:29:32
tags:
---

Classification vs. Regression: the ultimate dichotomy in machine learning? It's often presented this way, but the mathematical reality -- not to mention real-world applications -- are not so straightforward.

## Which metric?

Ranking problems are a grey area between classification and regression. Suppose you've compiled a dataset where the targets are average review score: maybe one to five stars, or "bad", "below average", "average", "good", "excellent". You have information about relative order, but you don't really care about predicting a specific value. Is the distance between a 3 star and 4 star review the same as between 4 star and 5 star review? You could treat the categories as separate classes but then you lose the notion that a prediction of 4 stars when the correct answer is 5 stars is better than guessing 1 star, since a classifier loss treats 1 and 4 as equally wrong.

The confusion is increased because we are taught to regard certain metrics as "regression metrics" and others as "classification metrics." ROCAUC is a classification metric, right? Yes, but remember to calculate the ROCAUC, you must first rank all your predictions so that you can check the true positive rate against the false positive rate at different thresholds. So the ranking concept is baked into the most popular classification metric.

On the other hand, if you train a regression model, you could still calculate the ROCAUC. Just threshold your targets into bins: say 4-5 stars is the positive class, and 1-3 stars is the negative class. Since the predictions from a regression model can obviously be ranked, you have everything you need to compute ROCAUC.

## Practical considerations

Say you're a wine supplier, and you've already determined that high reviews in a particular magazine increase demand for a particular vintage. Therefore it would be great if you could predict the review score ahead of time, so you could buy before the demand increases.  Let's say the scores range from 0-5, and the distribution looks like this:

<figure>
  <img src="/images/wine_quality_hist.svg" alt="Distribution of wine quality scores."/>
  <figcaption style="text-align: center"><em>Distribution of wine reviews, with 5 being the best.  We are interested in wines with scores >= 4, shaded here in orange.</em></figcaption>
</figure>

Only wines in the 4-5 range really get the demand bump. You don't really care about being able to distinguish between 0 and 1 or 0 and 2, but the distinction between 3 and 4 is important. How could you model the quality?

* As a multiclass classification problem, with each quality score 0-5 being a different class,
* As a regression problem, learning the predict the numerical quality score directly,
* As a binary classification problem, with qualities 0-3 being the negative class, and qualities 4-5 being the positive class, or
* A ranking problem. But how? 
* 
<figure>
  <img src="/images/wine_model_comparison.svg" alt="Distribution of wine quality scores."/>
  <figcaption style="text-align: center"><em>Distribution of wine reviews, with 5 being the best.  We are interested in wines with scores >= 4, shaded here in orange.</em></figcaption>
</figure>
