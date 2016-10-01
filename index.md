<link rel="stylesheet" type="text/css" href="assets/common.css">
<script src="assets/d3.min.js"></script>
<script src="assets/d3-path.min.js"></script>
<script src="assets/underscore.js"></script>
<h1>{{ typewriter.title }}</h1>
{{> byline.html}}

In the last year, we've seen incredible progress in generating images with neural networks.
Using techniques like Generative Adversarial Networks and Variational Auto-Encoders,
researchers are able to create extremely compelling image samples. **TODO:** Some of the below aren't samples.

<figure class="w-page">
<img src="assets/upsample_RecentSamples.png">
</figure>

However, if we look very closely at these generated images,
we often see this strange checkerboard pattern of artifacts.
It's more obvious in some cases than others,
but a large fraction of recent models exhibit this behavior.

<figure class="w-page">
<img src="assets/upsample_RecentArtifacts.png">
</figure>

Mysteriously, the checkerboard pattern tends to be most prominent in images with strong colors.
What's going on? Do neural networks hate bright colors?

---

Deconvolution & Magnitude
==========================

The actual cause of these artifacts is remarkably simple, once you see it.

When we have neural networks generate images, we often have them build them up
from low resolution, high-level descriptions.
This allows the network to describe the rough image and then fill in the details.

In order to do this, we need some way to go from a lower resolution image to a higher one.
We generally do this with the *deconvolution* (or transposed convolution) operation.
Roughly, deconvolution layers allows the model to use every point
in the small image to "paint" a square in the larger one.
(For a detailed disucssion, see [Dumoulin & Visin, 2016](https://arxiv.org/pdf/1603.07285v1.pdf).)

Unfortunately, deconvolution can lead to uneven overlap.
This creates magnitude problems in the output, with some outputs being much bigger than their neighbors.

TODO: Offset logic min /2

{{> assets/deconv1d.html}}

In particular, deconvolution has uneven overlap when the kernel size (the output window size) is not divisible by the stride (the spacing between points on the top).

The problem only gets worse in two dimensions. The uneven overlaps on the two axes multiply together, creating a checkerboard-like pattern of varying magnitudes.

{{> assets/deconv2d.html}}

When we stack multiple deconvolutions, the artifacts may cancel out, or they may become more complicated, higher level patterns.

*"lorem ipsum" text is there so that we can see diagrams in context of text around them.*

*Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum*

{{> assets/deconv1d_multi.html}}

*Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum*

Balanced Upsampling
====================

To avoid these artifacts,

*Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum*

<figure class="w-page">
{{> assets/upsample_DeconvTypes.svg}}
</figure>

*Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum*

<!--
Things Luke Vilnis suggested we look into:
* should we be calling it deconv?
* this paper argues for a different but related architecture http://128.84.21.199/pdf/1609.07009.pdf (seems like high-res literature already does soemthing similar to what we are doing)
-->
