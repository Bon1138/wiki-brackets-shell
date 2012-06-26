## Why am I reading this? 

All of the JavaScript functions in the <code>brackets.fs</code> and <code>brackets.app</code> objects are backed by native code in the application shell. This document describes how it all works.

You should read the [Architecture Overview] (https://github.com/adobe/brackets-shell/wiki/Architectural-Overview) document before reading this document.

## V8 Extensions

The native code is hooked up to JavaScript through a V8 Extension. General information on V8 Extensions (along with other ways to integrate with JavaScript in CEF) can be found on the CEF wiki:
http://code.google.com/p/chromiumembedded/wiki/JavaScriptIntegration

There are quite a few parts to writing a V8 Extension. Complicating things even more is the fact that CEF3 runs in a multi-process environment. JavaScript execution happens in the render process, but all other execution, including UI, happens in the browser process. 

## Step 1: Define the JavaScript 

The JavaScript side of the extension is all done in the <code>appshell/appshell_extensions.js</code> file. This file defines and exports the <code>brackets.fs</code> and <code>brackets.app</code> objects, and also defines the native bindings.

Native bindings are functions marked with the <code>native</code> keyword. For example:

    native function ShowOpenDialog();

These functions do not have a function body, and you do not declare any parameters. The function body is implemented in native code (see the next step), and parameters **can** be passed to the function. 

    ShowOpenDialog(callback, allowMultipleSelection, chooseDirectory, ...);

All async functions must pass a callback function as the first parameter. This callback function will be called once the operation is complete.

## Step 2: Call native code

The binding from JavaScript to native code is done in the <code>appshell/client_app.cpp</code> file. The first step is defining the V8 Extension itself, which is done in the <code>ClientApp::OnWebKitInitialized()</code> function. The call to <code>CefRegisterExtension()</code> binds all native function calls to the <code>AppShellExtensionHandler</code> class.

Whenever a native function is called, <code>AppShellExtensionHandler::Execute()</code> is invoked. This code runs in the render process, so only the most trivial extension code should be executed here. For Brackets, only <code>getElapsedMilliseconds()</code> is handled here. All others calls are passed to the browser process via a <code>CefProcessMessage</code>.

Before sending a message to the browser process, the callback function is saved off so it can be called later. A unique id is used for every message sent.

At this point the render process returns to its normally scheduled processing. The extension continues to execute asynchronously on the browser process.

## Step 3: Off to the browser process

This is where the bulk of the work occurs. Messages sent from the render process are received in the <code>ProcessMessageDelegate</code> class in the <code>appshell/appshell_extensions.cpp</code> file. The <code>OnProcessMessageReceived</code> method checks the arguments, calls platform-specific functions (if needed), and sends a message back to the render process. The argument checking and platform-specific code is different for each message received. 

Platform-specific functions are declared in <code>appshell/appshell_extensions_platform.h</code> and defined in <code>appshell/appshell_extensions_mac.mm</code> (for mac) and <code>appshell/appshell_extensions_win.cpp</code> (for windows).

## Step 4: Back to the render process

Once the message has been processed in the browser process, the callback function needs to be invoked. However, this function must be invoked from the render process, so another <code>CefProcessMessage</code> is created. 

The invokeCallback message is created for you in `appshell_extensions.cpp`. All your code needs to do is add any additional arguments to the <code>responseArgs</code> list. The error argument is added for you, and the message is sent.

## Step 5: Returning to JavaScript

Back in the <code>OnProcessMessageRecieved</code> method in <code>appshell/client_app.cpp</code>, the invokeCallback request is handled. This will call the JavaScript callback function, passing the parameters specified earlier.

## That's a lot of steps! What do I need to implement?

The good news is most extensions only require changes to <code>appshell/appshell_extensions.js</code> and <code>appshell/appshell_extensions.cpp</code> (and platform variants). Execution of your extension will pass through the other steps, but no changes should be required.