+++
title = 'Truncate to specified number of significant figures'
date = 2021-05-10
draft = false
toc = false

[taxonomies]
tags = ["personal", "beets"]
+++

It's a simple request. You have a number, e.g. 1.25, and want to truncate it to say 2 significant figures. You specifically need to truncate instead of rounding, i.e. you need to get 1.2 instead of 1.3. How do you do that quickly in Python? Check out this code snippet:

```py
from math import log10, floor
import decimal

decimal.getcontext().rounding = decimal.ROUND_DOWN

def truncate_sig(x, sig=2):
    x = float(x)  # make sure it's a float or decimal throws an error
    if x == 0:  # can't take log10(0) so just return 0
        return 0
    elif x <= 0:  # if it's negative, determine sigfigs of positive and multiply by -1
        return -1 * truncate_sig(np.abs(x), sig=sig)
    else:  # normal positive case
        x = decimal.Decimal(x)
        return float(round(x, sig - int(floor(log10(abs(x)))) - 1))
```

It's based on [this](https://stackoverflow.com/a/3413529/10046967) and [this](https://stackoverflow.com/a/39165933/10046967) on stackoverflow.
