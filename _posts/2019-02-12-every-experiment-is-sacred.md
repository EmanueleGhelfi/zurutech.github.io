---
author: "manughelfi"
layout: post
did: "blog6"
title:  "Every Experiment is Sacred"
slug:  Every Experiment is sacred
date:   2019-02-12 8:00:00
categories: machine-learning
img: sacred3.jpg
banner: sacred3.jpg
tags: machine-learning
description: "How to manage Machine Learning experiments and make them reproducible."
---

<style>
.blockquote {
    padding: 60px 80px 40px;
    position: relative;
}
.blockquote p {
    font-family: "Utopia-italic";
    font-size: 35px;
    font-weight: 700px;
    text-align: center;
}

/*blockquote p::before {
    content: "\f095"; 
    font-family: FontAwesome;
   display: inline-block;
   padding-right: 6px;
   vertical-align: middle;
  font-size: 180px;
 }*/

.blockquote:before {
  position: absolute;
  font-family: 'FontAwesome';
  top: 0;
  
  content:"\f10d";
  font-size: 200px;
  color: rgba(0,0,0,0.1);
   
}

.blockquote::after {
    content: "";
    top: 20px;
    left: 50%;
    margin-left: -100px;
    position: absolute;
    border-bottom: 3px solid #bf0024;
    height: 3px;
    width: 200px;
}

