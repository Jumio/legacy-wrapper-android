![Header Graphic](images/jumio_feature_graphic.jpg)

# Transition Guide for Jumio Legacy Wrapper SDK
This section covers all technical changes that should be considered when updating from previous versions, including, but not exclusively: API breaking changes or new functionality in the public API, major dependency changes, attribute changes, deprecation notices.

⚠️&nbsp;&nbsp;When updating your SDK version, __all__ changes/updates made in in the meantime have to be taken into account and applied if necessary.   

## 1.0.0
#### Dependency Updates
* The Jumio Legacy Wrapper SDK comes with all needed dependencies defined as transitive dependencies - so all the old dependencies that were introduced by Jumio 3.9.x can be removed
  * `"com.jumio.android:core:3.9.5@aar"`
  * `"com.jumio.android:bam:3.9.5@aar"`
  * `"com.jumio.android:nv:3.9.5@aar"`
  * `"com.jumio.android:nv-mrz:3.9.5@aar"`
  * `"com.jumio.android:nv-nfc:3.9.5@aar"`
  * `"com.jumio.android:nv-ocr:3.9.5@aar"`
  * `"com.jumio.android:nv-barcode:3.9.5@aar"`
  * `"com.jumio.android:nv-barcode-vision:3.9.5@aar"`
  * `"com.jumio.android:iproov:3.9.5@aar"`
  * `"com.jumio.android:dv:3.9.5@aar"`
  * `"androidx.appcompat:appcompat:1.2.0"`
  * `"androidx.room:room-runtime:2.2.6"`
  * `"com.google.android.material:material:1.2.1"`
  * `"androidx.cardview:cardview:1.0.0"`
  * `"androidx.constraintlayout:constraintlayout:2.0.4"`
  * `"org.jetbrains.kotlinx:kotlinx-serialization-core:1.0.0"`
  * `"org.jetbrains.kotlinx:kotlinx-serialization-json:1.0.0"`
  * `"com.google.android.gms:play-services-vision:20.1.3"`
  * `"org.jmrtd:jmrtd:0.7.24"`
  * `"org.ejbca.cvc:cert-cvc:1.4.6"`
  * `"org.bouncycastle:bcprov-jdk15on:1.67"`
  * `"net.sf.scuba:scuba-sc-android:0.0.18"`
  * `"com.iproov.sdk:iproov:6.4.3"`
  * `"androidx.core:core-ktx:1.3.2"`
  * `"org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.4.30"`
* Add the new dependency `"com.jumio.android:legacy-wrapper:1.0.0"`
* Make sure to update your code with the latest versions of Kotlin (1.9.10), the Android Gradle Plugin (8.1.3) and Gradle (8.0)

#### Custom UI Changes
* Custom UI is not supported by the Jumio Legacy Wrapper SDK. The public API is still part of the wrapper library for compatibility reasons, but it is not functional.

#### Public API Changes
* The following API functions have been deprecated
	* `public NetverifyCustomSDKController start(NetverifyCustomSDKInterface customSDKInterface)` in `NetverifySDK`
	* `public synchronized static NetverifySDK create(Activity rootActivity, String offlineToken, @Nullable String preferredCountry)` in `NetverifySDK`
	* `public static String getDebugID()` in `NetverifySDK`
	* `public void setCameraPosition(JumioCameraPosition cameraPosition)` in `NetverifySDK`
	* `public void setDataExtractionOnMobileOnly(boolean dataExtractionOnMobileOnly)` in `NetverifySDK`
	* `public void sendDebugInfoToJumio(boolean send)` in `NetverifySDK`
	* everything in `com.jumio.nv.custom`

## Copyright

&copy; Jumio Corp. 395 Page Mill Road, Palo Alto, CA 94306
