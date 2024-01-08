# Paper Summary
- [Distilling the Knowledge in a Neural Network](https://arxiv.org/pdf/1503.02531.pdf)
## Introduction
- Once the cumbersome model has been trained, we can then use a different kind of training, which we call "distillation" to transfer the knowledge from the cumbersome model to a small model that is more suitable for deployment.
- A more abstract view of the knowledge is that it is a ***learned mapping from input vectors to output vectors.***
- For cumbersome models that learn to discriminate between a large number of classes, the normal training objective is to maximize the average log probability of the correct answer, but a side-effect of the learning is that the trained model assigns probabilities to all of the incorrect answers and even when these probabilities are very small, some of them are much larger than others. The relative probabilities of incorrect answers tell us a lot about how the cumbersome model tends to generalize. An image of a BMW, for example, may only have a very small chance of being mistaken for a garbage truck, but that mistake is still many times more probable than mistaking it for a carrot.
- ***It is generally accepted that the objective function used for training should reflect the true objective of the user as closely as possible. Despite this, models are usually trained to optimize performance on the training data when the real objective is to generalize well to new data.*** It would clearly be better to train models to generalize well, but this requires information about the correct way to generalize and this information is not normally available. ***When we are distilling the knowledge from a large model into a small one, however, we can train the small model to generalize in the same way as the large model. If the cumbersome model generalizes well because, for example, it is the average of a large ensemble of different models, a small model trained to generalize in the same way will typically do much better on test data than a small model that is trained in the normal way on the same training set as was used to train the ensemble.***
## Related Works
- ***[1] circumvent this problem by using the logits (the inputs to the final softmax) rather than the probabilities produced by the softmax as the targets for learning the small model and they minimize the squared difference between the logits produced by the cumbersome model and the logits produced by the small model.***
## Methodology
- ***An obvious way to transfer the generalization ability of the cumbersome model to a small model is to use the class probabilities produced by the cumbersome model as "soft targets" for training the small model.*** For this transfer stage, we could use the same training set or a separate "transfer" set. When the cumbersome model is a large ensemble of simpler models, we can use an arithmetic or geometric mean of their individual predictive distributions as the soft targets. ***When the soft targets have high entropy, they provide much more information per training case than hard targets and much less variance in the gradient between training cases, so the small model can often be trained on much less data than the original cumbersome model and using a much higher learning rate.***
- ***Our more general solution, called "distillation", is to raise the temperature of the final softmax until the cumbersome model produces a suitably soft set of targets. We then use the same high temperature when training the small model to match these soft targets. We show later that matching the logits of the cumbersome model is actually a special case of distillation.***
- Distillation
    - Neural networks typically produce class probabilities by using a "softmax" output layer that converts the logit, $z_{i}$ , computed for each class into a probability, $q_{i}$, by comparing $z_{i}$ with the other logits.
    $$q_{i} = \frac{\exp({z_{j} / T})}{\sum_{j}\exp(z_{j} / T)}$$
    - where $T$ is a temperature that is normally set to $1$. ***Using a higher value for*** $T$ ***produces a softer probability distribution over classes.***
    - ***In the simplest form of distillation, knowledge is transferred to the distilled model by training it on a transfer set and using a soft target distribution for each case in the transfer set that is produced by using the cumbersome model with a high temperature in its softmax. The same high temperature is used when training the distilled model, but after it has been trained it uses a temperature of 1.***
    - When the correct labels are known for all or some of the transfer set, this method can be significantly improved by also training the distilled model to produce the correct labels. One way to do this is to use the correct labels to modify the soft targets, but we found that a better way is to simply use a weighted average of two different objective functions.
    - ***The first objective function is the cross entropy with the soft targets and this cross entropy is computed using the same high temperature in the softmax of the distilled model as was used for generating the soft targets from the cumbersome model.***
    - ***The second objective function is the cross entropy with the correct labels. This is computed using exactly the same logits in softmax of the distilled model but at a temperature of 1. We found that the best results were generally obtained by using a condiderably lower weight on the second objective function. Since the magnitudes of the gradients produced by the soft targets scale as $1 / T^{2}$ it is important to multiply them by $T^{2}$ when using both hard and soft targets. This ensures that the relative contributions of the hard and soft targets remain roughly unchanged if the temperature used for distillation is changed while experimenting with meta-parameters.***
## Training
### Loss
- Each case in the transfer set contributes a cross-entropy gradient, $dC / dz_{i}$, with respect to each logit, $z_{i}$ of the distilled model. If the cumbersome model has logits $v_{i}$ which produce soft target probabilities $p_{i}$ and the transfer training is done at a temperature of $T$, this gradient is given by:
- ***In the high temperature limit, distillation is equivalent to minimizing*** $1/2(z_{i} − v_{i})^{2}$, provided the logits are zero-meaned separately for each transfer case. At lower temperatures, distillation pays much less attention to matching logits that are much more negative than the average. This is potentially advantageous because these logits are almost completely unconstrained by the cost function used for training the cumbersome model so they could be very noisy. On the other hand, the very negative logits may convey useful information about the knowledge acquired by the cumbersome model. Which of these effects dominates is an empirical question. We show that when the distilled model is much too small to capture all of the knowledege in the cumbersome model, intermediate temperatures work best which strongly suggests that ignoring the large negative logits can be helpful.
### Datasets
- The transfer set that is used to train the small model could consist entirely of unlabeled data or we could use the original training set. We have found that using the original training set works well, especially if we add a small term to the objective function that encourages the small model to predict the true targets as well as matching the soft targets provided by the cumbersome model. ***Typically, the small model cannot exactly match the soft targets and erring in the direction of the correct answer turns out to be helpful.***
## References
- [1] [Model Compression](http://www.cs.cornell.edu/~caruana/compression.kdd06.pdf)

# Supplementary
$$L = \sum_{(x, y) \in \mathbb{D}} \bigg[L^{KD}_{CE}\big\{S(x, \tau), T(x, \tau)\big\} + \lambda L_{CE}\big\{S(x, 1), y\big\}\bigg]$$
- $\mathbb{D}$: Dataset
- $x$: Image
- $y$: Label
- $S$: Student model
- $T$: Teacher model
- $\tau$: Temperature