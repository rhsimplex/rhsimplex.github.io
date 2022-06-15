---
title: A Neural Network Trick for Ranking Problems 
date: 2022-06-10 09:29:32
tags:
---

Classification vs. Regression: the ultimate dichotomy in machine learning? It's often presented this way, but I've found that many real-world problems are not so clear-cut. Often you want to predict the extreme end of a distribution, or the classes are imbalanced but have some weak relative ordering. In these cases, both traditional classification and regression loss functions can underwhelm. In this post, I want to show you how I've applied the lesser-used [margin ranking loss](https://pytorch.org/docs/stable/generated/torch.nn.MarginRankingLoss.html) to tackle problems which lie between classification and regression. 

All model code and plots are available [in my Kaggle example notebook](https://www.kaggle.com/code/simplex1/a-neural-network-trick-for-ranking-problems/notebook).

## Practical considerations

If you're applying machine learning in a business or scientific context, you can be assured that whatever you are trying to predict will either suffer from extreme class imbalance or lie in some horrible non-normal distribution. Business collaborators in particular will expect a model to be able to pick out outliers, and biological problems offer a deluge of data where far less than 1% of the samples are actually of interest. This is in stark contrast to educational or benchmark datasets, which often have reasonably balanced class distributions or pre-normalized numerical targets.

Let's take a concrete example derived from the [UCI Wine Quality Dataset from Cortez et al](https://archive.ics.uci.edu/ml/datasets/wine+quality). Say you're a wine supplier, and you've already determined that a high review in some wine magazine increases demand for a particular wine. It would be great if you could predict the review score ahead of time from properties of the wine you can measure (like pH and alcohol content), so that you could buy before the demand increases.

Let's say the scores range from 0-5, and the distribution looks like this:

<figure>
  <img src="/images/wine_quality_hist.svg" alt="Distribution of wine quality scores."/>
  <figcaption style="text-align: center"><em>Distribution of wine review scores, with 5 being the best.  We are interested in wines with scores >= 4, shown here with hatching.</em></figcaption>
</figure>

Only wines in the 4-5 range really get the demand bump. Therefore, you don't care about being able to distinguish between 0 and 1 or 0 and 2, but the distinction between 3 and 4 is important. How could you predict the quality? You could use a small neural network, and model the problem as:

* _**Regression**_ This means using an L1 or L2 loss function, for instance. This captures the notion of ordering, since the quality score is learned directly, but treats ranking like a distance. A quality difference between 0 and 1 will be treated just like a quality difference between 3 and 4. However, you actually don't care that much about the difference between 0 and 1 while the difference between 3 and 4 is important.
* _**Multi-class**_ This means using a loss function like cross-entropy loss, and treating each quality score as a distinct class. Easy to implement, but has the distinct disadvantage of ignoring all ranking information.
* _**Single-class**_ This means using a loss function like binary cross-entropy, and thresholding your quality scores into two bins. This implicity keeps a notion of ordering since there are only two classes, but unfortunately throws out a lot of information.
* _**Single-class with ranking**_ As above, but try to capture some of the ordering of the quality targets with margin ranking loss as well.

As you can see in the figure below, capturing the ranking data in a clever way can really help you boost performance. But how do we do it?

<figure>
  <img src="/images/wine_model_comparison.svg" alt="Cross fold validation of various models"/>
  <figcaption style="text-align: center"><em>ROCAUC of the thresholded target for five-fold cross-validation using our various models. Wine quality 0-3 is the negative class, 4-5 is the positive class. Colors represent individual folds. Notice that adding a ranking component significantly improves performance.</em></figcaption>
</figure>

## Margin Ranking Loss

[Margin ranking loss](https://pytorch.org/docs/stable/generated/torch.nn.MarginRankingLoss.html) is one of the more obscure loss functions available in PyTorch.  It's usually used as a [contrastive loss](https://gombru.github.io/2019/04/03/ranking_loss/) for giving structure to an embedding space, but here we're going to use it as a pairwise ranking loss.

Let's start with the definition given in the PyTorch documentation:

{% mathjax %}
\mathrm{loss}\left( x_1, x_2, y\right) = \mathrm{max}\left( 0, -y * (x_1 - x_2) + \mathrm{margin} \right)
{% endmathjax %}

This is already a bit strange-looking considering typical loss functions take just the ground truth {% mathjax %} y {% endmathjax %} and prediction {% mathjax %} x {% endmathjax %}.  In the margin ranking loss formulation, you must take two predictions {% mathjax %} x_1 {% endmathjax %} and {% mathjax %} x_2 {% endmathjax %} and see if they are in the correct order: the ground truth label {% mathjax %} y=1 {% endmathjax %} if {% mathjax %} x_1 > x_2 {% endmathjax %} and -1 otherwise.

Let's ignore the margin term for the moment, setting it to 0 (don't worry, we'll come back to it). You can see that the loss will be zero if {% mathjax %} x_1 {% endmathjax %} and {% mathjax %} x_2 {% endmathjax %} are in the correct order, (meaning {% mathjax %} x_1 > x_2 {% endmathjax %}), and {% mathjax %} x_2 - x_1 {% endmathjax %} if they are in the wrong order.

This seems pretty reasonable, but we still have the problem that the margin ranking loss implementation operates on prediction pairs rather than the predictions (or logits) themselves. Fortunately, with a little ingenuity we can handle this ourselves.

```python
    import torch
    from torch.nn import BCEWithLogitsLoss, MarginRankingLoss
    
    
    class CombinedRankingLoss(BCEWithLogitsLoss):
        """Combines margin ranking loss with binary cross entropy loss
        
        Args:
            margin: the margin parameter for MarginRankingLoss
        """
        def __init__(self, margin=0.0):
            # We are deriving our loss funtion from BCEWithLogitsLoss,
            # so be sure to initialize it
            super().__init__()
            # Initialize the margin ranking loss
            self.mr_loss = MarginRankingLoss(margin=margin)
        
        def __call__(self, outputs, y, y_threshold):
            """Compute the loss for batch size n
            
            Args:
                outputs: the logits or predictions from the model, 
                    should be shape n x 1 or n x 1 x 1
                y: the ground truth targets. In the wine example, 
                    this is an integer quality 0-5
                y_threshold: a binarized target. In the wine example, 
                    this is 1 if quality >= 4, 0 otherwise
            """
            
            # First we have to generate the targets for all pairs in the batch. 
            # We can use the `combinations` function for this. Then it's just
            # a matter of taking the difference and taking the sign: -1 or +1
            y = -torch.combinations(y).diff(dim=1).sign().squeeze()
            
            # Let's do the same for the predictions/logits from the model
            _outputs = torch.combinations(outputs.squeeze(-1))
            
            # We split the columns so we have the right parameters for margin ranking
            x1 = _outputs[:, 0]
            x2 =  _outputs[:, 1]
            
            # Now we have everything in the correct format to
            # compute the margin ranking loss
            mr_loss = self.mr_loss(x1, x2, y)
            
            # We also compute the classification loss on our binary target,
            # calling the parent BCEWithLogitsLoss
            bce_loss = super().__call__(outputs, y_threshold)
            
            # Add the classification and ranking loss together, and you're done!
            return bce_loss + mr_loss
```

## But what about the margin term?

Understanding the margin parameter is key to using the margin ranking loss effectively. Recall the margin ranking loss formulation:

{% mathjax %}
\mathrm{loss}\left( x_1, x_2, y\right) = \mathrm{max}\left( 0, -y * (x_1 - x_2) + \mathrm{margin} \right)
{% endmathjax %}

There are three possibilities:

* _**No margin**_ ({% mathjax %} \mathrm{margin} = 0 {% endmathjax %}): This is exactly as described above. The loss will be zero if the prediction pair are in the right order, and the (positive) difference between them if they are in the wrong order.
* _**Negative margin**_ ({% mathjax %} \mathrm{margin} < 0 {% endmathjax %}): This makes the loss more forgiving, by moving some pairwise positive losses to zero. You can use this, for example, if your rankings are very noisy or you suspect that nearby rankings are so similar that the difference is meaningless.
* _**Positive margin**_ ({% mathjax %} \mathrm{margin} > 0 {% endmathjax %}): This makes the loss more difficult, by making some pairwise zero losses positive. It's essentially saying, yes this pair is in the right order but please increase the distinction. The best way to check if you should use a positive margin is to look at the margin ranking training loss for your model. If it's going to zero, that means might be able to squeeze more performance out by increasing the margin.

I chose the margin to be 1 based on the loss curves below. You can see that -1 and 0 are too easy, either trending in the wrong direction or falling directly to zero.

<figure>
  <img src="/images/margin_ranging_-1.svg" alt="Margin ranking -1"/>
  <img src="/images/margin_ranging_0.svg" alt="Margin ranking 0"/>
  <img src="/images/margin_ranging_1.svg" alt="Margin ranking 1"/>
  <figcaption style="text-align: center"><em>Margin ranking loss for different margins for Single-class with ranking models trained on the wine quality dataset.</em></figcaption>
</figure>

## Aside: Is ROCAUC the right metric?

We regard certain metrics as "regression metrics" and others as "classification metrics." ROCAUC is a classification metric, right? Yes, but remember that to calculate the ROCAUC, you must first rank all your predictions so that you can check the true positive rate against the false positive rate at different thresholds. So the ranking concept is baked into the most popular classification metric.

On the other hand, if you train a regression model, you can still calculate the ROCAUC. Just threshold your targets into bins, as we did above (wine quality 0-3 is the negative class, 4-5 is the positive class). Since the predictions from a regression model can obviously be ranked, you have everything you need to compute ROCAUC.

## Conclusion

You might be wondering why I didn't try a pure margin ranking loss. This usually doesn't work very well, as you need some kind of anchor (in this case, the binary cross entropy loss). This is key to the wider world of [contrastive representation learning](https://lilianweng.github.io/posts/2021-05-31-contrastive/), and a fascinating active field of deep learning research.

I also don't mean to imply that neural networks are the best way to solve this toy problem. The cool thing about neural networks, and gradient descent methods in general, is that you have flexibility in how you construct your loss function: it's no issue to mash together a ranking loss and classification loss as long as it's differentiable.

Once again, all model code and plots are available [in my Kaggle example notebook](https://www.kaggle.com/code/simplex1/a-neural-network-trick-for-ranking-problems/notebook).
