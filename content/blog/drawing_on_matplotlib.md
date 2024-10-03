+++
title = 'Drawing on a Matplotlib Plot'
date = 2024-06-26
draft = false
toc = false

[taxonomies]
tags = ["programming", "python"]
+++

It's easy to draw on a Matplotlib plot and capture the coordinates of the points by using a `LassoSelector`. Here's an example script:

```py
import matplotlib.pyplot as plt
from matplotlib.widgets import LassoSelector
import numpy as np

# function saying what to do with the vertices
def on_lasso(vertices):
    print(np.array(vertices))

data = np.random.random((500, 500))

# properties of the line to visualize what you just drew
line_props = dict(color="red", linewidth=2)

# make the visualization
fig, ax = plt.subplots()
lasso = LassoSelector(ax, on_lasso, props=line_props)
ax.imshow(data, vmin=0, vmax=1)
plt.show()
```

Instead of merely printing the vertices, you could save them to a list for later usage.
Or you could get fancy and draw them back on the plot.

I discovered this handy trick when writing [SolarAnnotator](https://github.com/jmbhughes/solarannotator), a tool to create ground truth data for solar image segmentation.

Overall, the `LassoSelector` is a powerful way to enable easy annotation in a Matplotlib plot.
