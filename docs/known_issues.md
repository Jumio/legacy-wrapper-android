![Header Graphic](images/jumio_feature_graphic.jpg)

# Known Issues

## Table of Contents
- [SDK Version 4.0.0 and Above](#sdk-version-400-and-above)
  - [Duplicate Files for 'libc++_shared.so' Library](#duplicate-lib-c)
- [Miscellaneous](#miscellaneous)
  - [Static Interface Methods Are only Supported with Android N](#Static-interface-methods-are-only-supported-with-Android-N)
  - [SDK Crashes Trying to Display Animations (Android Version 5 and Lower)](#sdk-crashes-trying-to-display-animations-(android-version-4-and-lower))
  - [Datadog in Dynamic feature modules](#datadog-in-dynamic-feature-modules)

# SDK Version 4.0.0 and Above

<div id="duplicate-lib-c">
<h2>Duplicate Files for 'libc++_shared.so' Library</h2>
</div>

If build fails with error message:

_2 files found with path 'lib/arm64-v8a/libc++\_shared.so' from inputs ..._

Please add the following `packagingOptions` to the configuration in your `build.gradle` file:

```
android{
  packagingOptions {
      pickFirst 'lib/armeabi-v7a/libc++_shared.so'
      pickFirst 'lib/arm64-v8a/libc++_shared.so'
  }
}

```

# Miscellaneous

## Static Interface Methods Are only Supported with Android N
Error message "Static interface methods are only supported with Android N" will be displayed when Java 8 compatibility is not enabled. In this case, please make sure to enable compatibility in your `build.gradle` file:
```
android {
...
  compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

## SDK Crashes Trying to Display Animations (Android Version 5 and Lower)
Running the SDK on API Level 21/Android Version 5 ("Lollipop") or lower, the application might crash when trying to display Jumio animations. In this case it is necessary to add the line `AppCompatDelegate.setCompatVectorFromResourcesEnabled(true)` to the `onCreate()` method of your application or `Activity`, ideally before `setContentView()` is called.

__Note:__ Refer to [required for vector drawable compat handling](https://stackoverflow.com/a/37864531/1297835) for further information.

## Datadog in Dynamic feature modules
Datadog registers a Content Receiver through its AndroidManifest. Therefore Datadog needs to be linked in the base app, otherwise the app will crash during start.
```
api "com.datadoghq:dd-sdk-android-rum:2.0.0"
```
__Note:__ Version numbers may vary.