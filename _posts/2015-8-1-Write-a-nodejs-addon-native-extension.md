---
layout: post
title: Write a nodejs addon native extension
---

# Introduction
- We are going to write and build a nodejs addon.
- Use libnotify packages in order to show desktop notifications.
- [Source code](https://github.com/charlyraffellini/nan-addon-node-notify)

- This example was runed in the following environment:
  - SO: Ubuntu 14.04.2 LTS
  - nodejs v0.12.7
  - node-gyp v2.0.2
  - Python 2.7.6
  - gcc 4.8.4
  - CPU Architecture: x86_64
  - CPU op-mode(s): 32-bit, 64-bit
  - Byte Order: Little Endian

---

# Outstanding

- `binding.gyp` is a JSON file that tell `node-gyp` how to build our addon.
  
  - The "include_dirs" array tells the compiler where to find the `nan.h` header file used by NAN, and the `libnotify/notify.h` header file.
  
    ```json
          "include_dirs": [ "<!(node -e \"require('nan')\")",
          "<!@(pkg-config libnotify --cflags-only-I | sed s/-I//g)"
          ]
    ```
  
  - The `libraries` array tells the compiler where to find the `libnotify` library:

    ```json
          "libraries": [
          "<!@(pkg-config libnotify --libs)"
          ]
    ```

    > NAN is *"Native Abstractions for Node.js" and exists mainly because the native V8 API has become nearly impossible to write stable code against. The choice is between targeting Node 0.10 and prior *or* Node 0.11 and after, but maintaining compatibility for both is very difficult without getting lost in macro soup. NAN exists to manage this pain and provides a single, and mostly stable, interface to dealing with both V8, Node.js and libuv.


    > Every _binding.gyp_ file has an array of targets, each target defines a native *module* that node-gyp will build. Each target requires a `"target_name"`, this name is used to compile a binary named *`<name>.node`*.

- This construct uses a C++ macro from NAN to expose a public *method* that is accessible via V8 to your JavaScript:
  
    ```cpp
    NAN_METHOD(Send) {
        // ...
    }
    ```

    - The `NAN_MACRO` hides the complexity between signature for methods that are exposed in Node.js 0.10 (and prior) and 0.11 (and later).

- NAN include heplers for return values like `NanReturnUndefined()`.

- Export methods (equivalent to `module.exports.send = Send` in JavaScript side)

    ```cpp
    void Init(Handle<Object> exports) {
      exports->Set(NanNew("send"), NanNew<FunctionTemplate>(Send)->GetFunction());
    }
    ```

    - `export` object received in `Init()` functions is the same as the received in a JavaScript module as `module.exports`.
.   
    - In order to set properties We use `NanNew()` creates a new V8 object, in this case a `String`.

    - `NanNew<FunctionTemplate>(Print)->GetFunction()` return a reference to the method so that we can expose it to JavaScript.

- Expose the `Init()` function to nodejs:

    ```c++
    NODE_MODULE(modulename, InitFunction)
    ```

    - `modulename` is the target name in **binding.gyp** file.

    - `InitFunction` is the function that receive the `exports` object.


- Access to function arguments when you use `NAN_METHOD` macro via `args` array inside the method.

    - To use `v8::Number` from an argument write `Local<Number> num = args[2].As<Number>();`.

    - `Local<Number>` is a object *handle* that allow you behave properly in V8 side.

    - To decode a `Local<String> str`  as a Utf8 pointer use `*String::Utf8Value(str)`.

    > The Local<Type> construct is required to wrap up the raw type object in a handle that can safely interact with the V8 runtime. The handle is used to attach to the garbage collector and automatically clean up the object when we fall out of scope. We can also use the handle to perform conversions and comparisons. In general we only interact with V8 data types when they are wrapped in a handle.

- V8 scopes

    - C++ doesn't have automatic garbage collection but Javascript does.

    > To replicate the concept of function scoping of variables, V8 introduces a HandleScope. When you declare a `HandleScope` at the top of a C++ function, all V8 objects created within that function will be attached to that scope in the same way that a function in JavaScript will capture new variables declared within it.
    
    > To declare one of these scopes in your code, use the NAN `NanScope()` call at the top of your function. When your function ends, the scope can then pass on the objects to the garbage collector if they have not been passed outside of the scope.

- The rest is invoke the **libnotify** functions.

    ```cpp
    int timeout = NOTIFY_EXPIRES_DEFAULT;
    notify_init("node-notify");

    Gtknotify* gtknotify_instance = new Gtknotify();
    gtknotify_instance->title = "Node.js";
    gtknotify_instance->icon = "terminal";
    gtknotify_instance->n = notify_notification_new(gtknotify_instance->title.c_str(), "", gtknotify_instance->icon.c_str());

    Local<String> str = args[0].As<String>();

    notify_notification_update(gtknotify_instance->n, gtknotify_instance->title.c_str(), *String::Utf8Value(str), gtknotify_instance->icon.c_str());
    notify_notification_set_timeout(gtknotify_instance->n, timeout);
    notify_notification_show(gtknotify_instance->n, NULL);
    ```

--- 

# References

- [V8 Data and Structures documentation](https://v8docs.nodesource.com/)
- [Nodejs addons](https://nodejs.org/api/addons.html)
- [nodeschool](http://nodeschool.io/)
- [Goingnative](https://github.com/workshopper/goingnative) including documentation about [call the callback function](https://github.com/workshopper/goingnative/blob/9ceaddaf9e4ade1cce46c5e821881045f0aa748b/exercises/call_me_maybe/problem.md) and [Node.js working threads](https://github.com/workshopper/goingnative/blob/9ceaddaf9e4ade1cce46c5e821881045f0aa748b/exercises/offloading_the_work/more.md) (UV\_THREADPOOL\_SIZE environment variable).
- [How to write your own native Node.js extension](http://syskall.com/how-to-write-your-own-native-nodejs-extension/)
- [node addon examples](https://github.com/nodejs/node-addon-examples)
