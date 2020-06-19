---
layout:     post
title:      FFMPEG框架学习——(6)windows环境下的配置
subtitle:   
date:       2020-06-18
author:     wukurua
header-img: img/linux/post-bg-linux.png
catalog: true
tags:
    - ffmpeg
---

# 一、环境配置 #

`.pro`文件中:

```makefile
INCLUDEPATH += $$PWD/ffmpeg/include

LIBS += $$PWD/ffmpeg/lib/avcodec.lib \
        $$PWD/ffmpeg/lib/avdevice.lib \
        $$PWD/ffmpeg/lib/avfilter.lib \
        $$PWD/ffmpeg/lib/avformat.lib \
        $$PWD/ffmpeg/lib/avutil.lib \
        $$PWD/ffmpeg/lib/postproc.lib \
        $$PWD/ffmpeg/lib/swresample.lib \
        $$PWD/ffmpeg/lib/swscale.lib \
        $$PWD/ffmpeg/sqlite3.lib

DESTDIR = bin
```

项目文件目录结构：

<img src="file:///C:\Users\asus\AppData\Roaming\Tencent\Users\827626083\QQ\WinTemp\RichOle\$23SYFFM[CFJC4SY3QXIJ6M.png" alt="img" style="zoom: 80%;" />

<img src="https://cdn.jsdelivr.net/gh/wukurua/cloudimg@master/img/20200618215705.png" style="zoom: 80%;" />

<img src="https://cdn.jsdelivr.net/gh/wukurua/cloudimg@master/img/20200618215806.png" style="zoom:80%;" />

# 三、查看摄像头名称

终端中:进入ffmpeg所在目录

```shell
C:\WINDOWS\system32>D:
D:\>cd D:\FFMPEG-NEW
```

执行命令`ffmpeg -list_devices true -f dshow -i dummy`：

```shell
D:\FFMPEG-NEW>ffmpeg -list_devices true -f dshow -i dummy
ffmpeg version git-2020-03-06-cfd9a65 Copyright (c) 2000-2020 the FFmpeg developers
  built with gcc 9.2.1 (GCC) 20200122
  configuration: --disable-static --enable-shared --enable-gpl --enable-version3 --enable-sdl2 --enable-fontconfig --ena
ble-gnutls --enable-iconv --enable-libass --enable-libdav1d --enable-libbluray --enable-libfreetype --enable-libmp3lame
--enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libopus --enable-libshine --enable-l
ibsnappy --enable-libsoxr --enable-libtheora --enable-libtwolame --enable-libvpx --enable-libwavpack --enable-libwebp --
enable-libx264 --enable-libx265 --enable-libxml2 --enable-libzimg --enable-lzma --enable-zlib --enable-gmp --enable-libv
idstab --enable-libvorbis --enable-libvo-amrwbenc --enable-libmysofa --enable-libspeex --enable-libxvid --enable-libaom
--enable-libmfx --enable-ffnvcodec --enable-cuda-llvm --enable-cuvid --enable-d3d11va --enable-nvenc --enable-nvdec --en
able-dxva2 --enable-avisynth --enable-libopenmpt --enable-amf
  libavutil      56. 42.100 / 56. 42.100
  libavcodec     58. 73.102 / 58. 73.102
  libavformat    58. 39.101 / 58. 39.101
  libavdevice    58.  9.103 / 58.  9.103
  libavfilter     7. 77.100 /  7. 77.100
  libswscale      5.  6.100 /  5.  6.100
  libswresample   3.  6.100 /  3.  6.100
  libpostproc    55.  6.100 / 55.  6.100
[dshow @ 01021940] DirectShow video devices (some may be both video and audio devices)
[dshow @ 01021940]  "USB2.0 HD UVC WebCam"
[dshow @ 01021940]     Alternative name "@device_pnp_\\?\usb#vid_0bda&pid_57f5&mi_00#6&1d58e0f6&0&0000#{65e8773d-8f56-11
d0-a3b9-00a0c9223196}\global"
[dshow @ 01021940] DirectShow audio devices
[dshow @ 01021940]  "麦克风 (Realtek High Definition Audio)"
[dshow @ 01021940]     Alternative name "@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{671B096E-276E-4DDA-8315
-7FD19E69A270}"
dummy: Immediate exit requested
```

这里有个`"USB2.0 HD UVC WebCam"`，双引号里的就是摄像头的名称。

# 三、和Linux相比之下的改变

```c
AVDictionary *options=NULL;
av_dict_set_int(&options,"rtbufsize",3041280*10,0);//the default buf size of windows is 3041280
AVInputFormat *IfmatC=av_find_input_format("dshow");//windows
res=avformat_open_input(&fmatC,"video=USB2.0 HD UVC WebCam",IfmatC,&options);//window
```

`avformat_open_input()`的第二个参数`url`再Linux中为`/dev/video0`，而在Windows中是`video=USB2.0 HD UVC WebCam`，其中USB2.0 HD UVC WebCam是刚才查看的摄像头名称。