
## Mac
####Prerequisites

* [Git command line tools](http://git-scm.com/downloads)
* Python (should be installed by default--enter `python --version` in a Terminal window to verify)
* Xcode 4 required to build the project
  * In newer versions of Xcode, you might also need to install the "Command Line Tools" in Xcode from Preferences > Downloads.

####Setup
Open a Terminal window at the `brackets-shell` directory and run `scripts/setup.sh`. This will download the CEF binary (if needed), create symlinks for the CEF directories, and create the XCode project.

####Building in XCode
Open appshell.xcodeproj in XCode and build the "Brackets" target.

####Building from the command line
Open a Terminal window at the `brackets-shell` directory.
Set the `BRACKETS_SRC` environment variable to point to the location of the brackets repo.
Run `scripts/build.sh`. If you don't have DropDMG installed you will get an error when trying to build the installer. This is okay. You can still run the build at `xcodebuild/Release/Brackets.app`.

####Running
When you launch Brackets, you will be prompted to select an `index.html` file. Navigate to your local copy of the brackets repo and select `src/index.html`. You will be prompted to select the `index.html` file every time you launch. 

You can also run the `tools/setup_for_hacking` script in the brackets repo to create a symlink in your Brackets.app. Pass the full path to the app you are building. Debug builds are in the `xcodebuild/Debug` directory, and Release builds are in the `xcodebuild/Release` directory.

## Windows

####Prerequisites

* Visual Studio 2010 or later required to build the project. The free Visual Studio Express works fine.
* [GitBash](http://code.google.com/p/msysgit/downloads/list) (I used Git-1.8.0-preview20121022.exe)
* [Python 2.7](http://www.python.org/getit/releases/2.7.3/) (Use the Windows X86 MSI or Windows X86-64 MSI)

Add Python to your path. The default python 2.7 install directory is `C:\Python27`.

####Verify Prerequisites
* Start GitBash
* Enter `python --version`. You should see "Python 2.7.3".
* Enter `echo $VS100COMNTOOLS`. You should see something like ""C:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\Tools\"

####Setup
Open a GitBash shell and navigate to the `brackets-shell` directory. Run `scripts/setup.sh`. This will download the CEF binary (if needed), create symlinks for the CEF directories, and create the Visual Studio solution file.

####Building in Visual Studio.
Open appshell.sln in Visual Studio. NOTE: If you are using Visual Studio Express, you may get warnings that say some of the projects couldn't be loaded. These can be ignored.
Build the "Brackets" target.

####Building from the command line
Open a GitBash window at the `brackets-shell` directory.
Set the `BRACKETS_SRC` environment variable to point to the location of the brackets repo.
Run `scripts/build.sh`. This will make a release build of Brackets at `Release/Brackets.exe`.

####Running
When you launch Brackets, you will be prompted to select an `index.html` file. Navigate to your local copy of the brackets repo and select `src\index.html`. You will be prompted to select the `index.html` file every time you launch. 

You can also run the `tools\setup_for_hacking` script in the brackets repo to create a symlink build directory. Pass the full path to the directory of the app you are building. Debug builds are in the `Debug` directory, and Release builds are in the `Release` directory.

## Linux

Not available yet. Please let us know if you'd like to help with the Linux version. -- You may want to take a look at https://groups.google.com/forum/?fromgroups=#!topic/brackets-dev/29vOJ6tvl8A%5B101-125-false%5D to get started.