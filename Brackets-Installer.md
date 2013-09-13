## How to Build

1. Follow the setup steps here:
    * https://github.com/adobe/brackets-shell/blob/master/installer/win/README.MD
    * https://github.com/adobe/brackets-shell/blob/master/installer/mac/README.MD
2. Run `grunt build-installer`

**Note:** If you're building a customized installer to run something other than [Brackets](https://github.com/adobe/brackets) itself, **_please_** be sure to change the Info.plist bundle id (Mac) and installer GUIDs (Win) so that whatever app you're building doesn't become entangled with Brackets installs on the same machine.