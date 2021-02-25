# Skyhook Location SDK (Native)

## Prerequisites

Operating systems:

* Windows XP Service Pack 2 or 3, Windows Vista, Windows 7, 8, 10
* Mac OS X 10.6 - 11.0
* Linux 2.6+ with wireless extensions or `nl80211` and root access

Hardware:

* Wi-Fi network card for location based on Wi-Fi networks
* Cell modem for location based on cell networks

## Importing SDK

Import the header file in your code:
```c
#include <wpsapi.h>
```

Link your executable against the `wpsapi` library:

* On Windows link against `wpsapi.lib` and make sure `wpsapi.dll` is accessible to the executable
* On Mac OS X link against `libwpsapi.dylib`
* On Linux link against `libwpsapi.so`

## API summary

The WPS API can be summarized to the following calls:

* `WPS_load()`
* `WPS_set_key()`
* `WPS_location()` or `WPS_periodic_location()`
* `WPS_unload()`

### Initialization

Invoke `WPS_load()` before making any other WPS API calls.

When WPS API is no longer needed, or the application is terminating, call `WPS_unload()` to free resources.

### Registration

Prior to using any location API calls the application should set the API key. This is accomplished by calling `WPS_set_key()`.

### API modes

The API works in 2 major modes: network centric and tiling.

#### Network-centric mode

In the network-centric model the API issues calls to a remote server to determine a location. This is the default mode.

#### Tiling mode

In the tiling mode the SDK downloads, from the server, a small portion of the database so the device can automonously determine its location, without further need to contact the server.

This mode is generally recommended mode when fast location tracking is required or when network is not constantly available.

The tiling mode is activated by calling `WPS_set_tiling()`.

### Geofencing

The API supports geofencing that allows the application to be notifed when the user enters, or leaves, a defined zone.

This feature is activated by calling the `WPS_geofence_set()` API. Once defined the application runs `WPS_periodic_location()` to monitor the device's location. When `WPS_periodic_location()` determines the device is moving inside a geofence – ie. a first location is determined to be outside, followed by a location determined to be inside the geofence – it triggers a `WPS_GEOFENCE_ENTER` callback. Similarly when `WPS_periodic_location()` determines the device is moving outside a geofence – ie. a first location is determined to be inside, followed by a location determined to be outside the geofence – it triggers a `WPS_GEOGENCE_LEAVE` callback.

When an application no longer desires to monitor a geofence it can call `WPS_geofence_cancel()` or `WPS_geofence_cancel_all()`.

Note: geofences can be added or removed while `WPS_periodic_location()` is running from the `WPS_periodic_location()` callback. The `WPS_periodic_location()` callback itself will still be invoked whether the location calculated corresponds to any defined geofence.

### Offline location

The API supports *offline* location that allows the application to determine the location of the device even offline and outside of tile coverage by collecting a *token* that can be replayed when the device is once again online. Offline tokens are only valid for 90 days after they are generated. Attempting to redeem a token more than 90 days old will result in an error.

## API reference

