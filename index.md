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

Deconvolution & Uneven Overlap
==============================

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
This also makes the overlap more uneven:
in one dimension, a stride 2, size 3 deconvolution has some outputs with twice the number of inputs as others,
but in two dimensions this becomes a factor of four!

{{> assets/deconv2d.html}}

Typically, neural nets use multiple layers of deconvolution when creating images,
iteratively building a larger image out of a series of lower resolution descriptions.
While it's possible for these 

When we stack multiple deconvolutions, the artifacts may cancel out, or they may become more complicated, higher level patterns.

*"lorem ipsum" text is there so that we can see diagrams in context of text around them.*

*Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum*

{{> assets/deconv1d_multi.html}}

*Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum*

-----

Balanced Upsampling
====================

To avoid these artifacts,

*Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum*

<figure class="w-page-clip-left">
<img src="assets/upsample_DeconvTypes.svg">
</figure>

*Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum*

<!--
Things Luke Vilnis suggested we look into:
* should we be calling it deconv?
* this paper argues for a different but related architecture http://128.84.21.199/pdf/1609.07009.pdf (seems like high-res literature already does soemthing similar to what we are doing)
-->
