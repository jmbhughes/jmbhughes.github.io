+++
title = 'Goal of Anomaly Detection in Non-stationary Data'
date = 2019-09-09
draft = false
+++

{{< youtube YOrf3zcF4oM >}}

I was explaining anomaly detection in non-stationary data to someone and threw together this crude example figure. The blue points are nominal and represent 90% of the points. The red are anomalous and represent 10% of the points. In this example, the red data is stationary while the blue passes through it. Thus, it would be very difficult to differentiate the red and blue points when they overlap. However, even if we only had a few frames of this video, we would like to be able to realize there are two dynamics going on.  

The code for this is:

```py
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation

i = 1
w = 1

N = 200
alpha = 0.1

xx = []
yy = []
steps = 30
for step in range(steps):
    anoms = np.random.binomial(N, alpha)
    x = np.random.uniform(i, i+w, size=(N-anoms,2))
    y = np.random.uniform(4, 5, size=(anoms,2))
    if step < 10:
        i += 0.5
        w += 0.5
    else:
        i -= 0.25
        w -= 0.25
    xx.append(x)
    yy.append(y)

x_lower, x_higher = 0, 15
y_lower, y_higher = 0, 15

fig, ax = plt.subplots(figsize=(8,8))

ax = plt.axis([x_lower, x_higher, y_lower, y_higher])

anomalies, = plt.plot([0], [np.sin(0)], 'r.')
typicals, = plt.plot([0], [np.sin(0)], 'b.')


def animate(i):
    anomalies.set_data(yy[i][:,0], yy[i][:,1])
    typicals.set_data(xx[i][:,0], xx[i][:,1])
    return anomalies,

# create animation using the animate() function
myAnimation = animation.FuncAnimation(fig, animate, frames=range(steps), interval=10, blit=True, repeat=True)

plt.show()

# Set up formatting for the movie files
Writer = animation.writers['ffmpeg']
writer = Writer(fps=5, metadata=dict(artist='Me'), bitrate=1800)
myAnimation.save('im.mp4', writer=writer)
```