Check the [full API reference](https://skyhookwireless.github.io/skyhook-location-native/doxygen/html) for more information on APIs that are exposed by the SDK.

## Reference application

A reference application named `wpsapitest` is provided with the SDK package. You can run it out of the box on your system to test location determination capabilities of the SDK.

Start the app without arguments to see all options that are supported.

**Note:** make sure to run `wpsapitest` *as root* on Linux.

### One shot call

When executed with a single API key argument, the app will determine a location once:
```sh
./wpsapitest -k <MY_KEY>
```

Example output:
```
42.352017, -71.047445   +/-9m   2+0+0  62ms
91 Farnsworth St
Boston, MA 02210
```

### Periodic location

Execute the following command to run periodic location updates with 1 second period indefinitely (`Ctrl+C` to interrupt), with tiling enabled:
```sh
./wpsapitest -k <MY_KEY> -p 1 -t <tile_dir>
```

Using existing tiles in *offline mode*:
```sh
./wpsapitest -k <MY_KEY> -p 1 -t <tile_dir> -o
```

Note that the `tile_dir` must pre-exist in the file system for tiling to work. Examples below will use `/tmp/tiles` as a directory to store tiles.

Example output for the above commands:
```
tiling directory: /tmp/tiles
42.352091, -71.047431   +/-122m 2+0+0  66ms
    0.0km/h
42.352091, -71.047431   +/-122m 2+0+0  67ms
    0.0km/h
42.352091, -71.047431   +/-122m 2+0+0  67ms
    0.0km/h
42.352091, -71.047431   +/-122m 2+0+0  67ms
    0.0km/h
```

### CSV output format

The app supports printing location data in the CSV format, which may be useful for post-processing the output (e.g. for plotting locations on a map).

In order to enable the CSV-fromatted output, use the `-V` or `--output-csv` switch:
```sh
./wpsapitest -k <MY_KEY> -p 1 -t /tmp/tiles -V
```

Output:
```
tiling directory: /tmp/tiles
1594766000,42.352091,-71.047431,122
1594766001,42.352091,-71.047431,122
1594766002,42.352091,-71.047431,122
1594766003,42.352091,-71.047431,122
1594766004,42.352091,-71.047431,122
1594766005,42.352091,-71.047431,122
1594766006,42.352091,-71.047431,122
1594766007,42.352091,-71.047431,122
```

Note that unformatted status messages like `tiling directory: /tmp/tiles` are printed to `stderr`, so you can collect location-only data by redirecting `stdout` to a file:
```sh
./wpsapitest -k <MY_KEY> -p 1 -t /tmp/tiles -V > locations.csv
```

### Offline location

Periodically generate and store tokens in offline mode, for 10 seconds:
```sh
./wpsapitest -k <MY_KEY> -K q1w2e3r4 -O token_dir -p 1 -i 10 -o
```

Output:
```
offline mode enabled
*** failure (6)!
saved offline token: /tmp/offline/1594750740.tok
*** failure (6)!
saved offline token: /tmp/offline/1594750742.tok
*** failure (6)!
saved offline token: /tmp/offline/1594750743.tok
*** failure (6)!
saved offline token: /tmp/offline/1594750745.tok
*** failure (6)!
saved offline token: /tmp/offline/1594750746.tok
*** failure (6)!
saved offline token: /tmp/offline/1594750747.tok
*** failure (6)!
saved offline token: /tmp/offline/1594750748.tok
```

Redeem previously stored tokens from a directory and resolve locations retroactively against the server:
```sh
./wpsapitest -k <MY_KEY> -K q1w2e3r4 -O token_dir
```

Output:
```
found 7 offline tokens in directory: /tmp/offline
42.352017, -71.047445   +/-9m   2+0+0  37082ms
42.352017, -71.047445   +/-9m   2+0+0  35079ms
42.352017, -71.047445   +/-9m   2+0+0  34082ms
42.352017, -71.047445   +/-9m   2+0+0  32081ms
42.352017, -71.047445   +/-9m   2+0+0  32081ms
42.352017, -71.047445   +/-9m   2+0+0  31082ms
42.352017, -71.047445   +/-9m   2+0+0  30082ms
```

## Logging

To turn logging on, define an environment variable named `WPS_LOG_CONFIGURATION` that contains the path to the `wpslog.properties` file.

Here's an example of a `wpslog.properties` file:
```
DEBUG,wpslog.txt
```

Note that the file should not contain a newline at the end.

You can also redirect logging to `stdout` or `stderr`:
```
DEBUG,stdout
```

The logging level can be one of the following:

* ERROR
* WARN
* NOTICE
* INFO
* DEBUG

## Location settings

Starting from version 4.9.2 WPS API will respect the system-wide location permission setting (applies only to Windows 8+ and OS X 10.7 or later). If location services are disabled, WPS API will return `WPS_ERROR_LOCATION_SETTING_DISABLED` in `WPS_location`, `WPS_periodic_location`, `WPS_certified_location` and `WPS_tune_location` calls.

In addition, starting from version 5.0.0 WPS API may return `WPS_ERROR_LOCATION_NOT_PERMITTED` if the particular application is not permitted to determine Wi-Fi based location.

See more information on your platform below.

### Windows

The system-wide location setting on Windows can be found in *Control Panel - Location Settings*. It is enabled by default.

If you receive `WPS_ERROR_LOCATION_SETTING_DISABLED` from WPS API in your application, that means the option *Turn on the Windows Location platform* or *Allow apps to access your locatiaon* is turned off by the user.

You might want to handle this by notifying the user that the location platform is currently disabled in the system and hence the application will not be able to determine location. You can also navigate the user to the *Location Settings* page directly so the user may re-enable the location setting back.

Below is a sample code snippet:
```c
if (MessageBox(NULL,
               L"Location is disabled in system. Do you want to open Control Panel to enable it?",
               L"My Application",
               MB_YESNO | MB_ICONQUESTION) == IDYES)
{
    ShellExecute(NULL, NULL, L"control", L"-name Microsoft.LocationSettings", NULL, SW_SHOW);
}
```

You can check [here](http://msdn.microsoft.com/en-us/library/windows/desktop/cc144191(v=vs.85).aspx) on more ways to pop up Control Panel items programmatically.

Note: Windows 8 or higher may show a prompt to enable location services the very first time when location is requested via Location API (check the Dialogues for enabling location section here). WPS API will never cause the system to show any kind of prompts.

### Mac OS X

OS X stores location settings under *System Preferences - Security & Privacy - Privacy - Location Services*. In addition to the global setting (*Enable Location Services*) it also lists applications permitted to determine location.

On OS X versions prior to 10.15 (Catalina) WPS API will not require the application to be listed and will grant permission whenever just the global setting is enabled.

For information on the latest OS X version, see the [Catalina section](#location-authorization-in-macos-catalina) below.

If you receive `WPS_ERROR_LOCATION_SETTING_DISABLED` from WPS API, that means that the global setting is disabled by the user. You might want to handle this by notifying the user that the location services are currently disabled in the system and hence the application will not be able to determine location. You can also navigate the user to the location settings page directly so the user may re-enable the location services back.

Below is a sample Apple script:
```
tell application "System Preferences"
    set securityPane to pane id "com.apple.preference.security"
    tell securityPane to reveal anchor "Privacy_LocationServices"
    activate
end tell
```

You can save it to a file (e.g. `locationsettings.scpt`) and run:
```sh
osascript locationsettings.scpt
```

If you want to implement it in Objective C or Swift, refer to the [Scripting Bridge documentation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ScriptingBridgeConcepts/UsingScriptingBridge/UsingScriptingBridge.html).

#### Location authorization in macOS Catalina

Due to changes introduced in macOS Catalina, WPS API will now require that the application has obtained an authorization to determine a high precision location, such as a location based on Wi-Fi access points. If the authorization hasn't been granted, WPS API will return `WPS_ERROR_LOCATION_NOT_PERMITTED`.

The application can rely on the SDK to obtain permission automatically when needed. This will result in the SDK showing a prompt to the user when location is being determined for the first time.

For production level apps it is recommended that the app handles location authorization on its own. This allows to provide the best user experience by integrating the location permission pop-up into the app's UI flow. Before displaying the pop-up, the app could provide explanation on why the permission is needed.

The code to request a location permission from the user would look like this:
```objc
CLLocationManager* locationManager = [[CLLocationManager alloc] init];
locationManager.delegate = delegate;
[locationManager requestAlwaysAuthorization];
```

Note that the calling thread must have a runloop running for the permission request to work properly.

Refer to [Apple's documentation](https://developer.apple.com/documentation/corelocation/cllocationmanager/1620551-requestalwaysauthorization) for more information.

#### Command line apps in macOS Catalina

For commmand line apps in macOS Catalina or later, extra steps are required to obtain the authorization to determine a Wi-Fi based location from WPS API:

* If the app makes any calls to `CLLocationManager`, those must be made *after* calling `WPS_load()`
* An `Info.plist` file must be placed in the same directory where the executable is located, with the `CFBundleIdentifier` and `CFBundleExecutable` fields populated

Please see the [quick start project](https://github.com/SkyhookWireless/location-quick-start-native) as an example of a command line application utilizing the SDK to determine Wi-Fi based location on macOS 10.15+.

#### Hardened Runtime

If your application is using [Hardened Runtime](https://developer.apple.com/documentation/security/hardened_runtime), make sure to enable access to Location in Xcode settings:

* Signing & Capabilities - Hardened Runtime - Resource Access - Location

The following key will be added in your `.entitlements` file:
```xml
    <key>com.apple.security.personal-information.location</key>
    <true/>
```

#### App Sandbox

If you application is using the [App Sandboxing](https://developer.apple.com/app-sandboxing) feature, enable the following options in Xcode settings:

* Signing & Capabilities - App Sandbox - Network - Outgoing Connections (Client)
* Signing & Capabilities - App Sandbox - App Data - Location

The following keys will be added in your `.entitlements` file:
```xml
    <key>com.apple.security.app-sandbox</key>
    <true/>
    <key>com.apple.security.network.client</key>
    <true/>
    <key>com.apple.security.personal-information.location</key>
    <true/>
```

## Legacy Changelog (non-GitHub releases prior to 5.0)

### Version 4.9.9

* Improved accuracy in location tracking mode

### Version 4.9.8

* Introduced `WPS_set_auth_token()`

### Version 4.9.7

* Improved support for customer proxy servers

### Version 4.9.6

* Improved tiling coverage over ethernet connection on Linux
* Improved stability on Windows
* Fixed some issues when SDK was used in a Windows NT service

### Version 4.9.5

* Introduced `WPS_set_tunable()`

### Version 4.9.4

* Extended the street address lookup with a new geospatial layer which represents a smaller township or municipal names found within a larger city region. This layer is sourced from open-source data without modification and more closely aligns with general jurisdictional and town boundaries.

### Version 4.9.3

* Optimized network load and power consumption in tiling mode
* Improved geofencing in UMTS and LTE networks
* Introduced `WPS_load()` and `WPS_unload()` calls
* Fixed a buffer overflow issue on Linux happening on very large Wi-Fi scans
* Switched to Secure Transport API on OS X and removed the pre-built OpenSSL libraries from the SDK bundle

### Version 4.9.2

* Added a check to respect the system-wide location permission settings. See Location Settings for more information.

### Version 4.9.1

* Added signed certified location
* Improved power consumption
* Improved registration for devices without Wi-Fi or Cell adapters. This feature enhances the usability of `WPS_ip_location()`.
* Added support for Raspbian Linux

### Version 4.9

* Introduced key-based authentication

### Version 4.8

* Added certified location
* Improved power consumption during geofencing
* Two new geofence types "INSIDE" and "OUTSIDE" for cases where immediate triggering is desired
* Fixed an edge case that could result in the client downloading extra tile data
* Improved accuracy of location on slow scanning devices

### Version 4.7.6

* Improved memory management
* Fixed a bug that could negatively affect time to fix on some CDMA devices

### Version 4.7.5

* Added support for location based on LTE cell towers
* Improved the accuracy of all cell locations
* Fixed a bug in tiling when venue and tuned locations fall on different sides of a tile boundary

### Version 4.7

* Added support for tuning locations with tiling enabled
* Added support for coarse loation based on region codes (known as LACs) provided by the network

### Version 4.6

* Improvements in power consumption and accuracy for geofences
* Enhanced indoor location for surveyed sites
* Added offline location

### Version 4.5

Version 4.5 was never released publicly.

### Version 4.4

* Added geofencing
* Improved long period support
* Improvements to positioning in remote mode

### Version 4.3

* Improved memory usage when downloading tiles

### Version 4.2

* Added auto-registration
* Improved offline location coverage

### Version 4.1

* Improved first fix accuracy
* Improved location when stationary
* Various improvements to hybrid algorithm, particularly in tracking mode

### Version 4.0

* Optimized power management
* Better data and bandwidth utilization
* Improvements to hybrid positioning
* Inertial Navigation System

### Version 3.4

* Added support for Windows 7, Windows Mobile 6, Mac OS X 10.6 (Snow Leopard)

### Version 3.3

* Replaced `WPS_tiling()` with `WPS_set_tiling()`
* `WPS_set_*` calls can now be made while `WPS_periodic_location()` or `WPS_location()` is in progress

### Version 3.2

* Added `ncell` and `nsat` to `WPS_Location`
* Added `altitude` to `WPS_Location`

### Version 3.1

* Added `WPS_version()`

### Version 3.0

* Added `WPS_tune_location()`
* Added `WPS_ERROR_TIMEOUT`

### Version 2.7

* Added `WPS_set_user_agent()`

### Version 2.6.1

* Changed return type from `int` to `WPS_Continuation` for `WPS_LocationCallback`

### Version 2.6

* Added `WPS_tiling()`
* Added `WPS_set_tier2_area()`

### Version 2.5

Official release.

### Version 2.4

* Added mixed-mode – local location determination if possible
* Added `WPS_periodic_location()`
* Added `WPS_set_local_files_path()`
* Changed returned code from `WPS_set_proxy()` and `WPS_set_server_url()` to void
* Added `speed`, `bearing` and `nap` to `WPS_Location`
* Removed `hpe` from `WPS_IPLocation`

### Version 2.3.1

* Added `WPS_register_user()`

### Version 2.3

* Faster scanning
* Caching to allow for better response time
* `WPSScanner.dll` is no longer needed

## Limited Use License

Copyright 2005-present Skyhook, Inc. All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted subject to the following:

Use and redistribution is subject to the Software License and Development Agreement, available at www.skyhookwireless.com.
Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Patents

Please visit [the website](https://www.skyhook.com/patents) to see the full list of patents issued to Skyhook.

## Troubleshooting

For any questions, please sign into your My Skyhook account at: [my.skyhook.com](https://my.skyhook.com).
