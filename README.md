# An open source MP4 repair

## What it is?

This utility is designed to recover h264 video in a broken, deleted, or incomplete (truncated) MP4, MOV, or 3GP files. In principle, it can also work with AVI, MKV, M4V and other containers.

Video restoration works well, but sound restoration is difficult and less reliable.

If you shoot a video on a camera, phone, action camera, record video from the screen or from drones, then it is possible that this utility may be useful to you.


## Why this fork

This is a fork of bookkojot's amazing (but possibly abandoned, judging by the unmerged patches) mp4fixer utility. This fork integrates patches from pemre and fmogollonr.


## Symptoms

A video file may not play for any number of reasons, but here are some common symptoms that mean you should try this tool:

* ``ffprobe foo.mp4`` renders a ``moov atom not found`` error
* VLC cannot play your file
* ``MP4Box -v -info foo.mp4`` reports that your file is truncated


## What is needed?

To restore the files you need:
1. The broken files or images of the card / flash drive / disk where the video was recorded
2. A new (SEPARATE) disk with sufficient free space
3. A machine running Linux or Windows, ``perl``, ``ffmpeg`` (and its associated ``ffprobe`` and ``ffplay`` commands)
4. To restore aac-sound, you need `faad2` (libfaad-dev package) + `gcc` (to compile ``aacfixer`` utility)
5. Sample of a *good* file of the same type, using the same settings, from the same device as your corrupted file.


## How to use it?

1. If you have a deleted or incomplete or truncated file, first get an image from it using the winhex utility or similar. You can also try to recover files, but you can lost few seconds. Linux users can point the card directly, like /dev/sdX, but having a backup when restoring data is always a good idea.
2. Take the old video file or shoot a new video that will be recorded under the same conditions as the one being restored. Be sure to record it exactly under the same conditions, including even shaking on the screen (same bitrate), if it was - so we will find exactly what we are looking for. The length of the video is 20 seconds, but if you feed more - it's OK.
3. You need to run:

```perl fixer.pl <good_file.mp4> <bad_file.mp4> <output_prefix>```

For instance:

```perl fixer.pl good.mp4 bad.mp4 recovered```

This creates:

* `recovered-out-video.h264` (the restored video)
* `recovered-out-audio.raw` (any restored audio)


## How it works?

To begin, intermediate files are created:

* `*-headers.aac` - the file that you will need later to restore the sound
* `*-headers.h264` - the video file that will be used to create headers
* `*-nals.txt` - temporary file, with reference video packets. Under Windows 7, this file is created long for an unknown reason to me. This file is needed to "learn" the features of the encoding of the sample file
* `*-nals-stat.txt` is what we "learned". If something goes wrong - send this file.
* `*-stat.mp4` - temporary file, actually a copy of the sample file, only without sound, cropped up to 20 seconds.

At the very end of the work will be created:

* `*-out-video.h264` - restored video
* `*-out-audio.raw` - the restored audio. To be precise - just what was between the video.

All files will be created in the current directory, from where you run the script.

You can take any player, for example ffplay and play `*-out-video.h264`, but with sound it will be a bit more complicated.

## About restoring sound:

Different video cameras can write sound differently. At a minimum, it can be in different formats.

1. If the sound was in mp3 format, then you can simply rename *-out-audio.raw to file.mp3 and open it with any player.
2. If the sound was in PCM format (and its subspecies, such as ULAW), as some Sony video cameras do, then you can easily save it if you convert it to WAV by typing something like:

```ffmpeg -f s16le -ar 48000 -ac 2 -i somefile-out-audio.raw -c copy output.wav```

Of course, you will need to choose the parameters for your device.
3. If the audio was in AAC format, as the most popular version, then we compile the attached utility:

`gcc aac.c -L. -lfaad -lm -o aacfixer`

(assuming that you compiled the faad2 library and put the files libfaad.a and neaacdec.h in the current directory)

Now run:

```./aacfixer somefile-headers.aac somefile-out-audio.raw```

And after a while the files will appear:

* `<prefix>-pure.wav` (what was decoded)
* `<prefix>-pure-adts.aac` (without recoding)

Sound recovery is done by brute force, so it can be slow.
