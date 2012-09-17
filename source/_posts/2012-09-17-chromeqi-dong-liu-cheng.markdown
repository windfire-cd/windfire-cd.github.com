---
layout: post
title: "chrome启动流程"
date: 2012-09-17 15:44
comments: true
categories: chrome
---

##主函数

在*src/chrome/app/*中，对应于不同平台存在不同的主函数入口文件，比如

- *chrome_exe_main_gtk.cc*
- *chrmoe_exe_main_win.cc*
- *chrome_exe_main_mac.cc*

以*chrome_exe_main_gtk.cc*为例

```cpp chrome_exe_main_gtk.cc
#include "build/build_config.h"

// The entry point for all invocations of Chromium, browser and renderer. On
// windows, this does nothing but load chrome.dll and invoke its entry point in
// order to make it easy to update the app from GoogleUpdate. We don't need
// that extra layer with on linux.

extern "C" {
	int ChromeMain(int argc, const char** argv);
}

int main(int argc, const char** argv) {
	  return ChromeMain(argc, argv);
}
```

其中ChromeMain(int,char*[])定义在同一目录下的chrome_main.cc中

```cpp chrome_main.cc
#include "chrome/app/chrome_main_delegate.h"

#include "content/public/app/content_main.h"

#if defined(OS_WIN)
#define DLLEXPORT __declspec(dllexport)

// We use extern C for the prototype DLLEXPORT to avoid C++ name mangling.
extern "C" {
DLLEXPORT int __cdecl ChromeMain(HINSTANCE instance,
sandbox::SandboxInterfaceInfo* sandbox_info);
}
#elif defined(OS_POSIX)
extern "C" {
__attribute__((visibility("default")))
int ChromeMain(int argc, const char** argv);
}
#endif

#if defined(OS_WIN)
DLLEXPORT int __cdecl ChromeMain(HINSTANCE instance,
sandbox::SandboxInterfaceInfo* sandbox_info) {
ChromeMainDelegate chrome_main_delegate;
return content::ContentMain(instance, sandbox_info, &chrome_main_delegate);
#elif defined(OS_POSIX)
int ChromeMain(int argc, const char** argv) {
ChromeMainDelegate chrome_main_delegate;
return content::ContentMain(argc, argv, &chrome_main_delegate);
#endif
}
```
*ChromeMainDelegate*继承自*ContentMainDelegate*主要提供启动相关函数以及进程
调用处理。其只是将父类函数提供一个简单的实现

其中*content::ContentMain(argc, argv, &chrome_main_delegate);*
负责主要循环，初始化环境，启动Run函数，退出并返回错误值，其定义如下

```cpp content_main.cc

#include "content/public/app/content_main.h"

#include "base/memory/scoped_ptr.h"
#include "content/public/app/content_main_runner.h"

namespace content {

#if defined(OS_WIN)
int ContentMain(HINSTANCE instance,
                sandbox::SandboxInterfaceInfo* sandbox_info,
				                ContentMainDelegate* delegate) {
#else
	int ContentMain(int argc,
	                const char** argv,
					                ContentMainDelegate* delegate) {
#endif  // OS_WIN

		  scoped_ptr<ContentMainRunner> main_runner(ContentMainRunner::Create());

		    int exit_code;

#if defined(OS_WIN)
  exit_code = main_runner->Initialize(instance, sandbox_info, delegate);
#else
    exit_code = main_runner->Initialize(argc, argv, delegate);
#endif  // OS_WIN

	  if (exit_code >= 0)
		      return exit_code;

			    exit_code = main_runner->Run();

				  main_runner->Shutdown();

				    return exit_code;
	}

}  // namespace content
```


