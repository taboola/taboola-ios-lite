# Taboola iOS Lite SDK Documentation

The Taboola Lite SDK allows developers to easily integrate Taboola's personalized content recommendations into their iOS applications. This documentation provides all the necessary steps and details to set up and use the SDK.

---

## Prerequisites

Before you begin, ensure your project meets the following requirements:

- **Minimum iOS Version**: 14.0
- **Xcode**: 13.0 or later
- **Swift**: 5.0 or later

---

## Installation

### Swift Package Manager

1. In Xcode, go to File > Add Packages
2. Enter the package URL: `https://github.com/taboola/taboola-ios-lite`
3. Select the version you want to use by setting the branch name
4. Click Add Package

### Manual Installation

1. Download the latest release from GitHub
2. Drag the `TaboolaLite.xcframework` into your Xcode project
3. Make sure "Copy items if needed" is checked
4. Select your target and click "Embed & Sign"

---

## Getting Started

### 1. Initialize the SDK

The `TBLSDK.initialize` method must be called before using any other SDK functionality. Initialize the SDK in your `AppDelegate`.
> **Important:** You must wait for `onTaboolaInitializationComplete` with status `SUCCESS` before calling `setupTaboolaNews`. Calling it earlier will result in `SDK_NOT_INITIALIZED` error.

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    let publisherId = "your-publisher-id"
    let userData = TBLUserData(email: email, globalUserId: globalUserId)
    
    TBLSDK.shared.initialize(publisherId: publisherId, data: userData, onTaboolaListener: OnTBLListener())
    return true
}
```

* **Parameters**:

  * `publisherId`: A valid Taboola PublisherId (e.g., `publisherId`).
  * `userData`: An instance of `TBLUserData` containing user-specific data.
  * `onTaboolaListener`: An implementation of `OnTBLListener` for lifecycle callbacks.

### 2. Add Taboola to a View

Once SDK is initialized (after receiving SUCCESS from `onTaboolaInitializationComplete`), add Taboola content using `TBLSDK.setupTaboolaNews`:

```swift
class NewsViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        TBLSDK.shared.setupTaboolaNews(view: view, onTBLNewsListener: OnTBLNewsListener())
    }
}
```

### 3. Remove Taboola Content

When you're done with the Taboola content, remove it from the view:

```swift
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    TBLSDK.shared.removeTaboolaNewsFromView()
}
```

### 4. Set User Data (Optional)

You can update user data at any time:

```swift
let userData = TBLUserData(
    "hashedEmail1",
    "gender",
    "age",
    "userInterestAndIntent"
)
TBLSDK.shared.setUserData(userData)
```

### 5. Cleanup on App Termination

To properly clean up SDK resources when the app terminates, add the following to your AppDelegate:

```swift
func applicationWillTerminate(_ application: UIApplication) {
    TBLSDK.shared.deinitialize()
}
```

This ensures that all SDK resources are properly released when the app is terminated.

---


### Set User Data Collection Preference

Use this method to enable or disable user data collection based on user consent.
This method helps control whether the SDK collects user data, adhering to user privacy preferences.
The function can be called **before or after** `TBLSDK.initialize`

```swift
TBLSDK.shared.setCollectUserData(allowPersonalization: Bool, allowDeviceContext: Bool)
```
- **Parameters**:
  - `allowPersonalization`: A boolean indicating whether the allowes personaliztion (true) or not (false).
  - `allowDeviceContext`: A boolean indicating whether the allowes collecting device context (true) or not (false).

---

---

## Listener Interfaces

### Taboola Event Listeners

#### OnTBLListener

The `OnTBLListener` interface listens to global SDK events:

```swift
public protocol OnTBLListener: AnyObject {
    func onTaboolaInitializationComplete(statusCode: TBLStatusCode)
    func onTaboolaLoadComplete(statusCode: TBLStatusCode)
    func onTaboolaSharePressed(url: String)
}
```

* **onTaboolaInitializationComplete**: Called when the SDK initialization finishes.
* **onTaboolaLoadComplete**: Called when the creation of the Taboola WebView completes.
* **onTaboolaSharePressed**: Triggered when a user presses the share button.

#### OnTBLNewsListener

The `OnTBLNewsListener` interface allows you to listen for news-fragment events from Taboola:

```swift
public protocol OnTBLNewsListener: AnyObject {
    func onTaboolaNewsSetupComplete(statusCode: TBLStatusCode)
    func onTaboolaNewsRefreshComplete(statusCode: TBLStatusCode)
    func onTaboolaNewsClickComplete(statusCode: TBLStatusCode)
}
```

* **onTaboolaNewsSetupComplete**: Called when the Taboola WebView is successfully added to the fragment.
* **onTaboolaNewsRefreshComplete**: Called when the Taboola WebView finishes refreshing content.
* **onTaboolaNewsClickComplete**: Called when clicking on opening category or item.

The `TBLStatusCode` enum includes the following statuses
Each status also includes a `message` property that provides a user-friendly description, which can be used for logging or displaying error messages in the UI:

- SUCCESS (200): "Success"
- BAD_REQUEST (400): "Bad Request - Please check your input."
- SERVICE_UNAVAILABLE (503): "Service Unavailable - Try again later."
- PUBLISHER_INVALID (-1): "Publisher Invalid - Please contact support."
- WEB_VIEW_NOT_FOUND(-2): "WebView Not Found - Please check if you deinitialized the SDK."
- SDK_DISABLED(-3): "SDK Disabled - SDK functionality has been disabled."
- SDK_NOT_INITIALIZED(-4): "SDK Not Initialized - Please call initialize() first."
- INVALID_VIEW_GROUP(-5): "Invalid View - View must be a ViewGroup and not null."
- TIMEOUT(-6):"Timeout - Try again later."

---


## Lifecycle Management

### Pause/Resume Taboola Content

When your app goes to background or returns to foreground:

```swift
// In AppDelegate
func applicationDidEnterBackground(_ application: UIApplication) {
    TBLSDK.shared.onPauseTaboolaNews()
}

