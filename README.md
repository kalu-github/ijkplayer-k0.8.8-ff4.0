#
#### 版本
```
linux for ff4.0--ijk0.8.8--20210426--001
```

#
#### 环境
```
deepin24
https://www.deepin.org/zh/download/
```
```
ndk14b
https://github.com/android/ndk/wiki/Unsupported-Downloads
```

#
#### 编译
```
1. 安装相关工具
   apt-get update
   apt-get install git
   apt-get install yasm
```
```
2. 配置系统环境变量
   /etc/profile
   export ANDROID_NDK=/home/kalu/Android/android-ndk-r14b
   export PATH=$ANDROID_NDK:$PATH
   export ANDROID_SDK=/home/kalu/Android/Sdk
   export PATH=${PATH}:$ANDROID_SDK/tools:$ANDROID_SDK/platform-tools
```
```
3. 设置的环境变量生效[ndk-build -v、adb version]
   source /etc/profile
```
```
4. 下载libyuv
   ./init-android-libyuv.sh
```
```
5. 下载soundtouch
   ./init-android-soundtouch.sh
```
```
6. 下载ffmpeg
   ./init-android.sh
```
```
7. 下载openssl
   ./init-android-openssl.sh
```
```
8.  软链module.sh
   cd config
   rm module.sh
   ln -s module-lite.sh module.sh
```
```
9.  编译openssl
   cd android/contrib
   ./compile-openssl.sh clean
   ./compile-openssl.sh all
```
```
10. 编译ffmpeg
   cd android/contrib
   ./compile-ffmpeg.sh clean
   ./compile-ffmpeg.sh all
```
```
11. 编译ijlayer
   cd android
   ./compile-ijk.sh all
```

#
#### 记录1 => 编译问题
```
1. undefined reference to "--disable-ffserver"
   #export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-ffserver"
```
```
2. undefined reference to "--disable-vda"
   #export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-vda"
```
```
3. undefined reference to 'ff_ac3_parse_header'
   export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS –disable-bsf=eac3_core"
```
```
4. compile-ijk.sh 不生成ijkplayer.so、ijksdk.so
   android/ijkplayer/xx/src/main/jin/Android.mk => 末尾新增 => include ../../../../../../ijkmedia/*.mk
```
```
5. ffmpeg 开启neon
   do-comfile-ffmpeg.sh 修改 FF_ASSEMBLER_SUB_DIRS="arm neon"
```
```
6. so大小 如果不需要视频滤镜功能可以关闭
   export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-avfilter"
```
```
7. so输出 地址
   **/android/ijkplayer/ijkplayer-armv7a/src/main/obj/armv7a/
```
```
8. 权限 终端提示权限拒绝
   sudo chmod -R 777 **
```

#
#### 记录2 => 解决可能存在与三方依赖库的so文件名重复冲突
```
1. libijkplayer.so
**/ijkmedia/ijkplayer/Android.mk文件，修改ijkffmpeg、ijksdk和ijkplayer的名称分别为ijkffmpeg***、ijksdk***、ijkplayer***
```
```
2. libijksdl.so
**/ijkmedia/ijksdl/Android.mk文件，修改ijkffmpeg和ijksdk的名称分别为ijkffmpeg***、ijksdk***
```
```
3. libijkffmpeg.so，
**/android/contrib/tools/do-compile-ffmpeg.sh文件，替换其中的两处libijkffmpeg.so，修改成自己想要的名称libijkffmpeg***.so
```
```
4. libijkffmpeg.so，
**/android/ijkplayer/**/src/mian/jni/Android.mk文件，替换其中的两处libijkffmpeg.so和ijkffmpeg.so，修改成自己想要的名称libijkffmpeg***.so
```

#
#### 模块
```
libavutil：
核心工具库，该模块是最基本的模块之一，其它这么多模块会依赖此模块做一些音视频处理操作。
```
```
libavformat： 
文件格式和协议库，该模块是最重要的模块之一，封装了Protocol层、Demuxer层、muxer层，使用协议和格式对于开发者是透明的。
```
```
libavcodec: 
编解码库，该模块也是最重要模块之一，封装了Codec层，但是有一些Codec是具备自己的License的，FFmpeg是不会默认添加，例如libx264,FDK-AAC, lame等库，但FFmpeg就像一个平台一样，可以将其它的第三方的Codec以插件的方式添加进来，然后为开发者提供统一的接口。
```
```
libswrsample：
音频重采样库，可以对数字音频进行声道数、数据格式、采样率等多种基本信息的转换。
```
```
libswscale：
视频压缩和格式转换库，可以进行视频分辨率修改、YUV格式数据与RGB格式数据互换。
```
```
libavdevice：
输入输出设备库，编译ffplay就需要确保该模块是打开的，时时也需要libSDL预先编译，因为该设备播放声音和播放视频使用的都是libSDL库。
```
```
libavfilter:
音视频滤镜库，该模块包含了音频特效和视频特效的处理，在使用FFmpeg的API进行编解码的过程中，直接使用该模块为音视频数据做特效物理非常方便同时也非常高效的一种方式。
```
```
libpostproc:
音视频后期处理库，当使用libavfilter的时候需要打开该模块开关，因为Filter中会使用该库中的一些基础函数。
```
