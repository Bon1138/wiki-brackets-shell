# ICU Upgrade steps

* Download the latest source code(not binary) from the ICU repo from http://site.icu-project.org/download (Choose the latest ICU4C Release)
* Do the above step for Unix (MAC and Linux) and Windows
* #### Windows:
  * Extract the downloaded source code and go to `icu\source\allinone` directory
  * There would be a solution file named `allinone.sln`. Open this file in the latest Visual Studio (>=VS2015)
  * Change the build configuration to `Release` and Platform to `Win32` in Visual Studio as we currently support 32 bit brackets only.
  * Once it is built, folders named `icu\bin`, `icu\lib` and `icu\include` would be created. These will contain all the `*.dll` files, lib files and header files respectively, which we will be using in Brackets for Dynamic Linking.
  * Currently we only use `icuuc58.dll`,  `icuin58.dll` and `icudt58.dll` as dlls, `icuin.lib` and `icuuc.lib` as lib files, and we use the complete `include` folder.  
  * Please note that these files might change as we update since ICU may change the functionality of these dlls, we might need to add/remove some dlls, but all the dlls would be present in `icu\bin`, `icu\lib` and `icu\include` directories only.
  * Now we once we have our updated dlls we have to put those updated dlls in S3 so that while building brackets-shell we fetch the latest executables.
  * Upload it to `files.brackets.io/icu` in S3. Please follow the same format which is being currently used.
  * Also if there are some changes in the dll (some are removed/renamed/added), then we have to update the linking step in the brackets-shell as well. Update the dll names in the `Gruntfile.js` and `appshell.gyp`.
  
* #### MAC and Linux
  * Extract the downloaded source code and go to `icu/source` directory
  * From Terminal run `./runConfigureICU MacOSX --enable-static --disable-shared` in case of MAC and `./runConfigureICU Linux --enable-static --disable-shared` in case of Linux to set configuration.
  * Run `make` to build the binaries from the source code.
  * Once it is built, folders named `icu/source/lib` would be created. These will contain all the `*.a` files and header files which we will be using in Brackets for Static Linking.
  * We need to get the `include` folder from the Windows build. We will upload both the lib and include folder to S3.
  * Currently we only use `libicuuc.a`,  `libicui18n.a` and `libicudata.a` as lib files and we use the complete `include` folder. 
  * Please note that these files might change as we update since ICU may change the functionality of these libs, we might need to add/remove some libs, but all the lib files would be present in that `icu/source/lib` directory only.
  * Now we once we have our updated lib files we have to put those updated libs in S3 so that while building brackets-shell we fetch the latest executables.
  * Upload it to `files.brackets.io/icu` in S3. Please follow the same format which is being currently used.
  * Also if there are some changes in the lib (some are removed/renamed/added), then we have to update the linking step in the brackets-shell as well. Update the lib names in the `Gruntfile.js` and `appshell.gyp`.