func applicationWillEnterForeground(_ application: UIApplication) {
    TBLSDK.shared.onResumeTaboolaNews()
}
```

### Scroll to Top

To programmatically scroll the Taboola content to the top:

```swift
TBLSDK.shared.onScrollToTopTaboolaNews()
```

---

## Advanced Configuration

### Set Log Level

To control the log verbosity of the SDK, you can set the log level as follows:

```swift
TBLSDK.shared.setLogLevel(TBLLogLevel.debug) // Options: .error, .warn, .info, .debug
```

### Update Reload Intervals (Testing Only)

For testing purposes, you can configure the reload intervals for the WebView content:

```
TBLSDK.shared.updateReloadIntervals(
    1, // Set WebView reload interval to 1 minutes
    1  // Set timer repeat interval to 1 minutes
)
```

## Changelog

### Version 1.2.0
- Change `TBLUserData` init with hashed email and GUID.
- Change `setCollectUserData` to pass `allowPersonalization` and `allowDeviceContext`

### Version 1.1.3
- Add a fix to status bar color.
- Add timout stutus to error handling.
- Fix Auto refresh.

### Version 1.1.2
- Dynamic SDK

### Version 1.1.1
- Status bar color change

### Version 1.1.0
- Remove location code.
- Not collection IDFA if no concent

### Version 1.0.8
- Enable deeplink in taboola webView
- Fix wait for `onTaboolaInitializationComplete`
- Remove collection of location
- Fix security issue

### Version 1.0.7
- Added lifecycle-safe listener interfaces: `OnTBLListener`, `OnTBLNewsListener`
- Expanded `TBLStatusCode` with full error message support
- Added mandatory success-check requirement before calling `setupTaboolaNews`

### Version 1.0.4
- New function `setCollectUserData` to enable/disable collecting user data.
- Add privacy manifest.
- Fix error event handling (like when the publisher id is incorrect)

### Version 1.0.3
- Remove handle crash and error events.

### Version 1.0.2
- Send an error event if pull to refresh failes.
- Handle click events in web pages.
- Remove javascript bridge in non taboola pages.
- Handle crash and error events.

### Version 1.0.1
- Added updateReloadIntervals method for testing WebView reload behavior
- Added setLogLevel method for controlling SDK log verbosity

### Version 1.0.0
- Initial release of the Taboola Lite SDK
- Includes user data configuration and publisher-specific settings
- Provides lifecycle management for proper SDK initialization and cleanup
- Supports event handling for Taboola content interactions
