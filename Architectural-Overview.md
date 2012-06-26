## Background

The Brackets application shell is built using the [Chromium Embedded Framework (CEF)] (http://http://code.google.com/p/chromiumembedded/). CEF is an open source web browser control based on Google Chromium. 

The Brackets app is based on the cefclient sample application provided by CEF. The app itself is really quite simple -- it hosts the CEF control to display the HTML/CSS/JavaScript for Brackets, and provides some extensions required for Brackets to work. These extensions include filesystem access, file dialogs, and code for launching and connecting to the Live Development browser.

### CEF vs. Chromium Content Shell

CEF3 (the version of CEF used by Brackets) is based on the new [Chromium Content API] (http://www.chromium.org/developers/content-module/content-api). The Chromium project also includes a [Content Shell] (http://src.chromium.org/viewvc/chrome/trunk/src/content/shell/) project, which is a minimal application that embeds the Chromium Content API. 

We looked at both of these for Brackets. At first glance, the cefclient app and the Content Shell app look very similar. However, when you dig a bit deeper, the differences become apparent. Content Shell is a very thin layer on top of the content API. It is intended to be the simplest possible app that uses the Content API. CEF, on the other hand, is designed to be a robust embeddable control. CEF provides a high-level API that insulates clients from API churn in the Content API. CEF also provides some key high-level functionality like context menus and support for opening the Developer Tools window from within the application.

## Processes and Threads

Chromium (and CEF3) use a [multi-process architecture] (http://www.chromium.org/developers/design-documents/multi-process-architecture). The browser process is the main process that runs the UI. The render process is responsible for doing all layout, rendering and JavaScript execution. There are additional processes for plugins, GPU, and Utilities. The majority of the work happens in the browser and render processes.

Within each process there are a number of threads. If you are doing UI work, you must make sure you are on the UI thread. See the [CEF Architecture Document] (http://code.google.com/p/chromiumembedded/wiki/Architecture) for more details on processes and threads.

## Main Classes

Here are some of the main classes used in the app.

### ClientApp

This is the main class that runs in the render process. It is responsible for:

* Initializing V8 extensions (in `OnWebKitInitialized()`). The extension entry point is the `AppShellExtensionHandler` class, defined in client_app.cpp. All native JavaScript functions invoke the `AppShellExtensionHandler::Execute()` method. 
* Handling process messages from the browser process (in `OnProcessMessageReceived()`). Currently there are two message that get handled here:
  * `invokeCallback` - called to invoke the asynchronous callback functions
  * `executeCommand` - called to execute a command via JavaScript

### ClientHandler

This is the main class that runs in the browser process. Only one instance of this class is created for **all** browser windows. This class is responsible for:

* Managing browser window instances
* Responding to loading, error, and console messages
* Initiating JavaScript commands (which are forwarded to the render process for sending)
* Handling process messages from the render process. The majority of the messages are native JavaScript function calls, which are handled in appshell_extensions.cpp.

### CefBrowser/CefBrowserHost

These are key CEF classes that are used throughout the program. Instances of the CefBrowser class are available in both the render and browser process, but CefBrowserHost is only available in the browser process.

### CefProcessMessage

All messages passed between the browser and render process are wrapped up in a CefProcessMessage.

### CefV8Context

A CefV8Context corresponds to a unique JavaScript execution environment. Every browser frame has a CefV8Context. You must be in a context before working with CefV8Values or calling CefV8Functions.

### CefV8Value

Represents a V8 (JavaScript) value. This can be a scalar type like number or String, or an Object/Function/Array.

### Other files of interest

* **appshell_extensions.js**: the JavaScript wrapper for all of the V8 extensions
* **cefclient.cpp**: the main entry point for the browser process
* **appshell_extensions.cpp**: the handler for native JavaScript functions.
* **appshell_extensions_mac.mm/appshell_extensions_win.cpp**: platform-specific code for V8 extensions

