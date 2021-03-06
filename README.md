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
* [diff and patch](#diff-and-patch)
* [git](#git)
* [Docker](#docker)
* [GCC basics](#gcc-basics)
* [Python](#python)
* [.deb files](#deb-files)


FFmpeg
======

Images → Video
--------------
* <code>ffmpeg -f image2 -i frame_%04d.png movie.avi</code><br/>
  Convert frames `frame_0000.png, frame_0001.png...` to video.
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

Video rescaling
---------------
* <code>ffmpeg -i movie.avi -vf scale=iw*0.5:-1 -q:v 0 half_resolution_movie.avi</code><br/>
  Resample video (`-vf scale=...`) to half-width (`iw*0.5`). 
  * The height is automatically computed to keep the aspect ratio constant (`:-1`). 
  * Keeps video quality (`-q:v 0`). 
  * `-vf` is short for `-filter:v`, the "video filter" function.
  
Video cropping
--------------
* <code>ffmpeg -i movie.avi <b>-filter:v "crop=640:480:100:0"</b> cropped-movie.avi</code><br>
  Cut a `640x480` cropout from a video. The crop is placed `100` pixels from the left edge, and `0` pixels from the top edge.

Change video playback speed
---------------------------
* <code>ffmpeg <b>-r 48</b> -i movie.avi <b>-r 24</b> movie-twice-as-fast.avi</code><br>
  Speed up or slow down a video.
  * The `-r` option in front(!) of the `-i` input specifies the assumed framerate at which the input video should be played back. This is a bit unintuitive: In the example, if `movie.avi` is actually a 24 fps video, this command will <i>play it back at double speed (48 fps) and record a new video at 24 fps</i>.
  * The `-r` option at the output specifies the output's encoding framerate. The default is 24 frames per second.
  * If the output's `-r` option is omitted, the output will be stored as a video with <b>all</b> input frames, along with the annotation that it is a 48 fps video. Doing this does not reduce the storage size of the output video. If the output's `-r` option is used, only those frames that match this framerate are kept.
  
Cut video
---------
* <code>ffmpeg -i movie.avi <b>-ss 00:10:00 -to 00:12:00</b> scene-from-movie.avi</code><br>
  Cut the segment between the timestamps 10m00s and 12m00s into a standalone video.
* <code>ffmpeg -i movie.avi <b>-ss 00:10:00 -t 120</b> scene-from-movie.avi</code><br>
  Cut a 120 second segment starting at timestamp 10m00s.
  

Transcoding videos into another codec
-------------------------------------

* <code>ffmpeg -i INPUT_VIDEO -vcodec libtheora -qscale:v 10 -c:a copy OUTPUT_VIDEO.ogv</code><br/>
  * `-vcodec libtheora` selects the free "libtheora" codec for the <b>[Theora](https://en.wikipedia.org/wiki/Theora)</b> video format. The option `-vcodec` is equivalent to `-c:v`.
  * `-qscale:v 10` requests the highest (scale 0-10) image quality.
  * `-c:a copy` copies the audio stream without changes.
  * (This requires an FFmpeg installation with libtheora support. Check if the output of `ffmpeg` contains `--enable-libtheora` in the `configuration` line)

* <code>ffmpeg -i INPUT_VIDEO -vcodec libx264 -crf 23 OUTPUT_VIDEO.mkv</code>
  * `libx264` selects the free encoder for the <b>[H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC)</b> format.
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

Video-in-video montage
----------------------
```
ffmpeg -i first_input_video.avi
       -i second_input_video.avi 
       -loop 1 -i alpha_mask.png
       -filter_complex "[2:v]alphaextract,split[part1][part2];
                        [1:v][part1]alphamerge[top];
                        [0:v][part2]alphamerge[bottom];
                        [top][bottom]overlay"
       -qscale:v 0 
       video-montage.avi
```
This command takes 2 input videos and 1 alpha mask image, and produces an overlay video, something like this:
```
AAAAAAAAAAA   BBBBBBBBBBB   00000000000   AAAAAAAAAAA
AAAAAAAAAAA + BBBBBBBBBBB + 00000011111 = AAAAAABBBBB
AAAAAAAAAAA   BBBBBBBBBBB   11111111111   BBBBBBBBBBB
  
   first         second        alpha         output
   video         video         mask          video
```

* `-i first_input_video.avi`, `-i second_input_video.avi` are the input videos; the order is important!
* `-i alpha_mask.png` is an image whose alpha channel will be used to determine where which input video is visible.<br>
  Since the other inputs are videos (with many frames) and this image has only a single frame, we have to tell ffmpeg that this alpha mask should be used for all video frames. This is what `-loop 1` *before* the image input does.
* `-filter_complex` is where the magic happens. It takes a series of commands which are processed in-order. The output of the last command in the chain is the output of the whole filter complex.
  * `[2:v]` is "the video channel of the input with index 2". Inputs are 0-indexed, so `[2:v]` is our alpha mask image (which was turned into a video stream by the `-loop 1` option above). `alphaextract` isolates the image's alpha channel and discards the rest of the image, and `split` turns the alpha channel into two outputs: a foreground mask and a background mask. We give these newly produced masks names (so we can refer to them later) using `[part1][part2]`.
  * `[1:v]` is the video part of `second_input_video.avi`. `[part1]` refers to one of the masks produced by the previous step. We need both because the `alphamerge` operation requires an input stream and a mask stream. It produces a masked video which we call `[top]`.
  * Same here: combine `first_input_video.avi` and the `[part2]` mask into a masked video which we call `[bottom]`.
  * Now we combine the `[top]` and `[bottom]` masked videos into one single video using the `overlay` directive. We do not give the output of this command a name because we do not need to process it any further within the `filter_complex`.
* `-qscale:v 0` finally specifies that we want a high-quality video when we write out `video-montage.avi`.

Fix WebM/VP9 color channels
-----------------------
<pre><code>ffmpeg [input from PNG frames] <b>-pix_fmt yuv420p</b> -codec:v libvpx-vp9 output_video.webm</code></pre>

My ffmpeg produces wrong colors when rendering a video from input frames. This `-pix_fmt` option fixes that.


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

Default permissions for new files/folders
-----------------------------------------
```
umask 027
```

With this in the `.bashrc`, newly created files get the permission set `rw-r-----` (640) and newly created folders get `rwxr-x---` (750) (the argument for `umask` specifies which permissions get <i>blocked</i>).


Shell
=====

Simple `for`-loop
-----------------
```
i=0; while test $i -lt 10; do echo $i; i=`expr $i + 1`; done
```

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

Information about files
-----------------------
* **General information**
  * `file <file>`<br>
    Print file type information.
  
* **Images**
  * `identify -format '%w x %h\n' <image>`<br>
    Print image dimensions. `%w x %h` prints e.g. `640 x 480`. `identify` is a part of ImageMagick.

Reduce PDF file size
--------------------
`gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dNOPAUSE -dQUIET -dBATCH -dPDFSETTINGS=/prepress -sOutputFile=<OUTPUT> <INPUT>`<br>
This uses `ghostscript` to downsample and compress images in the PDF (among other things; I'm not quite sure what exactly this command does).

For stronger compression, you can specify the downsampling mode and resolution for embedded images:
`gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dNOPAUSE -dQUIET -dBATCH -dPDFSETTINGS=/prepress -dColorImageDownsampleType=/Bicubic -dColorImageResolution=288 -dPrinted=false -sOutputFile=OUTPUT.pdf INPUT.pdf`

If you need an extra small document and can live with low resolution images, try this:
`gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dNOPAUSE -dQUIET -dBATCH -dPDFSETTINGS=/prepress -dColorImageDownsampleType=/Bicubic -dColorImageResolution=144 -dPrinted=false -sOutputFile=OUTPUT.pdf INPUT.pdf`

Search-and-replace recursively in a folder
------------------------------------------
`find . -type f -print0 | xargs -0 sed -i 's/$SEARCH_TERM/$REPLACE_TERM/g'`

**Warning: This is a powerful and potentially dangerous command.**

This replaces `$SEARCH_TERM` by `$REPLACE_TERM` in all files within the current directory.

diff and patch
==============

Create patchfile
----------------

<code>diff -c old_file new_file > patch_from_old_to_new_file</code>

* This creates a patchfile from the differences between `old_file` and `new_file`. 
* The `-c` option makes `diff` produce more "verbose" differences by including <u>c</u>ontext information around the diffs.

Apply patchfile
---------------

<code>patch -i patch_from_old_to_new_file</code>

* This applies the patchfile created in the previous step. 
* The patchfile contains information about the filename from which it was created, so in this example: if the target file is called `old_file` and is also in the current $PWD, this will automatically apply the patch to that file (it's actually somewhat more complex: see `man patch`).
* If the files do not match, use <br>
  <code>patch <b>another_file</b> -i patch_from_old_to_new_file</code>
* To produce another file instead of patching in-place, use <br>
  <code>patch another_file -i patch_from_old_to_new_file <b>-o patched_file</b></code>
  

Git
===

Correct a wrong commit message
--------------------
<code>git <b>--amend</b> ...files... -m "new message"</code>

Undo accidental `--amend` commit
--------------------------------
<code>git reset --soft HEAD@{1}</code><br/>
Afterwards, redo the commit (preferentially without `--amend` this time).
Credit: [StackOverflow](https://stackoverflow.com/a/1459264)

Undo `git add` <i>before</i> commit
-----------------------------------
<code>git <b>reset HEAD</b> -- WRONGLY_ADDED_FILE</code>

Undo `git add` <i>after</i> commit (but <i>before</i> push!)
-----------------------------------------------------------
<code>git <b>reset HEAD^</b></code><br/>
This resets your pointer to the previous commit. Afterwards you can `git add` and then `git commit` again.

Undo `git add` <i>after</i> push
--------------------------------
Nope, u dun fucked up.


Docker
======

Image/container management
--------------------------
These commands require that the user is in the `docker` group; else put a `sudo` in front of each `docker`.

* <code>docker image rm \`docker image list -f "dangling=true" -q\`</code><br/>
  Remove images which are no longer needed by any other image
* <code>docker container stop \`docker container ps -a -q\`</code><br/>
  Stop all containers. With `rm` instead of `stop`, this <i>removes</i> all containers.
  
Freezing OpenGL windows
-----------------------
My Docker+nvidia-docker2+nvidia/cudagl container setup freezes as soon as any OpenGL-using application opened a window, even `glxgears`. This problem was solved by <b>disabling</b> the "Allow Flipping" option in the `nvidia-settings` manager → "X Screen 0" tab → "OpenGL Settings" menu.


GCC basics
==========

Assume the following codebase, where `main.cpp` includes `#include "class.h"` and uses code from `class.cpp`.
```
$ ls
class.cpp  class.h  main.cpp
```

Compilation and static linking in one step
------------------------------------------
```
$ g++ class.cpp main.cpp -o my-app
$ ls 
class.cpp  class.h  main.cpp  my-app
```


Compilation and static linking
------------------------------

* Step 1: Compile object files from source code
```
$ g++ -c class.cpp -o class.o
$ g++ -c main.cpp -o main.o
$ ls
class.cpp  class.h  class.o  main.cpp  main.o
```
* Step 2: Link object files into executable
```
$ g++ main.o class.o -o my-app
$ ls
class.cpp  class.h  class.o  main.cpp  main.o  my-app
$ ldd my-app
        linux-vdso.so.1 [...]
        libc.so.6 [...]
        /lib64/ld-linux-x86-64.so.2 [...]
```

Compilation and dynamic linking
------------------------------

* Step 1: Compile object files from source code
```
$ g++ -c class.cpp -o class.o
$ g++ -c main.cpp -o main.o
$ ls
class.cpp  class.h  class.o  main.cpp  main.o
```
* Step 2: Create shared object file
```
$ g++ -shared class.o -o class.so
$ ls
class.cpp  class.h  class.o  class.so  main.cpp  main.o
```
* Step 3: Link object files into executable
```
$ g++ main.o class.so -o my-app
$ ls
class.cpp  class.h  class.o  class.so  main.cpp  main.o  my-app
$ ldd my-app
        linux-vdso.so.1 [...]
        class.so [...]
        libc.so.6 [...]
        /lib64/ld-linux-x86-64.so.2 [...]
```



Python
======

Create a new VirtualEnv
-----------------------
* Python2: `virtualenv -p python2 my-virtualenv-folder`
* With Python3, the preferred way is: (needs package `python3-venv`): `python3 -m venv my-virtualenv-folder`

Drop to an interactive prompt from within a script
--------------------------------------------------
```import code; code.interact(local=locals())```

This line starts an interactive console with access to all variables of that line's scope. When the interactive console is closed, the interpreter continues running the rest of the script. **Any changes made in the interactive console will persist.**


.deb Files
==========

Installing .deb files
---------------------
* <code>dpkg -i software.deb</code><br/>
  This installs software from a local `.deb` file. Unlike `apt`, this command does not automatically pull the necessary dependencies. This can typically be repaired using `apt-get --fix-broken install`. Note that software installed with `dpkg -i` can still be uninstalled with `apt-get remove`.

Unpacking .deb files locally
----------------------------
* <code>ar x ARCHIV.deb</code><br/>
  This unpacks a `.deb` archive file. Useful if some software is self-contained and can be used locally without <i>installing</i> the software.
  * <code>tar xfJ data.tar.xz</code><br/>
    The `ar x` command typically results in `.xz`-compressed tarfiles. Inflate using `unxz`, or `tar`'s `-J` flag.
