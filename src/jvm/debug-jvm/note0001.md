
相关连接


[https://hunterzhao.io/post/2018/01/29/compile-openjdk10-source-code-on-mac/](https://hunterzhao.io/post/2018/01/29/compile-openjdk10-source-code-on-mac/)

[https://www.cnblogs.com/JunFengChan/p/9266033.html](https://www.cnblogs.com/JunFengChan/p/9266033.html)


在

README-builds.html写的很清晰，

```
checking cups/ppd.h usability... yes
checking cups/ppd.h presence... yes
checking for cups/ppd.h... yes
configure: error: Could not find freetype! You might be able to fix this by running 'brew install freetype'.
/Users/gsh/zopenjdk/master-10/make/autoconf/generated-configure.sh: line 82: 5: Bad file descriptor
configure exiting with result code 1

```

这个问题 耽误了很久 

用brew 不断重新安装 

后来根据安装 readme 中的提示

```  

External Library Requirements
Different platforms require different external libraries. In general, libraries are not optional - that is, they are either required or not used.

If a required library is not detected by configure, you need to provide the path to it. There are two forms of the configure arguments to point to an external library: --with-<LIB>=<path> or  --with-<LIB>-include=<path to include> --with-<LIB>-lib=<path to lib>. The first variant is more concise, but require the include files an library files to reside in a default hierarchy under this directory. In most cases, it works fine.

As a fallback, the second version allows you to point to the include directory and the lib directory separately.

FreeType
FreeType2 from The FreeType Project is required on all platforms. At least version 2.3 is required.

To install on an apt-based Linux, try running sudo apt-get install libcups2-dev.
To install on an rpm-based Linux, try running sudo yum install cups-devel.
To install on Solaris, try running pkg install system/library/freetype-2.
To install on macOS, try running brew install freetype.
To install on Windows, see below.
Use --with-freetype=<path> if configure does not properly locate your FreeType files.

```








































