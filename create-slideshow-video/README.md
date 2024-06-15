# Memorial slideshow

This documents the steps I made to create a slideshow video for my dad's memorial service.

It's unclear whether the church where the memorial service will be held's computer connected to the projector has internet connectivity. It's also unclear what software it has to show a slideshow from a folder of images. To work around this, I thought the simplest way to handle this would be to just make a video that can be played with one click. For a long-running slideshow, just make a video that repeats the images multiple times. 
 
I originally planned to use Google Photo's highlight video feature to make a slideshow video for the memorial service. However, the service only allows you to create a highlight video from your own photos, not from shared albums. Instead, I downloaded the album images and processed them with ImageMagick and ffmpeg to make the videos I needed.

## Commands

To make the long slideshow video:

```
# Clean up filenames to replace characters that will cause problems with ffmpeg with underscores
mkdir renamed
rsync -avh ./src/ ./renamed/
rename "s/[\s'~()]+/_/g" renamed/*
# Make images 1920 x 1080, which is a 16:9 aspect ratio
# Also convert all of them to jpg in case ffmpeg has issues with HFIC
# See https://imagemagick.org/script/command-line-processing.php
mkdir resized
mogrify -auto-orient -path resized -thumbnail 1920x1080 -background black -gravity center -extent 1920x1080 -format jpg renamed/*
# Create an input file for ffmpeg that displays all the images for 3 seconds
find resized -name '*.jpg' | sed 's/^/file /' | sed 's/$/\noutpoint 3/' > in.txt
# Create a video from the images
ffmpeg -f concat -i in.txt -framerate 1 -c:v libx264 -shortest -r 30 -pix_fmt yuv420p slideshow.mp4
# Loop the video multiple times
ffmpeg -stream_loop 7 -i slideshow.mp4 -c copy 001_slideshow_long.mp4
```

Once the long slideshow video is done, you can make a version with a song playing in the background clipped to the length of the song:

```
ffmpeg -f concat -i short.txt -i ../audio/01-03\ I\ Think\ Of\ You.flac -framerate 1 -c:v libx264 -c:a aac -shortest -r 30 -pix_fmt yuv420p 003_final_song_i_think_of_you.mp4
```

## References

- [How to create a video from images with FFmpeg?](https://stackoverflow.com/questions/24961127/how-to-create-a-video-from-images-with-ffmpeg)

