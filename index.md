<link rel="stylesheet" type="text/css" href="assets/common.css">
<script src="assets/d3.min.js"></script>
<script src="assets/d3-path.min.js"></script>
<script src="assets/underscore.js"></script>
<h1>{{ typewriter.title }}</h1>
{{> byline.html}}

In the last year, we've seen incredible progress in generating images with neural networks.
Using techniques like Generative Adversarial Networks and Variational Auto-Encoders,
researchers are able to create extremely compelling image samples. **TODO:** Some of the below aren't samples.

<figure class="w-page-clip">
<img src="assets/upsample_RecentSamples.png">
</figure>

However, if we look very closely at these generated images,
we often see this strange checkerboard pattern of artifacts.
It's more obvious in some cases than others,
but a large fraction of recent models exhibit this behavior.

<figure class="w-page-clip">
<img src="assets/upsample_RecentArtifacts.png">
</figure>

Mysteriously, the checkerboard pattern tends to be most prominent in images with strong colors.
What's going on? Do neural networks hate bright colors?

---
Deconvolution & Overlap
=======================

The actual cause of these artifacts is remarkably simple, once you see it.

When we have neural networks generate images, we often have them build them up
from low resolution, high-level descriptions.
This allows the network to describe the rough image and then fill in the details.

In order to do this, we need some way to go from a lower resolution image to a higher one.
We generally do this with the *deconvolution* (or transposed convolution) operation.
Roughly, deconvolution layers allows the model to use every point
in the small image to "paint" a square in the larger one.
(For a detailed disucssion, see [Dumoulin & Visin, 2016](https://arxiv.org/pdf/1603.07285v1.pdf).)

Unfortunately, deconvolution can easily have uneven overlap,
putting more of the metaphorical paint in some places than others.
While the neural network could, in principle, carefully learn weights to avoid this
-- as we'll discuss in more detail later --
in practice they struggle to avoid it completely.

{{> assets/deconv1d.html}}

In particular, deconvolution has uneven overlap when the kernel size (the output window size) is not divisible by the stride (the spacing between points on the top).

The overlap pattern also forms in two dimensions.
The uneven overlaps on the two axes multiply together,
creating a characteristic checkerboard-like pattern of varying magnitudes.

{{> assets/deconv2d.html}}

In fact, the uneven overlap tends to be more extreme in two dimensions!
Because the two patterns are multiplied together, the uneveness gets squared.
For example, in one dimension, a stride 2, size 3 deconvolution has some outputs with twice the number of inputs as others,
but in two dimensions this becomes a factor of four.

Now, neural nets typically use multiple layers of deconvolution when creating images,
iteratively building a larger image out of a series of lower resolution descriptions.
While it's possible for these stacked deconvolutions to cancel out artifacts,
they often compound, creating artifacts on a variety of scales.

{{> assets/deconv1d_multi.html}}

Stride 1 deconvolutions -- which we often see as the last layer in successful models
-- are quite effective dampening artifacts. They remove artifacts of frequencies
that divide their size, and reduce others artifacts of frequency less than their
size. However, artifacts can still leak through, as seen in many recent models.

In addition to the high frequency checkerboard-like artifacts we observed above
early deconvolutions can create lower-frequency artifacts,
which we'll explore in more detail later.

-----
Overlap & Learning
====================

Thinking about things in terms of uneven overlap is -- while a useful framing --
kind of simplistic. For better or worse, our models learn weights for their deconvolutions.


In theory, they could learn to carefully write to unevenly overlapping squares so that the output
is evenly balanced.
In practice, however, neural networks struggle to learn to not create these patterns.
This is kind of surprising to us:
barring extreme regularization, it seems like they should be able to learn this.

In fact, not only do models with uneven overlap not learn to avoid this,
but models with even overlap often learn kernels that cause similar artifacts!
The artifacts seem milder, and have a different pattern, but they're present.
(See [Dumoulin, et al., 2016](https://arxiv.org/pdf/1606.00704v1.pdf),
which uses stride 2 size 4 deconvolutions, as an example.)

There's probably a lot of factors at play here.
One issue may be with the discriminator and its gradients, which we'll discuss more later.
But a big part of the problem seems to be deconvolution.
At best, deconvolution is fragile because it is very easily represents artifact creating functions, even when the size is carefully chosen.
At worst, creating artifacts is the default behavior of deconvolution.

Is there a different way to upsample that is more resistant to artifacts?

-----
Balanced Upsampling
====================

To avoid these artifacts,

<figure class="w-page-clip-left">
<img src="assets/upsample_DeconvTypes.svg">
</figure>


<!--
Things Luke Vilnis suggested we look into:
* should we be calling it deconv?
* this paper argues for a different but related architecture http://128.84.21.199/pdf/1609.07009.pdf (seems like high-res literature already does soemthing similar to what we are doing)
-->


-----
Artifacts in Gradients
======================
