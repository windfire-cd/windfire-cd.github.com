---
layout: post
title: "Chrome 下载编译"
date: 2012-08-28 10:33
comments: true
categories: chrome
---

##安装依赖并获取代码

1. 确认可以解压.tgz类型的文件
2. 下载代码[source tarball](http://chromium-browser-source.commondatastorage.googleapis.com/chromium_tarball.html)
3. 确认代码放置的分区空间足够
4. 解压代码
5. 安装[*depot_tools*](http://www.chromium.org/developers/how-tos/install-depot-tools)
6. 如果是ubuntu系统需要运行下面
	
	```bash
	$cd /path/to/chromium/src
	$sudo ./build/install-build-deps.sh
	```
7. 更新代码

	```bash
	$ gclient sync --force
	```

具体参见[Get the code](http://www.chromium.org/developers/how-tos/get-the-code)

##安装clang依赖

因为chrome编译很慢，这里尝试利用clang加快编译速度以及提高编译质量

```bash
$tools/clang/scripts/update.sh
```


##编译

###gcc

```bash
$./build/gyp_chromium
$make chrome -j4
```

###clang

```bash
$GYP_GENERATORS=ninja GYP_DEFINES=clang=1 ./build/gyp_chromium
$ninja -C out/Debug chrome #fast
```
或者

```bash
$GYP_GENERATORS=make GYP_DEFINES=clang=1 ./build/gyp_chromium
$make chrome -j4  # 4: Number of cores, change accordingly
```

[chrome clang](http://code.google.com/p/chromium/wiki/Clang)

##问题

**nacl超时**

在进行更新代码操作时可能会遇到

```bash
download_nacl_toolchain.py  timeout
```

如果没有下载完就进行编译，可能会遇到
```
LASTCHANGE is needed
```

这样的错误。

需要尝试重新更新代码

或者在build/common.gyi中将*'disable_nacl%'%: 0*置为1(这种方法是官网在编译
chrome os时超时的解决办法，未经尝试)


**webkit的svn超时**

一种方法：如果不需要webkit中的layouttest可以在.gclient中将其注销

```
solutions = [
{ "name"        : "src",
	"url"         : "https://src.chromium.org/chrome/trunk/src",
	"deps_file"   : "DEPS",
	"managed"     : True,
	"custom_deps" : {
		"src/third_party/WebKit/LayoutTests": None,
		"src/content/test/data/layout_tests/LayoutTests": None,
		"src/chrome_frame/tools/test/reference_build/chrome": None,
		"src/chrome_frame/tools/test/reference_build/chrome_win": None,
		"src/chrome/tools/test/reference_build/chrome": None,
		"src/chrome/tools/test/reference_build/chrome_linux": None,
		"src/chrome/tools/test/reference_build/chrome_mac": None,
		"src/chrome/tools/test/reference_build/chrome_win": None,
	},
	"safesync_url": "",
},
	]
```

另一种方法：可以人工下载webkit然后将其替换到chrome工程中去。


##参考

[Chromium Project](http://www.chromium.org/Home)
[chrome clang](http://code.google.com/p/chromium/wiki/Clang)
[Get the code](http://www.chromium.org/developers/how-tos/get-the-code)
