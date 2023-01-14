# Streamtest

This repo contains informations about test I did/do for crowsnest development.\
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
