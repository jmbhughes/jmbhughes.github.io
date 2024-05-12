+++
title = 'Training an autoencoder with mostly noise'
date = 2019-09-17
draft = false
toc = false
+++

I am working on a project where we wish to use anomaly detection to find what image patches have structure and which don't. As an aside, I ran an experiment on MNIST. You have 500 images of fives. You have 5000 images that are pure noise. You train a deep convolutional autoencoder. What you end up with is the following reconstruction:  

![explanatory figure](fig.png)

The top row are the inputs and the bottom row are the reconstructions. You find images of fives even when nothing is present.
