# Streamtest

This repo contains informations about test I did or do for crowsnest development.\
It should help to make some useful decisions for backends which currently used or will be used

## Current State

[Crowsnest](https://github.com/mainsail-crew/crowsnest.git) currently uses

|   Stream Service   |                     URL                     | mostly written in (also uses) |
| :----------------: | :-----------------------------------------: | :---------------------------: |
|     ustreamer      |     https://github.com/pikvm/ustreamer      |          C (python)           |
| rtsp-simple-server | https://github.com/aler9/rtsp-simple-server |           Go (C++)            |

### How it is implemented, currently

uses a bash script that relies on the application `crudini` to parse the TOML like configuration.\

Please read the code for full explanation.

Boild down _workflow_:

-   MJPG/Snapshots
    [parse config](https://github.com/mainsail-crew/crowsnest/blob/master/libs/configparser.sh) -> [create arguments for ustreamer](https://github.com/mainsail-crew/crowsnest/blob/master/libs/ustreamer.sh) -> run ustreamer -> keep running through while loop.

-   RTSP
    [parse config](https://github.com/mainsail-crew/crowsnest/blob/master/libs/configparser.sh) -> launch ffmpeg -> [launch rtsp-simple-server](https://github.com/mainsail-crew/crowsnest/blob/master/libs/rtspsimple.sh) (preconfigured in [crowsnest-rtsp.yml](https://github.com/mainsail-crew/crowsnest/blob/master/resources/crowsnest-rtsp.yml))

### Pros and Cons

#### Pros

-   Extremly light weight due using bash script
-   Easy to maintain, since everyone with basic bash scripting can modify it.
-   Updates of the scripts can easily done via `git fetch/pull`

#### Cons

-   limited due usage of external programms, like
    -   cut, sed, awk (basicly the whole gnu tool chain for string manipulation)
    -   test (bash builtin)
    -   ustreamer
    -   rtsp-simple-server
    -   crudini
    -   ffmpeg
    -   hard to extend with new features due lack of capabilities using bash script, always 'real' programs has to be involved here.
    -   H264 encoding for RTSP (needed for webrtc) only usable with Raspicams or UVC Cameras with inbuilt H264 Encoder.
    -   need of ffmpeg for rtsp

#### Camera types

|            supported cameras             | unsupported cameras |
| :--------------------------------------: | :-----------------: |
| v4l - uvc, legacy camera stack Raspicams | libcamera, Arducams |

#### Conclusion

Easy to configure for the Enduser, but is more or less limited to MJPEG usage for current existing\
GUI's like Mainsail or Fluidd, Octoprint isnt officially supported but might work also.

As we all know MJPEG is widly spread and relativly old, but work with ease in any browser and backends are easy to obtain. But, its inefficiant in terms of bandwith usage. As High Resolution streaming becomes a defacto standard it uses to much for small devices like Raspberry Pis over WiFi.

Therefor, in my humble opinion, is the only way to solve issues of this kind to use webrtc.
Which brings us to the downsides of what I want.

#### Downsides

We are limited in Hardware encoding on a Raspberry Pi, so we need solutions with more ore less no CPU usage and saving WiFi bandwidth as much as we can.

ffmpeg has great abilities to deliver, what ever format we need, but at the cost of either GPU or CPU usage. Most transcoding with it is done via CPU, which of course isnt what we want to waste because we want to 'save' that resources for fast printing and not video encoding.

This leads to the conclusion all that has that tiny GPU to do. Encoding in h264 is provided by \
OpenMaxIL on buster based OS Images or M2M on bullseye based OS Images.\
Right now I did not test, M2M Capabilities of the precompiled and provided package of ffmpeg.

**WebRTC** Sounds like the ultimate solution, but it is losely defined in terms of connection handling ([see defined here: webrtc.org](https://webrtc.org/))\
but as always "Every Chef uses it's own recipe to create THE soup...".\
Here comes [Janus Gateway](https://janus.conf.meetecho.com/) into play, this seems the used standard webrtc gateway in the industry. Therefor it should be used for our task. Lets define...

#### What we want, our goals should met

-   Resource efficiant transcoding of h264 video streams from different devices for our usecase, such as
    -   V4L UVC Devices, cheap Webcams widly spread since the pandamic.
    -   Raspicams that are designed at times of Legacy Camera Stack, namely Raspicam V1 and V2 Models
    -   Upcoming/Released Models of Raspicam V2.1 and 3
    -   Arducams ( since they grow in popularity, despite the fact they use propriatary drivers/modules, which I dont care a lot, I simply dont like proprietary garbage on OpenSource Platforms like Raspberry Pis...)
-   Usage of GPU instead of CPU on different SBC/Hardware.
-   Fallback options to MJPEG/Snapshots if either cant be provided.
-   As few dependencies as possible should be used
-   Open Source Projects are the main target for possible backends.
-   No License fee/payed or proprietary services/applications will be used!
-   No Docker environments. Don't want to deal with that overhead on regular basis.
-   Small code base that is easily to maintain.
-   Long term should be a language used that is easily to maintain and extend to features we need. Bash is fast enoug to start that backends but dont wont to use the whole gnu toolchain to manipulate strings in variables. Therefor my personal preference would be either Python or Go.
    Go because of the easiness to provide a single, static linked binary without a huge base of dependencies. Python because it has ability to use a Virtual Env where dependencies easily could be implemented due pip (requirements.txt). Not as fast as go but do we really need that kind of performance here? Idk.
-   Precompiled Binaries !!! I have no doubt about doing it in a seperate repo but dont want to compile heavy tasks on a small pi.

#### Evaluate

We want:

-   MJPEG Stream capabilties as fallback option
-   Jpeg snapshots for timelapses
-   webrtc as main stream source for videos
-   rtsp support to restream via OBS or similar.
-   resource efficency at best effort

What I tested, I am fiddling around with

|   Stream Service    |                     URL                      |                                                              Notes                                                              | Mjpeg/Jpeg Snapshots |        RTSP        |       webrtc       |
| :-----------------: | :------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------: | :------------------: | :----------------: | :----------------: |
|      ustreamer      |      https://github.com/pikvm/ustreamer      |                                                 Only h264 sink with CSI devices                                                 |  :heavy_check_mark:  |     :warning:      |        :x:         |
| rtsp-simple-server  | https://github.com/aler9/rtsp-simple-server  |                                                No Needs ffmpeg to deliver stream                                                |         :x:          | :heavy_check_mark: |        :x:         |
|    RTSPtoWebRTC     |    https://github.com/deepch/RTSPtoWebRTC    |                     Needs RTSP Server as source, non standard webrtc implementation, no precompiled binarys                     |         :x:          |        :x:         | :heavy_check_mark: |
|   camera-streamer   |  https://github.com/ayufan/camera-streamer   | AFAIK, works only on Raspberry Pis but not tested yet. Most promising for all requirements, wasnt able to compile on a Pi Zero2 |  :heavy_check_mark:  | :heavy_check_mark: | :heavy_check_mark: |
| rpi-webrtc-streamer | https://github.com/kclyu/rpi-webrtc-streamer |                                 Only works with older Raspicam Modules, not usable for our case                                 |         :x:          |        :x:         | :heavy_check_mark: |
|       go2rtc        |      https://github.com/AlexxIT/go2rtc       |                         Recently discovered by some nice guys in a special group :wink: Have to test!!!                         |      :question:      |     :question:     |     :question:     |
