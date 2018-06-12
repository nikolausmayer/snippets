snippets
========

A (poorly) curated list of awesome codes and commands that do Magic™. Mostly command line. All Linux.

![LICENSE](img/license-MIT-blue.svg)
![awesome](img/awesome-yes-yellow.svg)

Table of Contents
=================

* [ffmpeg](#ffmpeg)
* [shell](#shell)
* [Docker](#docker)
* [Sources](#sources)


FFmpeg
======

Images → Video
--------------
* <code>ffmpeg -f image2 -i frame_%04d.png movie.avi</code><br/>
  Convert frames to video.
* <code>ffmpeg -f image2 -i frame_%04d.png <b>-q:v 0</b> movie__good_quality.avi</code><br/>
  Convert frames to *good quality* video. `-q` is a shorthand for `-qscale`. `v` selects the "video" stream which is the only stream available in a video-from-images.
* <code>ffmpeg -f image2 <b>-framerate 5</b> -i frame_%04d.png movie__slowmotion.avi</code><br/>
  Convert frames to slow-motion video. `-framerate 5` tells ffmpeg that the input frames are a 5 fps source. The default output framerate of ffmpeg is(?) 25 fps, so this produces a video that is 5x slower than normal.
* <code>ffmpeg -f image2 <b>-start_number 10</b> -i frame_%04d.png movie__start_at_frame_10.avi</code><br/>
  Convert frames to video, starting at file `frame_0010.png`. This is useful if the input frames do not start at a low number such as 0 or 1: ffmpeg will actively search for the first frame, but it will not do so very eagerly and give up soon.

Video → Images
--------------
* <code>ffmpeg -i movie.avi -f image2 output_frame_%04d.png</code><br/>
  Convert video frames to image files.

Video manipulation
------------------
* <code>ffmpeg -i movie.avi -vf scale=iw*0.5:-1 -q:v 0 half_resolution_movie.avi</code><br/>
  Resample video (`-vf scale=...`) to half-width (`iw*0.5`). The height is automatically computed to keep the aspect ratio constant (`:-1`). Keeps video quality (`-q:v 0`). `-vf` is short for `-filter:v`, the "video filter" function.




Shell
=====

Simple `for`-loop
-----------------
<code>i=0; while test $i -lt 10; do echo $i; i=`` `expr $i + 1` ``; done</code>
* `i=0` initializes the variable `$i` to zero.
* `while ...; do ...; done` is the loop.
* `test $i -lt 10` is `i < 10` (`-lt` is "<i>l</i>ess <i>t</i>han"). In bash, you can use `[ $i -lt 10 ]` instead.
* `` `expr $i + 1` `` increments `$i`. 

Print every <i>n</i>-th line of a file
--------------------------------------
<code>awk '!(NR%<b><i>n</i></b>)' file</code><br/>
* `NR` contains the line index. 
* `NR%n` is a modulo on that index and evaluates to `true` if if is != 0, i.e. if a line is <i>not</i> an <i>n</i>-th line.
* We negate this truthiness using `!(NR%n)`. 
* (Double quotes also work instead of the single quotes used in this example.)




Docker
======

Image/container management
--------------------------
These commands require that the user is in the `docker` group; else put a `sudo` in front of each `docker`.

* <code>docker rmi \`docker images -f "dangling=true" -q\`</code>
  Remove images which are no longer needed by any other image
* <code>docker stop \`docker ps -a -q\`</code>
  Stop all containers. With `rm` instead of `stop`: Remove all containers.





Sources
=======
* http://www.theunixschool.com/2012/12/how-to-print-every-nth-line-in-file-in.html

