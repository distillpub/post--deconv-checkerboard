<link rel="stylesheet" type="text/css" href="assets/common.css">
<script src="assets/d3.min.js"></script>
<script src="assets/d3-path.min.js"></script>
<script src="assets/underscore.js"></script>
<h1>{{ typewriter.title }}</h1>
{{> byline.html}}

In the last year, we've seen incredible progress in generating images with neural networks.
Using techniques like Generative Adversarial Networks and Variational Auto-Encoders,
researchers are able to create extremely compelling image samples.

{{> assets/upsample_RecentSamples.svg}}

However, if we look very closely at these generated images,
we often see this strange checkerboard pattern of artifacts.
It's more obvious in some cases than others,
but a large fraction of recent models exhibit this behavior.

{{> assets/upsample_RecentArtifacts.svg}}

Mysteriously, the checkerboard pattern tends to be most prominent in images with strong colors.
What's going on? Do neural networks hate bright colors?

**TODO:** Improve above figures.

---

Deconvolution & Magnitude
==========================

The actual cause of these artifacts is remarkably simple, once you see it.

When we have neural networks generate images, we often have them build them up
from low resolution, high-level descriptions.
This allows the network to describe the rough image and then fill in the details.

In order to do this, we need some way to go from a lower resolution image to a higher one.
We generally do this with the *deconvolution* operation.
Roughly, deconvolution allows the model to... **TODO**

Unfortunately, deconvolution can lead to uneven overlap.
This creates magnitude problems in the output, with some outputs being much bigger than their neighbors.

{{> assets/deconv1d.html}}

In particular, deconvolution has these uneven overlap issues when the kernel size (the output window size) is not divisible by the stride (the spacing between points on the top). The size of the uneven overlap is the kernel size modulo the stride.

The problem only gets worse in two dimensions. The uneven overlaps on the two axes multiply together, creating a checkerboard-like pattern of varying magnitudes.

{{> assets/deconv2d.html}}

When we stack multiple deconvolutions, the artifacts may cancel out, or they may become more complicated, higher level patterns.

{{> assets/deconv1d_multi.html}}

Balanced Upsampling
====================

To avoid these artifacts,

{{> assets/upsample_DeconvTypes.svg}}



<!--
Things Luke Vilnis suggested we look into:
* should we be calling it deconv?
* this paper argues for a different but related architecture http://128.84.21.199/pdf/1609.07009.pdf (seems like high-res literature already does soemthing similar to what we are doing)
-->
