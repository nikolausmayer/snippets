snippets
========

A (poorly) curated list of awesome codes and commands that do Magic™. Mostly command line. All Linux.

![LICENSE](https://img.shields.io/badge/license-MIT-blue.svg)
![awesome](https://img.shields.io/badge/awesome-yes-yellow.svg)

Table of Contents
=================

* [ffmpeg](#ffmpeg)


FFmpeg
======

Videos from and to frame images
-------------------------------

* <code>ffmpeg -f image2 -i frame_%04d.png movie.avi</code><br/>
  Convert frames to video.
* <code>ffmpeg -f image2 -i frame_%04d.png <b>-q:v 0</b> movie__good_quality.avi</code><br/>
  Convert frames to *good quality* video. `-q` is a shorthand for `-qscale`. `v` selects the "video" stream which is the only stream available in a video-from-images.
* <code>ffmpeg -f image2 <b>-framerate 5</b> -i frame_%04d.png movie__slowmotion.avi</code><br/>
  Convert frames to slow-motion video. `-framerate 5` tells ffmpeg that the input frames are a 5 fps source. The default output framerate of ffmpeg is(?) 25 fps, so this produces a video that is 5x slower than normal.
* <code>ffmpeg -f image2 <b>-start_number 10</b> -i frame_%04d.png movie__start_at_frame_10.avi</code><br/>
  Convert frames to video, starting at file `frame_0010.png`. This is useful if the input frames do not start at a low number such as 0 or 1: ffmpeg will actively search for the first frame, but it will not do so very eagerly and give up soon.


