+++
title = 'The Value of a Peer-Reviewed Activity'
date = 2019-06-14
draft = false
toc = false
+++

This week, we have been talking about proof writing in the discrete mathematics course I'm teaching. Yesterday, I started class by having students answer how confident they are about their proof writing skills on a scale of 1 to 10, 1 being "clueless and not sure where to start", 5 being "Okay and ready for homework," and 10 being "I can do most any proof you throw at me with ease." I then had them individually complete three proofs in 15 minutes.  

![problem statement](problem.png)

Many students struggled with Problem 2 because they are not comfortable with sequence notation. Some misunderstood Problem 3 and tried proving something completely different than intended. After 15 minutes, they traded papers (with anonymous codes instead of their names so there was no embarrassment) and reviewed each someone else's papers as we went over them in class. They wrote some lovely, encouraging comments to each other. For example, one student had no idea of what to do on the second problem so they left it blank and their reviewer wrote, "Bet you can do it now! :D." Others noted where the proof failed and wrote a comment that they too had that difficulty. It's refreshing to see such kindness. Finally, they took the same survey about their proof writing again and there were dramatic changes in confidence.  

![stats for improvements](stats.png)

Many more students were confident in their abilities. I'm not sure who the one who reported 1 after the exercise is. I hope they come to office hours.  

This is in no way a rigorous test, but students expressed they learned more from this exercise because it forced them to think about the material instead of just going through the proof together on the board. I imagine there is a psychological benefit of seeing how someone else is doing too and being kind to them in written comments. It was also suggested that I do a problem example in class without proving it before class so students could hear my raw thought process first hand. I'll have to think about that. I like being prepared, but I'm sure I could find some way to do that.

```py
import matplotlib.pyplot as plt
import numpy as np

# set up the data
rank = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
pre = [0, 6, 25, 25, 13, 13, 19, 0, 0, 0] # pre exercise confidence
post = [7, 0, 7, 13, 33, 13, 7, 7, 13, 0] # post exercise confidence

# weighted average of confidence
pre_mean = np.sum(np.array(rank) * (np.array(pre)/100))
post_mean = np.sum(np.array(rank) * (np.array(post)/100))

# figure generation
fig, axs = plt.subplots(ncols=2, sharey=True)
axs[0].bar(rank, pre, align='center')
axs[1].bar(rank, post, align='center')
axs[0].set_title("Pre-exercise (mean={:0.1f})".format(pre_mean))
axs[1].set_title("Post-exercise (mean={:0.1f})".format(post_mean))
for ax in axs:
    ax.set_xticks(rank)
    ax.set_xlabel("Student confidence")
axs[0].set_ylabel("Percentage of students answering")
fig.show()
```
