## How to freeze all weights in Caffe model for finetunning?

For common Convolution layers, it is easy to set 'lr_mult' as 0 to freeze weights. Moreover, you can set 'propagate_down' as 0 to avoid the backward.
But when your model contains 'PReLU' layer, the 'propagate_down' parameter dose not seem to work. Now, you should add the param group, include lr_mult and
weight_decay, just like this:

```
layer {
  name: "prelu1"
  type: "PReLU"
  bottom: "conv1"
  top: "conv1"
  
  param { 
    lr_mult: 0 
    decay_mult: 0 
  }

}
```