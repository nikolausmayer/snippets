snippets
========

A (poorly) curated list of awesome codes and commands that do Magic™. Mostly command line. All Linux.

![LICENSE](img/license-MIT-blue.svg)
![awesome](img/awesome-yes-yellow.svg)

Table of Contents
=================

* [ffmpeg](#ffmpeg)
* [.bashrc](#bashrc)
* [shell (also includes `grep` etc)](#shell)
* [git](#git)
* [Docker](#docker)
* [Python](#python)
* [.deb files](#deb-files)


FFmpeg
======

Images → Video
--------------
* <code>ffmpeg -f image2 -i frame_%04d.png movie.avi</code><br/>
  Convert frames to video.
* <code>ffmpeg -f image2 -i frame_%04d.png <b>-q:v 0</b> movie__good_quality.avi</code><br/>
  Convert frames to *good quality* video. 
  * `-q` is a shorthand for `-qscale`. 
  * `v` selects the "video" stream which is the only stream available in a video-from-images.
* <code>ffmpeg -f image2 <b>-framerate 5</b> -i frame_%04d.png movie__slowmotion.avi</code><br/>
  Convert frames to slow-motion video. 
  * `-framerate 5` tells ffmpeg that the input frames are a 5 fps source. The default output framerate of ffmpeg is(?) 25 fps, so this produces a video that is 5x slower than normal.
* <code>ffmpeg -f image2 <b>-start_number 10</b> -i frame_%04d.png movie__start_at_frame_10.avi</code><br/>
  Convert frames to video, starting at file `frame_0010.png`. This is useful if the input frames do not start at a low number such as 0 or 1: ffmpeg will actively search for the first frame, but it will not do so very eagerly and give up soon.

Video → Images
--------------
* <code>ffmpeg -i movie.avi -f image2 output_frame_%04d.png</code><br/>
  Convert video frames to image files.

Video manipulation
------------------
* <code>ffmpeg -i movie.avi -vf scale=iw*0.5:-1 -q:v 0 half_resolution_movie.avi</code><br/>
  Resample video (`-vf scale=...`) to half-width (`iw*0.5`). 
  * The height is automatically computed to keep the aspect ratio constant (`:-1`). 
  * Keeps video quality (`-q:v 0`). 
  * `-vf` is short for `-filter:v`, the "video filter" function.

Transcoding videos into another codec
-------------------------------------

* <code>ffmpeg -i INPUT_VIDEO -vcodec libtheora -qscale:v 10 OUTPUT_VIDEO.ogv</code><br/>
  * `-vcodec libtheora` selects the free <b>[Theora](https://en.wikipedia.org/wiki/Theora)</b> video codec.
  * `-qscale:v 10` requests the highest (scale 0-10) image quality.
  * (This requires an FFmpeg installation with libtheora support. Check if the output of `ffmpeg` contains `--enable-libtheora` in the `configuration` line)

* <code>ffmpeg -i INPUT_VIDEO -vcodec libx264 -crf 23 OUTPUT_VIDEO.mkv</code>
  * `libx264` selects the free encoder for the <b>[h264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC)</b> codec.
  * `-crf` stands for "Constant Rate Factor" and selects an image quality setting. `23` is the default setting; possible settings are 0-51 where lower numbers are higher quality.
  * (This requires an FFmpeg installation with libx264 support. Check if the output of `ffmpeg` contains `--enable-libx264` in the `configuration` line)

Recording screencasts
---------------------
* <code>ffmpeg -f x11grab -s 1600x1120 -i :0.0+1600,0 -c:v libx264 -crf 23 screencast.mp4</code><br/>
  * `-f x11grab` is the "video source" for screengrabbing.
  * `-s 1600x1120` is the desired recording size. The default is(?) `640x480`.
  * `-i` selects the input, in this case:
    * `:0.0` is a display index. Can be found out using `echo $DISPLAY`.
    * My 2 monitors have the same `$DISPLAY`, so to record the right screen I shift the recording window by 1600 pixels to the right (and 0 pixels in vertical direction) using `+1600,0`.
  * `-c:v libx264 -crf 23` is the video codec selection. Any other option will work as well.


.bashrc
=======

Better history search
---------------------

```
bind '"\e[A": history-search-backward';
bind '"\e[B": history-search-forward';
```

This binds the UP/DOWN arrow keys to history search with autocompletion.

**This is my number one ```.bashrc``` productivity hack.**


Shell
=====

Simple `for`-loop
-----------------
<code>i=0; while test $i -lt 10; do echo $i; i=\`expr $i + 1\`; done</code>
* `i=0` initializes the variable `$i` to zero.
* `while ...; do ...; done` is the loop.
* `test $i -lt 10` is `i < 10` (`-lt` is "<i>l</i>ess <i>t</i>han"). In bash shells, you can use `[ $i -lt 10 ]` instead.
* <code>\`expr $i + 1\`</code> increments `$i`. There are more concise ways to write this in e.g. bash, but this is the only really portable version I know.

Print every <i>n</i>-th line of a file
--------------------------------------
<code>awk '!(NR%<b><i>n</i></b>)' file</code><br/>
* `NR` contains the line index. 
* `NR%n` is a modulo on that index and evaluates to `true` if if is != 0, i.e. if a line is <i>not</i> an <i>n</i>-th line.
* We negate this truthiness using `!(NR%n)`. 
* (Double quotes also work instead of the single quotes used in this example.)

grep
----
* Logical "OR"<br/>
  <code>grep "pattern1\\|pattern2" filename</code><br/>
* Logical "AND"<br/>
  There is no AND in grep, but it can be emulated via<br/>
  <code>grep "pattern1.*pattern2\\|pattern2.*pattern1" filename</code>


Git
===

Correct a wrong commit message
--------------------
<code>git <b>--amend</b> ...files... -m "new message"</code>

Undo `git add` <i>before</i> commit
-----------------------------------
<code>git reset HEAD -- ...wrongly added files...</code>

Undo `git add` <i>after</i> commit (but <i>before</i> push!)
-----------------------------------------------------------
<code>git reset HEAD^</code><br/>
This resets your pointer to the previous commit. Afterwards you can `git add` and then `git commit` again.

Undo `git add` <i>after</i> push
--------------------------------
Nope, u dun fucked up.


Docker
======

Image/container management
--------------------------
These commands require that the user is in the `docker` group; else put a `sudo` in front of each `docker`.

* <code>docker rmi \`docker images -f "dangling=true" -q\`</code><br/>
  Remove images which are no longer needed by any other image
* <code>docker stop \`docker ps -a -q\`</code><br/>
  Stop all containers. With `rm` instead of `stop`, this <i>removes</i> all containers.



Python
======

Create a new VirtualEnv
-----------------------
* Python2: `virtualenv -p python2 my-virtualenv-folder`
* Python3 (needs package `python3-venv`): `python3 -m venv my-virtualenv-folder`


.deb Files
==========

* <code>ar x ARCHIV.deb</code><br/>
  This unpacks a `.deb` archive file. Useful if some software is self-contained and can be used locally without <i>installing</i> the software.
