![Header Graphic](images/jumio_feature_graphic.jpg)

# Integration Guide for Android SDK

Jumio’s products allow businesses to establish the genuine identity of their users by verifying government-issued IDs in real-time. ID Verification, Identity Verification and other services are used by financial service organizations and other leading brands to create trust for safe onboarding, money transfers and user authentication.

## Table of Contents

- [Release Notes](#release-notes)
- [Code Documentation](#code-documentation)
- [Setup](#setup)
  - [Dependencies](#dependencies)
  	- [Autocapture](#autocapture)
  	- [Certified Face Liveness](#certified-face-liveness)
  - [SDK Version Check](#sdk-version-check)
  - [Root Detection](#root-detection)
  - [Device Supported Check](#device-supported-check)
  - [Privacy Notice](#privacy-notice)
  - [Digital Identity (DID)](#digital-identity-did)
- [Initialization](#initialization)
  - [when using apiToken and apiSecret](#when-using-apiToken-and-apiSecret)
  - [when using OAuth2](#when-using-oauth2)
- [Configuration](#configuration)
  - [when using apiToken and apiSecret](#when-using-apitoken-and-apisecret-1)
    - [Worfklow Selection](#worfklow-selection)
    - [Transaction Identifiers](#transaction-identifiers)
    - [Preselection](#preselection)
    - [Miscellaneous](#miscellaneous)
  - [when using OAuth2](#when-using-oauth2-1)
- [SDK Workflow](#sdk-workflow)
  - [Initialization](#initialization)
  - [Working With Fastfill](#working-with-fastfill)
  - [Retrieving Information](#retrieving-information)
- [Default UI](#default-ui)
- [Custom UI](#custom-ui)
  - [Controller Handling](#controller-handling)
  - [Credential Handling](#credential-handling)
  - [ScanPart Handling](#scanpart-handling)
  - [Result and Error Handling](#result-and-error-handling)
  - [Instant Feedback](#instant-feedback)
- [Customization](#customization)
  - [Customization Tool](#customization-tool)
  - [Default UI customization](#default-ui-customization)
  - [Custom UI customization](#custom-ui-customization)

## Release Notes

Please refer to our [Change Log](changelog.md) for more information. Current SDK version: **1.0.1**

For technical changes that should be considered when updating the SDK, please read our [Transition Guide](transition_guide.md).

## Code Documentation

Full API documentation for the Jumio Legacy Wrapper can be found [here](TODO).

## Setup

The [basic setup](../README.md#basics) is required before continuing with the following setup for the Jumio SDK. If you are updating your SDK to a newer version, please also refer to:

:arrow_right:&nbsp;&nbsp;[Changelog](changelog.md)  
:arrow_right:&nbsp;&nbsp;[Transition Guide](transition_guide.md)

### Dependencies
The [Sample app](../sample/JumioMobileSample/) apk size is currently around **17 MB**.

```groovy
dependencies {
	implementation "com.jumio.android:jumiolegacywrapper:1.0.1"
}
```

#### Autocapture
The SDK will automatically detect which type of ID document is presented by the user and guide them through the capturing process with live feedback.
The models can be bundled with the app directly to save time on the download during the SDK runtime. Therefore download the following files from [here](https://cdn.mobile.jumio.ai/model/classifier_on_device_ep_99_float16_quant.enc) and [here](https://cdn.mobile.jumio.ai/model/normalized_ensemble_passports_v2_float16_quant.enc) and add it to the app assets folder.

#### Certified Face Liveness

Jumio uses Certified Liveness technology to determine liveness.

If necessary, the iProov SDK version can be overwritten with a more recent one:

```groovy
implementation("com.iproov.sdk:iproov:8.3.1") {
	exclude group: 'org.json', module: 'json'
}
```

### SDK Version Check

Use `NetverifySDK.getSDKVersion();` to check which SDK version is being used.

### Root Detection

For security reasons, applications implementing the SDK should not run on rooted devices. Use either the below method or a self-devised check to prevent usage of SDK scanning functionality on rooted devices.
```
NetverifySDK.isRooted(Context context);
```

⚠️&nbsp;&nbsp;__Note:__ Please be aware that the JumioSDK root check uses various mechanisms for detection, but doesn't guarantee to detect 100% of all rooted devices.

### Device Supported Check

Use the method below to check if the current device platform is supported by the SDK.

```
NetverifySDK.isSupportedPlatform(Context context)
```

### Privacy Notice
If you submit your app to the Google Play Store a [Prominent Disclosure](https://support.google.com/googleplay/android-developer/answer/11150561) explaining the collected [User Data](https://support.google.com/googleplay/android-developer/answer/10144311) might be required. The collected user data also needs to be declared in your [Data Safety Form](https://play.google.com/console/developers/app/app-content/data-privacy-security) and the [Privacy Policy](https://play.google.com/console/developers/app/app-content/privacy-policy) related to your application.

Other stores might require something similar - please check before submitting your app to the store.

Please see the [Jumio Privacy Policy for Online Services](https://www.jumio.com/legal-information/privacy-notices/jumio-corp-privacy-policy-for-online-services/) for further information.

### Digital Identity (DID)
In case Digital Identity verification has been enabled for your account you can make use of DID verification within the SDK.

Over the course of DID verification the SDK will launch an according third party application representing your Digital Identity. Communication between both applications (your integrating application and the Digital Identity application) is done via a so-called "deep link". For more information on deep link handling on Android please check out their [official guide](https://developer.android.com/training/app-links).

#### Deep Link Setup
To enable your app specific deep link, our support team has to setup an according scheme of your choice for you. This scheme will be used by the SDK to identify your application while returning from the DID provider's application. For the scheme basically any string can be used, however it is recommended that it is unique to your application in some way. A suggestion would be your company name.

Following snippet shows how the deep link needs to be setup in your application's `AndroidManifest.xml` file:

```xml
<activity
	android:name="com.jumio.app.HostingActivity"
	android:exported="true"
	android:launchMode="singleTask"
	android:theme="@style/Theme.Jumio">
	<intent-filter>
		<action android:name="android.intent.action.VIEW" />

		<category android:name="android.intent.category.DEFAULT" />
		<category android:name="android.intent.category.BROWSABLE" />

		<data android:scheme="<your-app-scheme>" />
	</intent-filter>
</activity>
```

Please note that the properties `android:exported="true"` and `android:launchMode="singleTask"` need to be specified as well. The first parameter basically tells the Android system that your `Activity` can be found by the system and other applications. By specifying `launchMode="singleTask"` any already running task for this `Activity` will be resumed (instead of creating a new instance). Both are requirements so that the SDK can handle the according deep link correctly.

In case you are using Jumio's Default UI in your app (see section [Default UI](#default-ui)) you also need to specify `tools:replace="android:exported"` to `JumioActivity`'s `<activity>` tag like so:

```xml
<activity
	android:name="com.jumio.defaultui.JumioActivity"
	android:exported="true"
	android:launchMode="singleTask"
	tools:replace="android:exported">
	<intent-filter>
		...
	</intent-filter>
</activity>
```

As deep link handling happens on `Activity` level, the according data needs to be forwarded to the SDK via `Activity.onNewIntent()`. The following code snippet shows how this can be achieved. **If you're using Jumio's Default UI you can ignore this step**.

```kotlin
override fun onNewIntent(intent: Intent) {
	super.onNewIntent(intent)

	intent.data?.let { deepLink ->
		val activeScanPart = scanPart ?: return

		JumioDeepLinkHandler.consumeForScanPart(deepLink, activeScanPart)
	}
}
```

## Initialization
⚠️&nbsp;&nbsp;**Note:** We strongly recommend storing all credentials outside of your app! We suggest loading them during runtime from your server-side implementation.

### when using apiToken and apiSecret

You can use your token and secret to create an SDK instance

```java
sdk = NetverifySDK.create(Activity rootActivity, String apiToken, String apiSecret, JumioDataCenter dataCenter);
```

⚠️&nbsp;&nbsp;**Note:** If the wrapper is initialized via `apiToken`/`apiSecret` then packageName and versionName of the hosting app will be sent to Jumio.

### when using OAuth2

Your OAuth2 credentials are constructed using your API token as the Client ID and your API secret as the Client secret.
You can view and manage your Client ID and secret in the Customer Portal under:

- **Settings > API credentials > OAuth2 Clients**

Client ID and Client secret are used to generate an OAuth2 access token. Send a workflow request using the acquired OAuth2 access token to receive the SDK token necessary to initialize the Jumio SDK.

OAuth2 has to be activated for your account. Contact your Jumio Account Manager for activation. For more details, please refer to [Authentication and Encryption](../README.md#authentication-and-encryption).

```java
sdk = NetverifySDK.create(Activity rootActivity, String bearerToken, JumioDataCenter dataCenter);
```

## Configuration

### when using apiToken and apiSecret

#### ID Verification
Use ID verification callback to receive a verification status and verified data positions (see [Callback section](https://jumio.github.io/kyx/integration-guide.html#callback)). Please note: The Jumio Legacy SDK Wrapper will invoke a new calback and retrieval format compared to the Netverify SDK 3.9.x - to use the old format please contact Jumio Support. Make sure that your customer account is enabled to use this feature. A callback URL can be specified for individual transactions (for URL constraints see chapter [Callback URL](https://jumio.github.io/kyx/integration-guide.html#jumio-callback-ip-addresses)). This setting overrides any callback URL you have set in the Jumio Customer Portal. Your callback URL must not contain sensitive data like PII (Personally Identifiable Information) or account login.

__Note:__ Not possible for accounts configured as Fastfill only.
```
netverifySDK.setCallbackUrl("YOURCALLBACKURL");
```

__Note:__ The callback URL must not contain sensitive data like PII (Personally Identifiable Information) or account login.


Set the following setting to switch to Fastfill mode (which performs data extraction only):
```
netverifySDK.setEnableVerification(false);
```
Identity Verification is automatically enabled if it is activated for your account. Set the following setting to disable Identity Verification on a transaction level:
```
netverifySDK.setEnableIdentityVerification(false);
```

__Note:__ Identity Verification requires portrait orientation in your app.

#### Preselection
You can specify issuing country (using [ISO 3166-1 alpha-3](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country codes), ID type(s) and/or document variant to skip their selection during the scanning process. In the example down below Austria ("AUT") has been preselected, as well as a specific document variant (PLASTIC), and added PASSPORT and DRIVER_LICENSE as preselected document types. When all three parameters are preselected, the document selection screen in the SDK can be skipped entirely.

```
netverifySDK.setPreselectedCountry("AUT");
netverifySDK.setPreselectedDocumentVariant(NVDocumentVariant.PLASTIC);

ArrayList<NVDocumentType> documentTypes = new ArrayList<>();
documentTypes.add(NVDocumentType.PASSPORT);
documentTypes.add(NVDocumentType.DRIVER_LICENSE);
netverifySDK.setPreselectedDocumentTypes(documentTypes);
```
__Note:__ Fastfill does not support paper IDs, except German ID cards.

#### Transaction Identifiers
The customer internal reference allows you to identify the scan (max. 100 characters).

```
netverifySDK.setCustomerInternalReference("YOURSCANREFERENCE");
```
Use the following property to identify the scan in your reports (max. 100 characters).
```
netverifySDK.setReportingCriteria("YOURREPORTINGCRITERIA");
```
You can also set a unique identifier for each user (max. 100 characters).

```
netverifySDK.setUserReference("USERREFERENCE");

```
__Note:__ Transaction identifiers must not contain sensitive data like PII (Personally Identifiable Information) or account login.

#### Jumio Watchlist Screening
[Jumio Screening](https://www.jumio.com/screening/) is supported on the Jumio Android SDK. The following SDK method is used to set watchlist screening on transaction level. Enable to override the default search, or disable watchlist screening for this transaction.
```
netverifySDK.setWatchlistScreening(NVWatchlistScreening.ENABLED);
```
This method can be used to define the search profile for watchlist screening:
```
netverifySDK.setWatchlistSearchProfile("YOURPROFILENAME");
```

#### Miscellaneous
In case Fastfill is used (enableVerification=false), data extraction can be limited to be executed on device only by enabling `setDataExtractionOnMobileOnly`
```
netverifySDK.setDataExtractionOnMobileOnly(true);
```

### when using OAuth2
Every Jumio SDK instance is initialized using a specific [`sdk.token`][token]. This token contains information about the workflow, credentials, transaction identifiers and other parameters. Configuration of this token allows you to provide your own internal tracking information for the user and their transaction, specify what user information is captured and by which method, as well as preset options to enhance the user journey. Values configured within the [`sdk.token`][token] during your API request will override any corresponding settings configured in the Customer Portal.

#### Worfklow Selection

Use ID verification callback to receive a verification status and verified data positions (see [Callback section](https://jumio.github.io/kyx/integration-guide.html#callback)). Make sure that your customer account is enabled to use this feature. A callback URL can be specified for individual transactions (for URL constraints see chapter [Callback URL](https://jumio.github.io/kyx/integration-guide.html#jumio-callback-ip-addresses)). This setting overrides any callback URL you have set in the Jumio Customer Portal. Your callback URL must not contain sensitive data like PII (Personally Identifiable Information) or account login. Set your callback URL using the `callbackUrl` parameter.

Use the correct [workflow definition key](https://jumio.github.io/kyx/integration-guide.html#workflow-definition-keys) in order to request a specific workflow. Set your key using the `workflowDefinition.key` parameter.

```json
{
  "customerInternalReference": "CUSTOMER_REFERENCE",
  "workflowDefinition": {
	"key": "X"
  },
  "callbackUrl": "YOUR_CALLBACK_URL"
}
```

For more details, please refer to our [Workflow Description Guide](https://support.jumio.com/hc/en-us/articles/4408958923803-KYX-Workflows-User-Guide).

ℹ️&nbsp;&nbsp;**Note:** Identity Verification requires portrait orientation in your app.

#### Transaction Identifiers

There are several options in order to uniquely identify specific transactions. `customerInternalReference` allows you to specify your own unique identifier for a certain scan (max. 100 characters). Use `reportingCriteria`, to identify the scan in your reports (max. 100 characters). You can also set a unique identifier for each user using `userReference` (max. 100 characters).

For more details, please refer to our [API Guide](https://jumio.github.io/kyx/integration-guide.html#request-body).

```json
{
  "customerInternalReference": "CUSTOMER_REFERENCE",
  "workflowDefinition": {
	"key": "X"
  },
  "reportingCriteria": "YOUR_REPORTING_CRITERIA",
  "userReference": "YOUR_USER_REFERENCE"
}
```

⚠️&nbsp;&nbsp;**Note:** Transaction identifiers must not contain sensitive data like PII (Personally Identifiable Information) or account login.

#### Preselection

You can specify issuing country using [ISO 3166-1 alpha-3](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country codes, as well as ID types to skip selection during the scanning process. In the example down below, Austria ("AUT") and the USA ("USA") have been preselected. PASSPORT and DRIVER_LICENSE have been chosen as preselected document types. If all parameters are preselected and valid and there is only one given combination (one country and one document type), the document selection screen in the SDK can be skipped entirely.

For more details, please refer to our [API Guide](https://jumio.github.io/kyx/integration-guide.html#request-body).

⚠️&nbsp;&nbsp;**Note:** "Digital Identity" document type can not be preselected!

```json
{
  "customerInternalReference": "CUSTOMER_REFERENCE",
  "workflowDefinition": {
	"key": X,
	"credentials": [
	  {
		"category": "ID",
		"type": {
		  "values": [
			"DRIVING_LICENSE",
			"PASSPORT"
		  ]
		},
		"country": {
		  "values": [
			"AUT",
			"USA"
		  ]
		}
	  }
	]
  }
}
```

#### Miscellaneous

Use [`cameraFacing`][camerafacing] attribute of [`JumioScanView`][jumioscanview] to configure the default camera and set it to `FRONT` or `BACK`.

```kotlin
scanView.cameraFacing = JumioCameraFacing.FRONT
```

## SDK Workflow

### Initialization
Use the `initiate()` method to preload the SDK and avoid the loading spinner after the SDK start, be notified of successful initialization, successful scans, and error situations.
```
netverifySDK.initiate(new NetverifyInitiateCallback() {
	@Override
	public void onNetverifyInitiateSuccess() {
		// YOURCODE
	}
	@Override
	public void onNetverifyInitiateError(String errorCode, String errorMessage, boolean retryPossible) {
		// YOURCODE
	}
});
```
To show the SDK, call the respective method below within your activity or fragment.

Activity: `netverifySDK.start();`
Fragment: `startActivityForResult(netverifySDK.getIntent(), NetverifySDK.REQUEST_CODE);`

__Note:__ The default request code is 200. To use another code, override the public static variable `NetverifySDK.REQUEST_CODE` before displaying the SDK.

### Working With Fastfill
Implement the standard `onActivityResult()` method in your activity or fragment for successful scans (`Activity.RESULT_OK`) and user cancellation notifications (`Activity.RESULT_CANCELED`). Call `netverifySDK.destroy()` once you received the result and don't need the instance anymore. If you want to scan multiple documents, you don't need to call delete on the netverifySDK instance. Simply check if the internal resources are deallocated by calling `netverifySDK.checkDeallocation(<NetverifyDeallocationCallback>)`. Once this callback is executed, it is safe to start another workflow. This check is optional and should only be called once the SDK has returned a result and another document scan needs to be performed.

```
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	if (requestCode == NetverifySDK.REQUEST_CODE) {
		if (resultCode == Activity.RESULT_OK) {
			// OBTAIN PARAMETERS HERE
			// YOURCODE
		} else if (resultCode == Activity.RESULT_CANCELED) {
			// String scanReference = data.getStringExtra(NetverifySDK.EXTRA_SCAN_REFERENCE);
			// String errorMessage = data.getStringExtra(NetverifySDK.EXTRA_ERROR_MESSAGE);
			// String errorCode = data.getStringExtra(NetverifySDK.EXTRA_ERROR_CODE);
			// YOURCODE
		}
		// CLEANUP THE SDK AFTER RECEIVING THE RESULT
		// if (netverifySDK != null) {
		// 	netverifySDK.destroy();
		//      netverifySDK.checkDeallocation(deallocationCallback)
		// 	netverifySDK = null;
		// }
	}
}
```

### Retrieving Information
The following tables give information on the specification of all document data parameters and errors.

#### Class ___NetverifyDocumentData___
|Parameter | Type  	| Max. length    | Description     |
|:-------------------|:----------- 	|:-------------|:-----------------|
|selectedCountry|	String|	3|	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code as provided or selected|
|selectedDocumentType|	NVDocumentType |	|	PASSPORT, DRIVER_LICENSE, IDENTITY_CARD or VISA as provided or selected|
|idNumber|	String|	100	|Identification number of the document|
|personalNumber|	String|	14|	Personal number of the document|
|issuingDate|	Date|	|	Date of issue|
|expiryDate| Date|	|	Date of expiry|
|issuingCountry|	String|	3|	Country of issue as [ISO 3166-1 alpha-3](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code|
|lastName|	String|	100	|Last name of the customer|
|firstName|	String|	100	|First name of the customer|
|dob|	Date|		|Date of birth|
|gender|	NVGender|		| Gender M, F or X|
|originatingCountry|	String|	3|	Country of origin as [ISO 3166-1 alpha-3](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code|
|addressLine|	String|	64	|Street name|
|city|	String|	64	|City|
|subdivision|	String|	3|	Last three characters of [ISO 3166-2:US](https://en.wikipedia.org/wiki/ISO_3166-2:US) or [ISO 3166-2:CA](https://en.wikipedia.org/wiki/ISO_3166-2:CA) subdivision code	|
|postCode|	String|	15|	Postal code	|
|mrzData|	NetverifyMrzData|		|MRZ data, see table below|
|optionalData1|	String|	50|	Optional field of MRZ line 1|
|optionalData2|	String|	50	|Optional field of MRZ line 2|
|placeOfBirth|	String|	255	|Place of Birth	|
|extractionMethod|	NVExtractionMethod| |Extraction method used during scanning (MRZ, OCR, BARCODE, BARCODE_OCR or NONE) |
|imageData|	NetverifyImageData|	|Wrapper class for accessing image data for all scan sides from an ID verification session in case this is enabled by your Account Manager. See [NetverifyImageData](https://jumio.github.io/mobile-sdk-android/com/jumio/nv/NetverifyImageData.html) for details on how to retrieve the images|


#### Class ___NetverifyMrzData___
|Parameter  |Type 	| Max. length | Description      |
|:----------|:------|:------------|:-----------------|
|format|	NVMRZFormat|		|
|line1|	String|	50|	MRZ line 1	|
|line2|	String| 50|	MRZ line 2	|
|line3|	String|	50|	MRZ line 3	|
|idNumberValid|	boolean | | True if ID number check digit is valid, otherwise false	|
|dobValid| boolean | | True if date of birth check digit is valid, otherwise false	|
|expiryDateValid|	boolean| | True if date of expiry check digit is valid or not available, otherwise false |
|personalNumberValid	| boolean | |	True if personal number check digit is valid or not available, otherwise false |
|compositeValid| boolean | | True if composite check digit is valid, otherwise false	|

#### Error Codes
List of all **_error codes_** that are available via the `code` property of the `NetverifyError` object. The first letter (A-J) represents the error case. The remaining characters are represented by numbers that contain information helping us understand the problem situation([x][yyyy]).

|Code        	  | Message  | Description      |
| :--------------:|:---------|:-----------------|
|A[x][yyyy]| We have encountered a network communication problem | Retry possible, user decided to cancel |
|B[x][yyyy]| Authentication failed | Secure connection could not be established, retry impossible |
|C[x]0401| Authentication failed | API credentials invalid, retry impossible |
|E[x]0000| No Internet connection available | Retry possible, user decided to cancel |
|F00000| Scanning not available at this time, please contact the app vendor | Resources cannot be loaded, retry impossible |
|G00000| Cancelled by end-user | No error occurred |
|H00000| The camera is currently not available | Camera cannot be initialized, retry impossible |
|I00000| Certificate not valid anymore. Please update your application | End-to-end encryption key not valid anymore, retry impossible |
|J00000| Transaction already finished | User did not complete SDK journey within session lifetime |
|N00000| Scanning not available at this time, please contact the app vendor | Required images are missing to finalize the acquisition |

__Note:__ Please always include the whole code when filing an error related issue to our support team.

### Custom UI

Custom UI is not supported in the Jumio Legacy Wrapper. The public API is still available to not cause breaking changes, however it is not functional.

### Instant Feedback

The use of Instant Feedback provides immediate end user feedback by performing a usability check on any image the user took and prompting them to provide a new image immediately if this image is not usable, for example because it is too blurry. Please refer to the [JumioRejectReason table](#class-jumiorejectreason) for a list of all reject possibilities.

## Customization

The Jumio SDK provides various options to customize its UI. If you are using [Default UI](#default-ui) you can change each screen to fulfil your needs. In case you decide to implement the verification workflow on your own (see [Custom UI](#custom-ui)) you also have the possibility to influence the look and feel of some views provided by the SDK, e.g. [`JumioScanView`][jumioscanview].

### Customization Tool

[Jumio Surface](https://jumio.github.io/surface-tool/) is a web tool that offers the possibility to apply and visualize all available customization options for the Jumio SDK, as well as an export feature that generates all data needed to import the desired changes straight into your codebase.

[![Jumio Surface](images/surface_tool.png)](https://jumio.github.io/surface-tool/)

### Default UI customization

The surface tool utilizes each screen of Jumio's [Default UI](#default-ui) to visualize all items and colors that can be customized. If you are planning to use the [Default UI](#default-ui) implementation, you can specify the `Theme.Jumio` as a parent style in your application and override according attributes within this theme to match your application's look and feel.

After customizing the SDK via the surface tool, you can click the **Android-Xml** button in the **Output** menu on the bottom right to copy the code from the theme `AppThemeCustomJumio` to your Android app's `styles.xml` file.

Apply your custom theme that you defined before by replacing `Theme.Jumio` in the `AndroidManifest.xml:`

```xml
<activity
  android:name="com.jumio.defaultui.JumioAcitivty"
  android:theme="@style/AppThemeCustomJumio">
	...
</activity>
```

#### Dark Mode

`Theme.Jumio` attributes can also be customized for dark mode. If you haven't done so already, create a `values-night` folder in your resources directory and add a new `styles.xml` file. Adapt your custom Jumio theme for dark mode. The SDK will switch automatically to match the system settings of the user device.

# Security

All SDK related traffic is sent over HTTPS using TLS and public key pinning. Additionally, the information itself within the transmission is also encrypted utilizing **Application Layer Encryption** (ALE). ALE is a Jumio custom-designed security protocol that utilizes RSA-OAEP and AES-256 to ensure that the data cannot be read or manipulated even if the traffic was captured.

# Support

## Licenses

The software contains third-party open source software. For more information, see [licenses](licenses).

This software is based in part on the work of the Independent JPEG Group.

## Contact

If you have any questions regarding our implementation guide please contact **Jumio Customer Service** at [support@jumio.com](mailto:support@jumio.com). The [Jumio online helpdesk](https://support.jumio.com) contains a wealth of information regarding our services including demo videos, product descriptions, FAQs, and other resources that can help to get you started with Jumio.

## Copyright

&copy; Jumio Corporation, 395 Page Mill Road, Suite 150, Palo Alto, CA 94306

The source code and software available on this website (“Software”) is provided by Jumio Corp. or its affiliated group companies (“Jumio”) "as is” and any express or implied warranties, including, but not limited to, the implied warranties of merchantability and fitness for a particular purpose are disclaimed. In no event shall Jumio be liable for any direct, indirect, incidental, special, exemplary, or consequential damages (including but not limited to procurement of substitute goods or services, loss of use, data, profits, or business interruption) however caused and on any theory of liability, whether in contract, strict liability, or tort (including negligence or otherwise) arising in any way out of the use of this Software, even if advised of the possibility of such damage.

In any case, your use of this Software is subject to the terms and conditions that apply to your contractual relationship with Jumio. As regards Jumio’s privacy practices, please see our privacy notice available here: [Privacy Policy](https://www.jumio.com/legal-information/privacy-policy/).

[token]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk/-jumio-s-d-k/token.html
[datacenter]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk/-jumio-s-d-k/data-center.html
[sdkversion]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk/-jumio-s-d-k/-companion/version.html
[isrooted]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk/-jumio-s-d-k/-companion/is-rooted.html
[camerafacing]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-camera-facing/index.html
[acquiremode]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-acquire-mode/index.html
[fallbackreason]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-fallback-reason/index.html
[userconsented]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.controller/-jumio-controller/user-consented.html
[isconfigured]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-credential/is-configured.html
[setidconfiguration]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-i-d-credential/set-configuration.html
[supportedcountries]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-i-d-credential/supported-countries.html
[getphysicaldocuments]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-i-d-credential/get-physical-documents-for-country.html
[getdigitaldocuments]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-i-d-credential/get-digital-documents-for-country.html
[iscompletecredential]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-credential/is-complete.html
[iscompletecontroller]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.controller/-jumio-controller/is-complete.html
[startscanpart]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.scanpart/-jumio-scan-part/start.html
[finishscanpart]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.scanpart/-jumio-scan-part/finish.html
[finishcredential]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-credential/finish.html
[finishcontroller]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.controller/-jumio-controller/finish.html
[isretryable]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.error/-jumio-error/is-retryable.html
[confirm]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-confirmation-view/confirm.html
[retakeconfirmation]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-confirmation-view/retake.html
[retakereject]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-reject-view/retake.html
[retrycontroller]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.controller/-jumio-controller/retry.html
[onfinished]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.interfaces/-jumio-controller-interface/on-finished.html
[onscanstep]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.interfaces/-jumio-scan-part-interface/on-scan-step.html
[onupdate]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.interfaces/-jumio-scan-part-interface/on-update.html
[canfinish]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-c-a-n_-f-i-n-i-s-h/index.html
[prepare]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-p-r-e-p-a-r-e/index.html
[started]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-s-t-a-r-t-e-d/index.html
[attachactivity]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-a-t-t-a-c-h_-a-c-t-i-v-i-t-y/index.html
[attachfile]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-a-t-t-a-c-h_-f-i-l-e/index.html
[scanview]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-s-c-a-n_-v-i-e-w/index.html
[digitalidentityview]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-d-i-g-i-t-a-l_-i-d-e-n-t-i-t-y_-v-i-e-w/index.html
[imagetaken]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-i-m-a-g-e_-t-a-k-e-n/index.html
[nextpart]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-n-e-x-t_-p-a-r-t/index.html
[processing]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-p-r-o-c-e-s-s-i-n-g/index.html
[confirmationview]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-c-o-n-f-i-r-m-a-t-i-o-n_-v-i-e-w/index.html
[rejectview]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-r-e-j-e-c-t_-v-i-e-w/index.html
[retry]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-r-e-t-r-y/index.html
[addonscanpart]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-a-d-d-o-n_-s-c-a-n_-p-a-r-t/index.html
[thirdpartyverification]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/-t-h-i-r-d_-p-a-r-t-y_-v-e-r-i-f-i-c-a-t-i-o-n/index.html
[jumioactivityattacher]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-activity-attacher/index.html
[jumiofileattacher]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-file-attacher/index.html
[jumioscanview]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-scan-view/index.html
[jumiocontroller]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.controller/-jumio-controller/index.html
[jumiocontrollerinterface]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.interfaces/-jumio-controller-interface/index.html
[jumioresult]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.result/-jumio-result/index.html
[jumioidresult]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.result/-jumio-i-d-result/index.html
[jumiofaceresult]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.result/-jumio-face-result/index.html
[jumiorejectreason]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.reject/-jumio-reject-reason/index.html
[jumioerror]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.error/-jumio-error/index.html
[jumiocredential]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-credential/index.html
[jumioidcredential]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-i-d-credential/index.html
[jumiodocumentcredential]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-document-credential/index.html
[jumiofacecredential]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-face-credential/index.html
[jumiodatacredential]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-data-credential/index.html
[jumiocredentialcategory]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-credential-category/index.html
[jumiophysicaldocument]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.document/-jumio-physical-document/index.html
[jumiodigitaldocument]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.document/-jumio-digital-document/index.html
[jumiodocumenttype]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.document/-jumio-document-type/index.html
[jumiodocumentvariant]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.document/-jumio-document-variant/index.html
[jumiocredentialpart]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-credential-part/index.html
[jumioscanstep]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-step/index.html
[jumioretryreason]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.retry/-jumio-retry-reason/index.html
[jumioretrygeneric]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.retry/-jumio-retry-reason-generic/index.html
[jumioretrydv]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.retry/-jumio-retry-reason-document-verification/index.html
[jumioretrynfc]: https://jumio.github.io/mobile-sdk-android/jumio-nfc/com.jumio.sdk.retry/-jumio-retry-reason-nfc/index.html
[jumioretryiproov]: https://jumio.github.io/mobile-sdk-android/jumio-iproov/com.jumio.sdk.retry/-jumio-retry-reason-iproov/index.html
[jumioretrydi]: https://jumio.github.io/mobile-sdk-android/jumio-digital-identity/com.jumio.sdk.retry/-jumio-retry-reason-digital-identity/index.html
[jumioconfirmationview]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-confirmation-view/index.html
[jumiorejectview]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-reject-view/index.html
[jumioconfirmationhandler]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.handler/-jumio-confirmation-handler/index.html
[jumiorejecthandler]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.handler/-jumio-reject-handler/index.html
[jumioscanupdate]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-update/index.html
[jumiofileattacher]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-file-attacher/index.html
[jumioscanpart]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.scanpart/-jumio-scan-part/index.html
[jumiomultipart]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-credential-part/-m-u-l-t-i-p-a-r-t/index.html
[jumioscanmode]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-scan-mode/index.html
[jumioconsenttype]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.enums/-jumio-consent-type/index.html
[jumioconsentitem]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.consent/-jumio-consent-item/index.html
[credentialpartslist]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.credentials/-jumio-credential/credential-parts.html
[proguardrules]: https://github.com/Jumio/mobile-sdk-android/blob/master/sample/JumioMobileSample/proguard-rules.pro
[jumiodiview]: https://jumio.github.io/mobile-sdk-android/jumio-core/com.jumio.sdk.views/-jumio-digital-identity-view/index.html
