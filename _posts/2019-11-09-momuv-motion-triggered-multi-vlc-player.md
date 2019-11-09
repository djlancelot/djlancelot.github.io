---
layout: post
title:  "MoMuV - Motion-triggered Multi-VLC player"
date:   2019-11-09 00:41:06 -0800
categories: media 
---
MoMuV is a helper app that uses a video capture device such as a webcam or an IP camera and starts one or more video players simultaneously to play an animation, such as a Holiday projection.

![Halloween projection](/images/momuv-window.jpg)

## How to use MoMuV?

To use MoMuV you need to have [VLC Player](https://www.videolan.org/vlc/index.html) on your machine. 
After downloading the source code, install the requirements from the source directory using `pip3 install -f requirements.txt`. When the installation is complete, the script can be run like:
```
python players.py -v "C:\Program Files (x86)\VideoLAN\VLC\vlc.exe" --stop 80 --sensitivity  150000 -c "rtsp://user:pass@192.168.50.150/live" -i C:\halloween2019\hweenall1s.mp4
```
Where 
- `"C:\Program Files (x86)\VideoLAN\VLC\vlc.exe"` is the path to the VLC executable, 
- `80` seconds is the duration of the video. After 80s the video will pause and wait for motion
- `150000` is the minimal motion needed to trigger the animation (actual values are printed on the command line for calibration)
- `"rtsp://user:pass@192.168.50.150/live"`
is the URI to the IP camera. Try 0 for a webcam
- `C:\halloween2019\hweenall1s.mp4` is the video's location. The `-i` option can be repeated for multiple videos.

Read more about the usage on the [project page](https://github.com/djlancelot/momuv).

## Why was MoMuV made?
When we go to Universal Studios, I am always amazed how they bring practical movie tricks to life at their entertainment center. Although it looks really cool at the theme park, I believe that these tricks can be executed at home too with a help of a small video projector and a computer with a webcam. 

For Halloween, I decided to create a small animation and needed a tool to play the animation when trick or treaters arrive at our place. The video was projected on the window against an old, white flat sheet (shower linen works too). I needed a tool to play the animation automatically when motion is detected. Initially I split the animation into two parts to use a TV as a primary screen and use the projector to display additional content so I built this requirement into the application.
Unfortunatelly, when doing my research, I couldn't find any freely available tools to achieve my goals, so I started to think about other ways solutions.

## How does MoMuV work?
 I like VLC player and FFMPEG and was thinking about integrating it directly into my application, but I only had a limited time for executing this project. I've found an interesting [hack by Marios Zindilis](https://zindilis.com/blog/2016/10/23/control-vlc-with-python.html) and I started working with that. Loosely coupling [VLC](https://www.videolan.org/vlc/index.html) with the application seemed appropriate as I didn't want to struggle with writing a platform independent video player with a GUI. I've also found out that the [opencv library](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_tutorials.html) has a very good video capture integration and I started using that to open a webcam and check if the difference between consecutive frames are significant enough to start the animation. (The app prints the detected motion level when idle.) 

When the motion level is high enough, the video starts rolling and plays for the period defined in the input parameters. It pauses the video after that so make sure the video's length is greater than the value defined. When motion is detected again, the video starts from the beginning. When I write video it can mean more videos too as the number of video player windows depend on the input parameters. 

That is all about MoMuV. Make sure to [check out the project page](https://github.com/djlancelot/momuv) and give it a try.
