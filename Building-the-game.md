Building the game is simple, but since the game relies on AssetBundles to load scenes and other content, you will need to build the content first.

### 1: Building content
Building content means packaging all of the game's assets in AssetBundles, as defined in the Addressable Group window (to see it, go to **Window** > **Asset Management** > **Addressables** > **Groups**). When ready to build content, go to the Groups window and choose the option called **Build** > **New Build** > **Default Build Script**:

![Addressables Build Content dropdown](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/ContentBuild.png?raw=true)

This will produce the AssetBundles that will be stored in the Library folder at `Library/com.unity.addressables/aa/[Platform]`, ready to be put into the build.

### 2: Building the game
Once content has been built, you can kick off a regular build from the menu **File** > **Build Settings** and then clicking the **Build** button. Notice that in the Build Settings window you should see only one scene in the list, called Initialization. The rest of the scenes are packaged in AssetBundles.

If the process completes without errors in the Console, you will find the build in the folder you chose to build into. To find the built AssetBundles, navigate to `/Chop Chop_Data/StreamingAssets/aa/[Platform]` (on Windows) or, (on Mac) right-click the .app file and choose **Show package contents**, then navigate to `/Contents/Data/StreamingAssets/aa/`.

Notice that if you only update scripts, you can create a new build without necessarily building the content first.

**Note:** In case you encounter an error mentioning the Windows 10 SDK (like the one below), make sure to install the Universal Windows Platform Build Support module.

``` 
Exception: C++ code builder is unable to build C++ code. In order to build C++ code for Windows Desktop, you must have one of these installed:
Visual Studio 2015 with C++ compilers and Windows 10 SDK (it cannot build C++ code because it is not installed or missing C++ workload component)
Visual Studio 2015 installation is found by looking at "SOFTWARE\Microsoft\VisualStudio\14.0_Config\InstallDir" in the registry
Windows 10 SDK is found by looking at "SOFTWARE\Wow6432Node\Microsoft\Microsoft SDKs\Windows\v10.0\InstallationFolder" in the registry
```

You can install it from the Unity Hub by going to the Installs tab and adding the module.