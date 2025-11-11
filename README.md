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
4. Select your target and select "Do Not Embed" (the framework is static)

---

## Getting Started

### 1. Initialize the SDK

The `TBLSDK.initialize` method must be called before using any other SDK functionality. Initialize the SDK in your `AppDelegate`.
> **Important:** You must wait for `onTaboolaInitializationComplete` with status `SUCCESS` before calling `setupTaboolaNews`. Calling it earlier will result in `SDK_NOT_INITIALIZED` error.

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    let publisherId = "your-publisher-id"
    let userData = TBLUserData(
        email: email,
        globalUserId: globalUserId,
        allowPersonalization: true,
        allowDeviceContext: true
    )
    let newsParams = TBLNewsParams() // Empty for standard flow

    TBLSDK.shared.initialize(publisherId: publisherId, data: userData, newsParams: newsParams, onTaboolaListener: OnTBLListener())
    return true
}
```

* **Parameters**:

  * `publisherId`: A valid Taboola PublisherId (e.g., `publisherId`).
  * `userData`: An instance of `TBLUserData` containing user-specific data (email, globalUserId, and consent preferences).
  * `newsParams`: An instance of `TBLNewsParams` containing referral data (pass empty `TBLNewsParams()` for standard flow).
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

You can update user data at any time. When `setUserData` is called, the WebView will automatically reload with the new data - no need to call `initialize` again:

```swift
let userData = TBLUserData(
    email: "user@example.com",
    globalUserId: "user-123",
    allowPersonalization: true,  // Set to false if user opts out
    allowDeviceContext: true      // Set to false to limit device data collection
)
TBLSDK.shared.setUserData(userData)
```

* **Parameters**:
  * `email`: User's email address
  * `globalUserId`: Unique identifier for the user
  * `allowPersonalization`: Controls whether personalized recommendations are enabled (default: true)
  * `allowDeviceContext`: Controls whether device context data is collected (default: true)

### 5. Set News Parameters (Optional)

You can update news parameters after initialization. This is useful when the news tab is opened from different sources (e.g., from a chat tab banner with referral data):

#### Scenario 1: News Tab Opened Directly (Standard Flow)
If the user opens the news tab by tapping on the tab itself, no referral data is needed:

```swift
// User clicks the news tab
let newsParams = TBLNewsParams()
TBLSDK.shared.initialize(publisherId: publisherId, data: userData, newsParams: newsParams, onTaboolaListener: onTaboolaListener)
```

#### Scenario 2: News Tab Opened from Chat Tab Banner (CTB)
If the user opens the news tab by clicking a news banner, you need to pass the referral data:

1. **Get Data from Deeplink**: When the user clicks the CTB, your deeplink will contain a `jd` query parameter (e.g., `line://nv/newsRow?jd=v2_fdfsdf`).
2. **Extract the `jd` value** from the deeplink.
3. **Create the TBLNewsParams object**:

```swift
let newsParams = TBLNewsParams(referralCode: TBLReferralCode.CHAT_PAGE, journeyData: jd)
```

4. **Pass the object to the SDK** using one of these methods:

   **a)** If `initialize` has not been called yet, pass the `newsParams` directly:
   ```swift
   TBLSDK.shared.initialize(publisherId: publisherId, data: userData, newsParams: newsParams, onTaboolaListener: onTaboolaListener)
   ```

   **b)** If `initialize` has already been called, use the `setNewsParams()` function:
   ```swift
   TBLSDK.shared.setNewsParams(newsParams)
   ```

### 6. Cleanup on App Termination

To properly clean up SDK resources when the app terminates, add the following to your AppDelegate:

```swift
func applicationWillTerminate(_ application: UIApplication) {
    TBLSDK.shared.deinitialize()
}
```

This ensures that all SDK resources are properly released when the app is terminated.

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

### Version 1.4.0
- `TBLUserData` now includes `allowPersonalization` and `allowDeviceContext` parameters (both default to `true`) to manage user data collection preferences.
- Removed `setCollectUserData()` function - use `TBLUserData` consent parameters instead.

### Version 1.3.0
- Framework is now static (changed from dynamic).
- Added `newsParams` parameter to `initialize` function for handling referral data from different sources (e.g., chat tab banner).
- Added new `setNewsParams(_ newsParams: TBLNewsParams)` function to update news parameters after initialization.
- Enhanced `setUserData` to automatically reload the WebView with new user data - no need to call `initialize` again.

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
