---
layout: post
title: Roll out my own Javascript native Api
---

##### This post is inspired in [How to roll out your own Javascript API with V8](http://syskall.com/how-to-roll-out-your-own-javascript-api-with/).

# Introduction

- We are going to compile V8 Javascript Engine v4.3
- Modify shell.cc example in order to display desktop notifications through GTK library.
- This example was runed in the following environment:
  - SO: Ubuntu 14.04.2 LTS
  - CPU Architecture: x86_64
  - CPU op-mode(s): 32-bit, 64-bit
  - Byte Order: Little Endian



![js-alert]({{ site.baseurl }}/images/js-alert.png)

# Compiling v8

- Clone [V8 repository](https://github.com/v8/v8-git-mirror)
- Add the behind lines in `.git/config`, and run `git fetch` in order to fetch remote references:

```
  fetch = +refs/branch-heads/*:refs/remotes/branch-heads/*
  fetch = +refs/tags/*:refs/tags/*
```

- Checkout 4.3 version `git checkout -b 4.3 -t branch-heads/4.3`
- Before compile the engine, if not have installed **icu/i18n** library, read the [Louis P. Santillan response](https://groups.google.com/forum/#!msg/v8-users/KfS2XDdZQkc/GmQOUmYldLEJ). You install either **icu/i18n** library or compile using `i18nsupport=off` parameter.
- Set environment variable `export PKG_CONFIG_PATH="$PKG_CONFIG_PATH:/usr/lib/x86_64-linux-gnu/pkgconfig"`
- Build the engine:
  ```
  make dependencies & make x64.release
  ```
- Should have a folder out/x64.release with the results:

```
v8/out/x64.release/obj.target/tools/gyp/libv8_libbase.a
v8/out/x64.release/obj.target/tools/gyp/libv8_libplatform.a
v8/out/x64.release/obj.target/tools/gyp/libv8_snapshot.a
v8/out/x64.release/obj.target/tools/gyp/libv8_base.a
```
# Running the alert API

- Should create the following structure:

```
jsnotify/
  |-- deps/  # third party code
  |   `-- v8  # move your v8 folder here
  `-- src/ # our code goes here
      `-- jsnotify.cpp
```

- Copy the `deps/v8/samples/shell.cc` content into your `src/jsnotify.cpp`. Or you can clone my repository:

```bash
> $ git clone git@github.com:charlyraffellini/jsnotify.git
```

- Create your own alert() API function:

```c++
// INSERT THIS BEFORE int RunMain(int argc, char* argv[]) {
// We need those two libraries for the GTK+ notification
#include <gtkmm.h>
#include <libnotifymm.h>
void Alert(const v8::FunctionCallbackInfo<v8::Value>& args);

// INSERT THIS AT END OF FILE
// The callback that is invoked by v8 whenever the JavaScript 'alert'
// function is called.  Displays a GTK+ notification.
void Alert(const v8::FunctionCallbackInfo<v8::Value>& args) {
  v8::String::Utf8Value str(args[0]); // Convert first argument to V8 String
  const char* cstr = ToCString(str); // Convert V8 String to C string

  Notify::init("Basic");
  // Arguments: title, content, icon
  Notify::Notification n("Alert", cstr, "terminal");
  // Display notification
  n.show();

  return;
}

```

- Bind your function to the global context:

```c++
// INSERT AFTER v8::Handle<v8::ObjectTemplate> global = v8::ObjectTemplate::New();
// Bind the global 'alert' function to the C++ Alert callback.
global->Set(v8::String::NewFromUtf8(isolate, "alert"),
            v8::FunctionTemplate::New(isolate, Alert));
```

- Now compile and link your code:

```bash
> $ g++ src/jsnotify.cpp -o jsnotify -Ideps/v8 -lpthread `pkg-config --cflags --libs gtkmm-2.4 libnotifymm-1.0` -Wl,--start-group deps/v8/out/x64.release/obj.target/{tools/gyp/libv8_{base,libbase,snapshot,libplatform},third_party/icu/libicu{uc,i18n,data}}.a -Wl,--end-group -lrt -std=c++0x -ldl
```

- Run your API:

```bash
> $ ./jsnotify
V8 version 4.3.61.38 [sample shell]
> alert('alert me');
```


---

References:

- [How to roll out your own Javascript API with V8](http://syskall.com/how-to-roll-out-your-own-javascript-api-with/)

- [V8 Hello World Shell](https://developers.google.com/v8/get_starteds) 