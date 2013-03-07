## Mac
####Prerequisites

* [Git command line tools](http://git-scm.com/downloads)
* Python (should be installed by default--enter `python --version` in a Terminal window to verify)
* Xcode 4 required to build the project
  * In newer versions of Xcode, you might also need to install the "Command Line Tools" in Xcode from Preferences > Downloads.

####Setup
Open a Terminal window at the `brackets-shell` directory and run `scripts/setup.sh`. This will download the CEF binary (if needed), create symlinks for the CEF directories, and create the XCode project.

You will need to _re-run setup.sh_ later if new source files are added or if brackets-shell updates to a newer CEF build.

####Building in XCode
Open appshell.xcodeproj in XCode and build the "Brackets" target.

####Building from the command line
Open a Terminal window at the `brackets-shell` directory and run `scripts/build.sh`.

####Running
The build output is located at `xcodebuild/Release/Brackets.app`.

When you launch this app, you will be prompted to select an `index.html` file. Navigate to your local copy of the brackets repo and select `src/index.html`. You will be prompted to select the `index.html` file every time you launch. 

You can also run the `tools/setup_for_hacking` script in the brackets repo to create a symlink in your compiled Brackets.app. Pass the full path to the app you are building. Debug builds are in the `xcodebuild/Debug` directory, and Release builds are in the `xcodebuild/Release` directory.

## Windows

####Prerequisites

* Visual Studio 2010 or later required to build the project. The free Visual Studio Express works fine.
* Windows Vista or later (although brackets-shell _runs_ on Windows XP, you cannot _build_ it on XP)
* [GitBash](http://code.google.com/p/msysgit/downloads/list) (I used Git-1.8.0-preview20121022.exe)
* [Python 2.7](http://www.python.org/getit/releases/2.7.3/) (Use the Windows X86 MSI or Windows X86-64 MSI)

Add Python to your path. The default python 2.7 install directory is `C:\Python27`.

####Verify Prerequisites
* Start GitBash
* Enter `python --version`. You should see "Python 2.7.3".
* Enter `echo $VS100COMNTOOLS`. You should see something like ""C:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\Tools\"

####Setup
Open a GitBash shell and navigate to the `brackets-shell` directory. Run `scripts/setup.sh`. This will download the CEF binary (if needed), create symlinks for the CEF directories, and create the Visual Studio solution file.

You will need to _re-run setup.sh_ later if new source files are added or if brackets-shell updates to a newer CEF build.

####Building in Visual Studio.
Open appshell.sln in Visual Studio. NOTE: If you are using Visual Studio Express, you may get warnings that say some of the projects couldn't be loaded. These can be ignored.
Build the "Brackets" target.

####Building from the command line
Open a GitBash window at the `brackets-shell` directory and run `scripts/build.sh`.

####Running
The build output is located at `Release\Brackets.exe`.

When you launch this executable, you will be prompted to select an `index.html` file. Navigate to your local copy of the brackets repo and select `src\index.html`. You will be prompted to select the `index.html` file every time you launch. 

You can also run the `tools\setup_for_hacking` script in the brackets repo to create a symlink in the build directory. Pass the full path to the directory containing the exe you are building. Debug builds are in the `Debug` directory, and Release builds are in the `Release` directory.

## Linux

Not available yet. Please let us know if you'd like to help with the Linux version. -- You may want to take a look at https://github.com/adobe/brackets/wiki/Linux-Version to get started.