@import url(https://fonts.googleapis.com/css?family=Open+Sans:400italic);
.otro-blockquote{
  font-size: 1.4em;
  width:60%;
  margin:50px auto;
  font-family:Open Sans;
  font-style:italic;
  color: #555555;
  padding:1.2em 30px 1.2em 75px;
  border-left:8px solid #78C0A8 ;
  line-height:1.6;
  position: relative;
  background:#EDEDED;
}

.otro-blockquote::before{
  font-family:Arial;
  content: "\201C";
  color:#78C0A8;
  font-size:4em;
  position: absolute;
  left: 10px;
  top:-10px;
}

.otro-blockquote::after{
  content: '';
}

.otro-blockquote span{
  display:block;
  color:#333333;
  font-style: normal;
  font-weight: bold;
  margin-top:1em;
}
</style>
Managing Machine Learning experiments is usually painful.

<div markdown="1" class="blog-image-container"  style="width:60%;height:60%; display: block;margin-left: auto;margin-right: auto;">
![Figure 1 - Experiments](/images/sacred3.jpg "Figure 1 - Experiments"){:class="blog-image"}
</div>

The usual workflow when approaching a problem using Machine Learning tools is the following. You study the problem, define a possible solution, then you implement that solution and measure its quality. Often the solution depends on some parameters, we will refer to the set of parameters with the term **configuration**. Parameters can include model type, optimizers, learning rate, batch size, steps, losses, and many others.

The configuration highly influences the performance of your solution. If you have enough time and computational power, it is possible to use a Bayesian approach for solving the hyper-parameter selection problem, but in real situations, it is common to perform a limited set of experiment and select the best configuration among them.

In this article we describe how to manage experiments in a good way, ensuring inspectability and reproducibility. We will focus on Machine Learning experiments in python using [Tensorflow](www.tensorflow.com) even if this approach can be applied to all computational experiments.

## Simple (Bad) Solution

To keep track of the configuration the usual way to go (at least in Tensorflow) is to save your models and logs inside a folder with a common pattern for the parameters name and value.

Using this setting with [Tensorboard](https://www.tensorflow.org/guide/summaries_and_tensorboard) (the official Tensorflow visualization tool) it is possible to relate the plot to its configuration. However, this is **not** the right way to manage experiments since it is very hard to see how parameters affect performance. In fact, Tensorboard offers none way to order experiments by configuration, selecting only some experiments or order experiments by performance.
In this way, we do not have any control and tracking of our source code, except for our VCS. Unfortunately, when doing experiments we do not always push our changes since they might be tiny changes or only trials. This can cause issues since our experiments might not be reproducible.
An experiment using this style can be implemented in the following way:
```python
# other imports ...

import tensorflow as tf

def get_logdir(config):
    logdir = ""
    for key in config:
        logdir = logdir + f"_{key}_{config[key]}"
    return logdir

def main():
    # load the configuration
    config = load_config()

    # load dataset, model, summaries, loss using the configuration

    # merge summaries
    summary_op = tf.summary.merge_all()

    # define a logdir to keep track of the configuration
    logdir = f"logs/{get_logdir(config)}"

    # instantiate a tensorflow saver
    saver = tf.train.Saver()

    writer = tf.SummaryWriter(log_dir)

    with tf.Session() as sess:
        # train loop
        for step in range(config['steps']):
                # your training code ...

                loss_value = sess.run(loss)

                # write summaries
                writer.add_summary(summaries, step)
                writer.flush()

                # save model
                saver.save(sess, logdir)
```

In this code snippet we load the model, dataset and other stuff using the configuration loaded. The logdir defined using the configuration is needed for associating a configuration to the correct plot in Tensorboard.
In this way we obtain the following folder structure using only 4 parameters:
```
├── lr=0.0002_batch_size=32_kernel_size=5_steps=100
├── lr=0.0001_batch_size=32_kernel_size=5_steps=100
├── lr=0.0002_batch_size=32_kernel_size=7_steps=100
├── lr=0.0002_batch_size=16_kernel_size=7_steps=100
```
This is not very comfortable and definitely it is not the right way to manage experiments and make them reproducible.


## Sacred Solution

<blockquote class="blockquote"><p>
Every experiment is sacred <br>
Every experiment is great <br>
If an experiment is wasted <br>
God gets quite irate
</p></blockquote>
<br>
[Sacred](https://sacred.readthedocs.io/en/latest/index.html) is a tool that lets you configure, organize, and log computational experiments. It is designed to introduce a minimal overhead and code addition.

With sacred you can:

- Keep track of all parameters of your experiment
- Run your experiment with different settings
- Save configuration and results in files or database
- Reproduce your results.

Sacred dumps and saves **everything** into MongoDB including:

- Metrics (training/validation loss, accuracy, or anything else you decide to log)
- Console output
- All the source code you executed
- All the imported library with their versions used at runtime
- All your configuration parameters
- Hardware spec of your host
- Random seeds
- Artifacts and resources.

To visualize and interact with your experiments a nice visualization board for this tool is [Omniboard](https://github.com/vivekratnavel/omniboard).

<div markdown="1" class="blog-image-container">
![Figure 1 - Omniboard](/images/omniboard4.png "Figure 1 - Omniboard"){:class="blog-image"}
</div>
*Figure 1 - Omniboard*

<div markdown="1" class="blog-image-container">
![Figure 2 - Omniboard - Experiment details](/images/omniboard5.png "Figure 2 - Omniboard - Experiment details"){:class="blog-image"}
</div>
*Figure 2 - Omniboard - Experiment details*

Omniboard lets you tag, annotate and order your experiments making inspection and model selection easy.

## Sacred Integration

Integrating sacred in your code is painless. The previous code becomes:

```python
# other imports ...

# tensorflow
import tensorflow as tf
from tensorflow.python.training.summary_io import SummaryWriterCache

# sacred
from sacred import Experiment
from sacred.observers import MongoObserver

# instantiate a sacred experiment
ex = Experiment("experiment_name")
# add the MongoDB observer
ex.observers.append(MongoObserver.create())

# load the configuration
ex.add_config("default.json")
ex.add_config("experiment.json")

@ex.automain
@LogFileWriter(ex)
def main(batch_size, learning_rate, steps, ... params, _run):
    # load dataset, model, summaries, loss using the parameters

    # merge summaries
    summary_op = tf.summary.merge_all()

    # define a logdir using the experiment id of sacred
    logdir = f"logs/{_run._id}"

    # instantiate a tensorflow saver
    saver = tf.train.Saver()

    writer = tf.SummaryWriter(log_dir)

    with tf.Session() as sess:
        # train loop
        for step in range(steps):
                # your training code ...

                loss_value = sess.run(loss, feed_dict)

                # log loss to sacred
                ex.log_scalar("training.loss", loss_value, step)

                summaries = sess.run(summary_op, feed_dict)

                # write summaries as tensorflow logs
                writer.add_summary(summaries, step)
                writer.flush()

                # save model
                saver.save(sess, logdir)
```

In this way we create a new instance of `Experiment` and an instance of `MongoObserver` in order to store data to MongoDB. We add two different configurations, a default configuration (`default.json`) and an experiment configuration (`experiment.json`). The function decorated with `@ex.automain` is the main function of the experiment and its parameters are automatically injected by Sacred. The `LogFileWriter(ex)` decorator is used to store the location of summaries produced by Tensorflow (created by `tensorflow.summary.FileWriter`) into the experiment record specified by the ex argument. Whenever a new `FileWriter` instantiation is detected in a scope of the decorator or the context manager, the path of the log is copied to the experiment record exactly as passed to the `FileWriter`. The location(s) can be then found under `info["tensorflow"]["logdirs"]` of the experiment.

Using this approach the folder structure is clean (folders are named with a progressive id) and we detach the folder name from the actual experiment configuration.

## Conclusion
Managing experiments in a correct way is very important, especially when doing research. Reproducibility is an essential requirement for computational studies including
those based on machine learning techniques. Sacred is a very powerful tool, very easy to integrate in your codebase, saving you the effort of developing a custom way to keep track of your experiments.

## References

- [Tensorflow](https://www.tensorflow.org)
- [Tensorboard](https://www.tensorflow.org/guide/summaries_and_tensorboard)
- [Sacred](https://sacred.readthedocs.io/en/latest/index.html)
- [Sacred repository](https://github.com/IDSIA/sacred)
- [Omniboard](https://github.com/vivekratnavel/omniboard)
- [Medium Article](https://medium.com/@u39kun/managing-your-machine-learning-experiments-and-making-them-repeatable-in-tensorflow-pytorch-bc8043099dbd)