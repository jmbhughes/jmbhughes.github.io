+++
title = 'AIA Temporal Coloring'
date = 2022-06-03
draft = false
math = false
toc = false
+++


## Temporal coloring

I downloaded a few hours of AIA data in 171 Angstroms for another project.
(For the non-solar people, AIA is the Atmospheric Imaging Assembly
  aboard NASA's Solar Dynamics Observatory. It takes pictures of the Sun in
  extreme ultraviolet.) I started playing around with it, and I ended up making
  the hypnotic and enchanting, at least in my opinion, video at the end of this
  post.

It was a simple process. Just combine different images in time as the colors of
an RGB image. For this video, I set the current time as red, an image five minutes
in the future as green, and an image ten minutes into the future as blue. It
makes a kind of cool shimmery effect. Most of the image is white because the
Sun didn't change dramatically on that timescale, but the dynamic regions
really pop with color.  

Each frame is separated by 12 seconds. So it's four hours of data. If you want
to download data of your own, check out [this SunPy example](https://docs.sunpy.org/en/stable/generated/gallery/acquiring_data/downloading_cutouts.html#sphx-glr-generated-gallery-acquiring-data-downloading-cutouts-py). It's easier to fetch a lot of
image cutouts instead of the full images.

{{< youtube uC3y9Vxyu_o >}}

It's pretty, but I don't know that it has much scientific value. Just a little
artsy representation of some amazing scientific data.
