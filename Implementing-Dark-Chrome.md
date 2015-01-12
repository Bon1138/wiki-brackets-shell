# Introduction

Brackets' implementation of a Dark Chrome is pretty much how the [Proposal](https://github.com/adobe/brackets-shell/wiki/Architecture-Proposal----Windows-Chrome-Theming) stated it would be implemented with a base class `cef_window` which `cef_dark_window` and `cef_host_window` and `cef_main_window` were derived.

# Object List

|Name | Purpose/Use |
| ----| ----------- |
| cef_window | a generic window object which has features to subclass an existing `HWND` and has a `Create` method to create a new window |
| cef_dark_window | a window object which draws a custom dark window frame |
| cef_host_window | a window object which has an embedded cef window |
| cef_popup_window | a window object which is a popup window created with `window.create()` in javascript |
| cef_main_window | a window object which is created at startup to host Brackets |

NOTE: cef_registry encapsulates all of the registry access used throughout brackets but it is just a collection of high-level APIs with no objective.

# Fail

The original implementation worked well for all Platforms (XP, Vista, 7, and Windows 8) but there was one flaw -- the dreaded white stripe on multiple monitors while maximized -- [brackets/5323](https://github.com/adobe/brackets/issues/5323).

To remedy this, a two new classes were added to the hierarchy to completely control the drawing -- we were still subject to whatever drawing surface Windows gave us to draw on top of to make the UI dark.

|Name | Purpose/Use |
| ---- | ----------- |
| cef_dark_window | a window object which draws a custom dark window frame |
| cef_buffered_dc | an object which manages an off-screen device context for buffered drawing |

# Current State

The current implementation will check to see if the Windows DesktopWindowManager (DWM) is available. An `cef_dark_window` object is used to create an Aero glass surface for drawing if the DWM is installed.  Otherwise, the `cef_dark_window` implementation is used in which will draw a border on a secondary monitor when the window is maximized.

