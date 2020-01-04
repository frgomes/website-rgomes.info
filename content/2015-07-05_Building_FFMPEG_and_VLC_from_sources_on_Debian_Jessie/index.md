+++
title = "Building FFMPEG and VLC from sources on Debian Jessie"
date = 2015-07-05T22:04:00Z
[taxonomies]
categories = ["articles", "recipe"]
tags = ["linux", "ffmpeg", "vlc", "build", "sources", "explained"]
+++
_Debian Jessie and older versions do not come with ``ffmpeg``. Instead, it comes with ``libav``, which is a fork from ``ffmpeg``. HOwever, there are situations when you find yourself in the need of ``ffmpeg``. In this article, we explain how you can build ``ffmpeg`` from sources and we will also build our own ``vlc`` which will be capable of employing our own build version of ``ffmpeg`` behind the scenes._

Start by upgrading your packages and installing ```build-essential```:

```bash
#!/bin/bash

sudo apt-get update
sudo apt-get install build-essential 
```

Download ``ffmpeg`` and ``vlc``:

```bash
#!/bin/bash

mkdir -p $HOME/sources/software
cd $HOME/sources/software
wget http://ffmpeg.org/releases/ffmpeg-2.7.1.tar.gz
wget http://get.videolan.org/vlc/2.2.0/vlc-2.2.0.tar.xz
```

Install build dependencies ``ffmpeg`` needs:

```bash
#!/bin/bash

sudo apt-get install libmp3lame-dev libvorbis-dev libtheora-dev \
    libspeex-dev yasm pkg-config libfaac-dev libopenjpeg-dev \
    libx264-dev libass-dev
```

Compile and install ``ffmpeg``:

```bash
#!/bin/bash

tar xpf ffmpeg-2.7.1.tar.gz
cd ffmpeg-2.7.1/
./configure --enable-gpl --enable-postproc --enable-swscale \
    --enable-avfilter --enable-libmp3lame --enable-libvorbis \
    --enable-libtheora --enable-libx264 --enable-libspeex \
    --enable-shared --enable-pthreads --enable-libopenjpeg \
    --enable-libfaac --enable-nonfree --enable-libass
make
sudo make install
sudo /sbin/ldconfig
```

Install build dependencies ``vlc`` needs:

```bash
#!/bin/bash

sudo apt-get build-dep vlc
```

Compile and install ``vlc``:


```bash
#!/bin/bash

./configure --prefix=/usr/local --with-ffmpeg-tree=/usr/local \
  --enable-x11 --enable-xvideo --disable-gtk \
  --enable-sdl --enable-ffmpeg --with-ffmpeg-mp3lame \
  --enable-mad --enable-libdvbpsi --enable-a52 --enable-dts \
  --enable-libmpeg2 --enable-dvdnav --enable-faad \
  --enable-vorbis --enable-ogg --enable-theora --enable-faac\
  --enable-mkv --enable-freetype --enable-fribidi \
  --enable-speex --enable-flac --enable-livedotcom \
  --with-livedotcom-tree=/usr/lib/live --enable-caca \
  --enable-skins --enable-skins2 --enable-alsa --disable-kde\
  --disable-qt --enable-wxwindows --enable-ncurses \
  --enable-release
make
sudo make install
```

Remember that now you have two versions of ``vlc`` installed in your system. The version of ``vlc`` we installed employs ``ffmpeg`` under the hood from ``/usr/local``, whilst the version which comes with Debian employs ``libav``.


----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
