PROJECT(sffmpeg)

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
INCLUDE(ExternalProject)
INCLUDE_DIRECTORIES(BEFORE ${CMAKE_BINARY_DIR}/include)
LINK_DIRECTORIES(${CMAKE_BINARY_DIR}/lib)
SET(ENV{PATH} ${CMAKE_BINARY_DIR}/bin:$ENV{PATH})
SET(ENV{PKG_CONFIG_PATH} ${CMAKE_BINARY_DIR}/lib/pkgconfig)

EXTERNALPROJECT_ADD(
  yasm
  URL https://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh yasm
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR}
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  nasm
  URL https://www.nasm.us/pub/nasm/releasebuilds/2.13.03/nasm-2.13.03.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh nasm
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR}
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  zlib
  URL https://github.com/madler/zlib/archive/v1.2.11.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh zlib
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --static
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  openssl
  URL https://www.openssl.org/source/openssl-1.0.2o.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh openssl
  CONFIGURE_COMMAND ./config no-shared no-idea no-mdc2 no-rc5 --prefix=${CMAKE_BINARY_DIR}
  BUILD_COMMAND $(MAKE) depend && $(MAKE)
  INSTALL_COMMAND $(MAKE) install_sw
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  opencore-amr
  URL https://downloads.sourceforge.net/project/opencore-amr/opencore-amr/opencore-amr-0.1.5.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh opencore-amr
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  fdkaac
  URL https://downloads.sourceforge.net/project/opencore-amr/fdk-aac/fdk-aac-0.1.6.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh fdkaac
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  lame
  DEPENDS nasm
  URL https://downloads.sourceforge.net/project/lame/lame/3.100/lame-3.100.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh lame
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-nasm
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  libogg
  URL https://downloads.xiph.org/releases/ogg/libogg-1.3.3.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh libogg
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  speex
  DEPENDS libogg
  URL https://downloads.xiph.org/releases/speex/speex-1.2.0.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh speex
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-sse  --disable-oggtest --with-ogg=${CMAKE_BINARY_DIR}
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  libvorbis
  DEPENDS libogg
  URL https://downloads.xiph.org/releases/vorbis/libvorbis-1.3.6.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh libvorbis
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --disable-oggtest
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  libtheora
  DEPENDS libogg
  URL https://downloads.xiph.org/releases/theora/libtheora-1.2.0alpha1.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh libtheora
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --disable-oggtest --disable-examples --with-ogg=${CMAKE_BINARY_DIR}
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  opus
  URL https://downloads.xiph.org/releases/opus/opus-1.2.1.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh opus
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --disable-doc
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  xvidcore
  DEPENDS yasm
  URL http://downloads.xvid.org/downloads/xvidcore-1.3.5.tar.gz
  URL_HASH SHA256=165ba6a2a447a8375f7b06db5a3c91810181f2898166e7c8137401d7fc894cf0
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh xvidcore
  CONFIGURE_COMMAND cd build/generic && PATH=$ENV{PATH} ./configure --prefix=${CMAKE_BINARY_DIR}
  BUILD_COMMAND cd build/generic && PATH=$ENV{PATH} $(MAKE)
  INSTALL_COMMAND cd build/generic && $(MAKE) install
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  x264
  DEPENDS nasm
  GIT_REPOSITORY git://git.videolan.org/x264.git
  GIT_TAG 7d0ff22e8c96de126be9d3de4952edd6d1b75a8c # master 2018-01-18
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh x264
  CONFIGURE_COMMAND PATH=$ENV{PATH} ./configure --prefix=${CMAKE_BINARY_DIR} --enable-static --enable-pic --enable-lto
  BUILD_COMMAND PATH=$ENV{PATH} $(MAKE)
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  x265
  DEPENDS yasm
  HG_REPOSITORY https://bitbucket.org/multicoreware/x265
  HG_TAG 930e3cc385df412a227b2671856a6ea871696694 # stable 2018-04-19
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh x265
  CONFIGURE_COMMAND cd build/linux && PATH=$ENV{PATH} ${CMAKE_COMMAND} -G "Unix Makefiles" -D CMAKE_INSTALL_PREFIX:PATH=${CMAKE_BINARY_DIR} -D ENABLE_CLI:BOOL=OFF -D ENABLE_SHARED:BOOL=OFF ../../source
  BUILD_COMMAND cd build/linux && PATH=$ENV{PATH} $(MAKE)
  INSTALL_COMMAND cd build/linux && PATH=$ENV{PATH} $(MAKE) install
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  libvpx
  DEPENDS yasm
  URL https://github.com/webmproject/libvpx/archive/v1.7.0.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh libvpx
  CONFIGURE_COMMAND PATH=$ENV{PATH} ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --size-limit=16384x16384 --enable-postproc --enable-multi-res-encoding --enable-temporal-denoising --disable-unit-tests --disable-install-docs --disable-examples
  BUILD_COMMAND PATH=$ENV{PATH} $(MAKE)
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  rtmpdump
  DEPENDS zlib openssl
  GIT_REPOSITORY git://git.ffmpeg.org/rtmpdump
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh rtmpdump
  CONFIGURE_COMMAND ""
  BUILD_COMMAND $(MAKE) PREFIX=${CMAKE_BINARY_DIR} SHARED= XCFLAGS=-I${CMAKE_BINARY_DIR}/include XLDFLAGS=-L${CMAKE_BINARY_DIR}/lib XLIBS=-ldl
  INSTALL_COMMAND $(MAKE) PREFIX=${CMAKE_BINARY_DIR} SHARED= install
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  freetype
  URL https://download.savannah.gnu.org/releases/freetype/freetype-2.9.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh freetype
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static --without-png
  INSTALL_COMMAND $(MAKE) install && ln -s ${CMAKE_BINARY_DIR}/include/freetype2 ${CMAKE_BINARY_DIR}/include/freetype2/freetype
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  fribidi
  URL https://github.com/fribidi/fribidi/releases/download/v1.0.2/fribidi-1.0.2.tar.bz2
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh fribidi
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static --disable-docs
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  expat
  URL https://downloads.sourceforge.net/project/expat/expat/2.2.5/expat-2.2.5.tar.bz2
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh expat
  CONFIGURE_COMMAND PKG_CONFIG_PATH=$ENV{PKG_CONFIG_PATH} ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  fontconfig
  DEPENDS freetype expat
  GIT_REPOSITORY git://anongit.freedesktop.org/fontconfig
  GIT_TAG c78afa906699933e87889895ca2039887943b639 # master 2018-04-19
  URL ${CMAKE_SOURCE_DIR}/vendor/fontconfig-2.13.0.tar.bz2
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh fontconfig
  CONFIGURE_COMMAND NOCONFIGURE=1 ./autogen.sh && PKG_CONFIG_PATH=$ENV{PKG_CONFIG_PATH} ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  harfbuzz
  DEPENDS freetype fontconfig
  URL https://www.freedesktop.org/software/harfbuzz/release/harfbuzz-1.7.6.tar.bz2
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh harfbuzz
  CONFIGURE_COMMAND CFLAGS=-I${CMAKE_BINARY_DIR}/include CPPFLAGS=-I${CMAKE_BINARY_DIR}/include LDFLAGS=-L${CMAKE_BINARY_DIR}/lib ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static --without-glib --without-cairo --with-icu=builtin
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  libass
  DEPENDS yasm freetype fribidi fontconfig harfbuzz
  URL https://github.com/libass/libass/releases/download/0.14.0/libass-0.14.0.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh libass
  CONFIGURE_COMMAND PATH=$ENV{PATH} PKG_CONFIG_PATH=$ENV{PKG_CONFIG_PATH} ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
  BUILD_COMMAND PATH=$ENV{PATH} $(MAKE)
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  libuuid
  URL https://downloads.sourceforge.net/project/libuuid/libuuid-1.0.3.tar.gz
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh libuuid
  CONFIGURE_COMMAND PATH=$ENV{PATH} PKG_CONFIG_PATH=$ENV{PKG_CONFIG_PATH} ./configure --prefix=${CMAKE_BINARY_DIR} --disable-shared --enable-static
  BUILD_IN_SOURCE 1
)
EXTERNALPROJECT_ADD(
  ffmpeg
  DEPENDS yasm opencore-amr fdkaac lame libogg speex libvorbis libtheora opus xvidcore x264 x265 libvpx rtmpdump freetype libass openssl libuuid
  URL https://ffmpeg.org/releases/ffmpeg-3.4.2.tar.bz2
  PATCH_COMMAND ${CMAKE_SOURCE_DIR}/patches/patch-manager.sh ffmpeg
  CONFIGURE_COMMAND PATH=$ENV{PATH} PKG_CONFIG_PATH=$ENV{PKG_CONFIG_PATH} ./configure --prefix=${CMAKE_BINARY_DIR} --datadir=${CMAKE_BINARY_DIR}/etc --disable-shared --enable-static --enable-pic --pkg-config-flags=--static --enable-gpl --enable-version3 --enable-nonfree --disable-doc --disable-debug --disable-ffplay --disable-ffserver --disable-outdevs --disable-hwaccels --disable-autodetect --disable-bzlib --enable-runtime-cpudetect --extra-cflags=-I${CMAKE_BINARY_DIR}/include\ -static --extra-ldflags=-L${CMAKE_BINARY_DIR}/lib --extra-libs=-lstdc++\ -lexpat\ -ldl --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libfdk-aac --enable-libmp3lame --enable-libspeex --enable-libtheora --enable-libvorbis --enable-libx264 --enable-libx265 --enable-libxvid --enable-libvpx --enable-libopus --enable-librtmp --enable-libfreetype --enable-libass --enable-openssl
  BUILD_COMMAND PATH=$ENV{PATH} $(MAKE)
  BUILD_IN_SOURCE 1
)
