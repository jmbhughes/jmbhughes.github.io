+++
title = 'Automating SUVI YouTube'
date = 2023-01-22
draft = false

[extra]
katex = true
toc = true

[taxonomies]
tags = ["solar", "python", "programming"]
+++

## Intro

The Sun is changing all the time. In order to identify interesting new events to study, a solar physicist
must monitor data streams observing the Sun on a regular basis. I thought it might be neat to make a
tool that would automatically download extreme ultraviolet images of the Sun from the GOES-R SUVI instrument
and then upload them as a movie to YouTube for easy monitoring. My goal was to use [Prefect](https://www.prefect.io/)
to run the entire process on a schedule. However, it appears YouTube has [special restrictions on who can upload
using scripts](https://github.com/jonnekaunisto/simple-youtube-api/issues/22). For now, I can have the video creation
automated but have to manually upload the video each day.  

## Core Code

Below is the core of the code that makes this happen.

```py
from datetime import datetime, timedelta
import tempfile
from glob import glob
import os
import shutil
from dateutil.parser import parse as parse_datetime
from typing import Optional

from prefect import task, flow, get_run_logger
from goessolarretriever import Retriever, Satellite, Product
from astropy.io import fits
from moviepy.editor import VideoClip, ImageClip, concatenate_videoclips
from moviepy.video.io.bindings import mplfig_to_npimage
import matplotlib.pyplot as plt
import numpy as np


@task
def download_suvi_data(start_date: datetime, end_date: datetime,
                       satellite : Satellite = Satellite.GOES16,
                       product : Product = Product.suvi_l2_thmap) -> str:
    """ Downloads SUVI data as specified to a temporary directory and returns that directory path"""
    logger = get_run_logger()
    r = Retriever()
    results = r.search(satellite, product, start_date, end_date)
    logger.info(f"Found {len(results)} {product} images. Now will download to a temp dir.")

    temp_directory = tempfile.mkdtemp()

    r.retrieve(results, temp_directory)
    logger.info(f"Downloaded {product} images to {temp_directory}")
    return temp_directory


@task
def render_suvi_channel_movie(channel_filenames, duration=0.25,
                      channel_power=0.25, channel_min=0, channel_max=3):
    """ Renders a movie for one SUVI channel where each frame gets the specified duration. 

    Since the SUVI images have a high dynamic range, we raise the brightneses to the `channel_power`
    and scale the output plot between `channel_min` and `channel_max`. 
    """
    fig, ax = plt.subplots(figsize=(10, 10))
    fig.tight_layout()

    def make_frame(t):
        with fits.open(channel_filenames[t]) as hdul:
            channel_head = hdul[1].header
            channel_data = hdul[1].data

        ax.clear()
        ax.set_title(parse_datetime(channel_head['DATE-OBS']).strftime("%Y-%m-%d %H:%M"))
        ax.imshow(np.sign(channel_data) * np.power(np.abs(channel_data), channel_power),
                      vmin=channel_min, vmax=channel_max)
        ax.set_axis_off()
        return ImageClip(mplfig_to_npimage(fig)).set_duration(duration)

    frames = [make_frame(i) for i in range(len(channel_filenames))]
    return concatenate_videoclips(frames)


@flow
def make_suvi_channel_movie(out_filename: str, start_date: datetime, end_date: Optional[datetime], channel=Product.suvi_l2_ci195):
    logger = get_run_logger()

    temp_channel_directory = download_suvi_data(start_date, end_date, product=channel)
    channel_filenames = sorted(glob(os.path.join(temp_channel_directory, "*.fits")))

    logger.info("Beginning to render movie.")
    thmap_animation = render_suvi_channel_movie(channel_filenames)

    logger.info(f"Beginning to write movie to {out_filename}.")
    thmap_animation.write_videofile(out_filename, fps=24)

    logger.info(f"Deleting {temp_channel_directory} as a clean up step.")
    shutil.rmtree(temp_channel_directory)

if __name__ == "__main__":
    now = datetime.now()
    three_days_ago = datetime(now.year, now.month, now.day) - timedelta(days=3)
    make_suvi_channel_movie(f"suvi_195_{three_days_ago.strftime('%Y%m%d')}.mp4", three_days_ago, None)
```

## Walkthrough

It's easiest to read this code from bottom to top. You'll see that the script when executed starts by calling `make_suvi_channel_movie`.
This Prefect flow handles the main logic. (If you've never used Prefect I'd recommend checking out [their docs](https://docs.prefect.io/). I find them quite helpful.])

First, we use a Python package I made a while back called [goes-solar-retriever](https://github.com/jmbhughes/goes_solar_retriever)
to download the SUVI images for a given day. We download them all to one temporary directory. We'll open them one-by-one in the
`render_suvi_channel_movie` function to avoid overloading our memory with too many images. We can make one frame at a time.
(You may still have trouble with this approach if you tried a long period of data because of how I used `ImageClip` from `MoviePy`.
I'm not sure how much memory each clip consumes. Another approach is to write a `.png` file for each frame and then use a tool like `ffmpeg`
to generate a movie.)

In the internal `make_frame` function, we do some fancy scaling of the images so that they look better. We also set each frame to show for
0.25 seconds. With 360 images per day from SUVI's Level 2 products, we can make a 1.5 minute video. YouTube converts any videos shorter than
a minute to the shorts format. That would be undesirable for us because it means you cannot scrobble through the video; you're stuck watching it from
beginning to end.

## Automating the YouTube upload

As I mentioned in the intro, due to restrictions from YouTube you cannot upload from a script and make the video public automatically it seems.
Everytime I tried my video got locked for review. I believe they're worried that spammers/scammers could upload many videos using scripts. It seems
there is a way around this, but it takes a special review that I didn't apply for.

However, if you had permissions to upload public videos there is a nice Python package called `simple-youtube-api` that simplifies interfacing with YouTube.
If you read [its GitHub](https://github.com/jonnekaunisto/simple-youtube-api) or just Googling around you can find instructions on setting up the needed credentials.

```py
@task
def upload_to_youtube(filename, start_date):
    # log in into the channel
    channel = Channel()
    channel.login("client_secret.json", "credentials.storage")

    # setting up the video that is going to be uploaded
    video = LocalVideo(file_path=filename)

    # setting snippet
    video.set_title(f"SUVI on {start_date.strftime('%Y-%m-%d')}")
    video.set_description("This is a description")
    video.set_category("science")
    video.set_default_language("en-US")

    # setting status
    video.set_embeddable(True)
    video.set_license("creativeCommon")
    video.set_privacy_status("private")
    video.set_public_stats_viewable(True)
    video.set_made_for_kids(False)

    # uploading video and printing the results
    video = channel.upload_video(video)
    print(video.id)
    print(video)

    # liking video
    video.like()
```

By injecting that task at the end of your flow, you could upload the videos automatically.

## Extending with thematic maps

We can extend the code above to generate thematic map movies from SUVI as well. The thematic map product identifies what different phenomena are present on the Sun,
e.g. bright regions, coronal holes, filaments. You can read more about my thematic map work in [this press release](https://www.ncei.noaa.gov/news/solar-classification)
or in [this paper](https://www.swsc-journal.org/articles/swsc/full_html/2019/01/swsc180074/swsc180074.html). The image below shows a three-color composite of SUVI images
on the left and a thematic map on the right. Yellow, for example, are bright regions while green are coronal holes.

![thematic map gif](thematic.gif)

The code gets a bit more complicated because the thematic maps neede to be rendered with a specific color table.
I added a utility `build_thmap_cmap` to help with this process.

### Code for thematic maps

```py
from matplotlib.colors import ListedColormap
from matplotlib.patches import Patch

THEMATIC_MAP_COLORS = {"unlabeled": "white",
                       "outer_space": "black",
                       "bright_region": "#F0E442",
                       "filament": "#D55E00",
                       "prominence": "#E69F00",
                       "coronal_hole": "#009E73",
                       "quiet_sun": "#0072B2",
                       "limb": "#56B4E9",
                       "flare": "#CC79A7"}

THEME_MAPPING = {1: 'outer_space',
                 3: 'bright_region',
                 4: 'filament',
                 5: 'prominence',
                 6: 'coronal_hole',
                 7: 'quiet_sun',
                 8: 'limb',
                 9: 'flare'}


def build_thmap_cmap():
    colortable = [THEMATIC_MAP_COLORS[THEME_MAPPING[i]] if i in THEME_MAPPING else 'black'
                  for i in range(max(list(THEME_MAPPING.keys())) + 1)]
    return ListedColormap(colortable)

@task
def render_suvi_double_movie(thmap_filenames, channel_filenames, duration=0.1,
                      channel_power=0.25, channel_min=0, channel_max=3):
    fig, axs = plt.subplots(ncols=2, figsize=(20, 10))
    fig.tight_layout()

    thmap_cmap = build_thmap_cmap()

    def make_frame(t):
        with fits.open(thmap_filenames[t]) as hdul:
            thmap_head = hdul[0].header
            thmap_data = hdul[0].data

        with fits.open(channel_filenames[t]) as hdul:
            channel_head = hdul[1].header
            channel_data = hdul[1].data

        # plot thmap
        axs[0].clear()
        axs[0].set_title(parse_datetime(thmap_head['DATE-OBS']).strftime("%Y-%m-%d %H:%M"))
        axs[0].imshow(thmap_data, cmap=thmap_cmap, vmin=-1, vmax=10, interpolation='none')
        axs[0].set_axis_off()
        legend_elements = [Patch(facecolor=color, edgecolor="black", label=label.replace("_", " "))
                           for label, color in THEMATIC_MAP_COLORS.items()]
        axs[0].legend(handles=legend_elements, loc='upper center', bbox_to_anchor=(0.5, 0.95),
                      ncol=3, fancybox=True, shadow=True)

        # plot channel image
        axs[1].clear()
        axs[1].set_title("195")
        axs[1].imshow(np.sign(channel_data) * np.power(np.abs(channel_data), channel_power),
                      vmin=channel_min, vmax=channel_max)
        axs[1].set_axis_off()
        return ImageClip(mplfig_to_npimage(fig)).set_duration(duration)

    frames = [make_frame(i) for i in range(len(thmap_filenames))]
    return concatenate_videoclips(frames)


@flow
def make_suvi_double_movie(out_filename: str, start_date: datetime, end_date: datetime, channel=Product.suvi_l2_ci195):
    logger = get_run_logger()

    temp_channel_directory = download_suvi_data(start_date, end_date, product=channel)
    temp_thmap_directory = download_suvi_data(start_date, end_date)

    thmap_filenames = glob(os.path.join(temp_thmap_directory, "*.fits"))
    channel_filenames = glob(os.path.join(temp_channel_directory, "*.fits"))

    logger.info("Beginning to render movie.")
    thmap_animation = render_suvi_double_movie(thmap_filenames, channel_filenames)

    logger.info(f"Beginning to write movie to {out_filename}.")
    thmap_animation.write_videofile(out_filename, fps=24)

    logger.info(f"Deleting {temp_thmap_directory} as a clean up step.")
    shutil.rmtree(temp_thmap_directory)
    shutil.rmtree(temp_channel_directory)
```

## Results and future improvements

You can see the first such movie I created [at this YouTube video](https://youtu.be/T4FXE5ZAjSs). The colormap could be
changed to match the standard SUVI colormaps. It also doesn't have high enough contrast in my opinion. But it's a first step.

I would like to continue this line of automated visualization and include other detection algorithms. For example, it would be cool
to be able to identify all the bright points on the Sun over the course of a day, analyze their properties, and output a result chart.
