Title: Debugging WebKit on Android
Date: 2013-09-05 22:30  
Tags: Android, debugging, WebKit 
Slug: 2013-09-05/debugging-webkit-on-android  
Author: kelwin  
Summary: Step-by-step guide for source-level debugging of Android WebKit.


Debugging is the best way to understand crash of WebKit, which will help a lot to write exploits. Since WebKit is open-source, we could achieve source-level debugging. Now let me introduce of the steps. 

## Building Android from Source
No matter whether you use emulator or real device(Nexus series), we should get corresponding symbols and sources. Building the whole system yourself will help to generate all the stuff needed - symbols and sources for all the binaries and shared libraries. You can follow the [official guide](http://source.android.com/source/building.html) and get the emulator images or flash your device. Symbols will be generated in the directory `out/target/product/generic/symbols`.

I've compiled an emulator of Android 4.0.4 and get all the symbols needed in this [archive](http://pan.baidu.com/share/link?shareid=1555478836&uk=3660622653).

## Preparing Symbols and Source Code
In most cases, we need symbols of  `app_process`, `libc.so`, `libdvm.so`, `libstdc++.so`, `libwebcore.so` and `linker`. `app_process` is the process that every Android application forks from. For source code, we only need `external/webkit`.

Copy the symbols to the directory `symbols`and the source code of WebKit to the directory `source`. Thus we get the symbols and source code done. I've already get this done in the archive above.

## Debugging WebKit

Start your emulator or device. If you use my archive, you can create an avd of Android 4.0.4 and run `$ ./startemu.sh <avd name>` to start the emulator.

Open Android Browser(or any app using WebView) and start gdbserver:

    $ adb shell
    * daemon not running. starting it now on port 5037 *
    * daemon started successfully *
    # ps | busybox grep browser
    app_22    709   37    149728 46064 ffffffff 400113c0 S com.android.browser
    # gdbserver :1337 --attach 709
    Attached; pid = 709
    Listening on port 1337

Open port forwarding:

    $ adb forward tcp:1337 tcp:1337

Start gdb, set symbol and source paths, and start remote debuging:

    $ arm-linux-androideabi-gdb app_process
    (gdb) set solib-search-path symbols
    (gdb) directory sources
    (gdb) target remote :1337
 
Now everything works like a charm!

## References

1. [How C/C++ Debugging Works on Android](http://mhandroid.wordpress.com/2011/01/25/how-cc-debugging-works-on-android/)
2. [An Android Hacker's Journey](http://cansecwest.com/slides/2013/An%20Android%20Hacker's%20Journey-%20Challenges%20in%20Android%20Security%20Research.pptx)

