---
word: iOS
title: iOS SDK
order: 11
shared: true
---

iOS Cloud SDK
=======

Particle iOS Cloud SDK enables iOS apps to interact with Particle-powered connected products via the Particle Cloud. It’s an easy-to-use wrapper for Particle REST API. The Cloud SDK will allow you to:

- Manage user sessions for the Particle Cloud (access tokens, encrypted session management)
- Claim/Unclaim devices for a user account
- Get a list of instances of user's Particle devices
- Read variables from devices
- Invoke functions on devices
- Publish events from the mobile app and subscribe to events coming from devices *(Coming Soon)*

All cloud operations take place asynchronously and use the well-known completion blocks (closures for swift) design pattern for reporting results allowing you to build beautiful responsive apps for your Particle products and projects.
iOS Cloud SDK is implemented as an open-source Cocoapod library. See [Installation](#ios-cloud-sdk-installation) section for more details. It works well for both Objective-C and [Swift](#support-for-swift-projects) projects.

**Rebranding notice**

Spark has been recently rebranded as Particle.
Code currently refers to `SparkCloud` and `SparkDevice`, this will soon be replaced with `ParticleCloud` and `ParticleDevice`. A new Cocoapod library will be published and current one will be depracated and point to the new one. This should not bother or affect your code.

**Beta notice**

This SDK is still under development and is currently released as Beta, although tested, bugs and issues may be present, some code might require cleanups.

## Getting Started

- Perform the installation step described under the **Installation** section below for integrating in your own project
- You can also [Download Particle iOS Cloud SDK](https://github.com/spark/spark-sdk-ios/archive/master.zip) and try out the included iOS example app
- Be sure to check [Usage](#ios-cloud-sdk-usage) before you begin for some code examples

## Usage

Cloud SDK usage involves two basic classes: first is `SparkCloud` which is a singleton object that enables all basic cloud operations such as user authentication, device listing, claiming etc. Second class is `SparkDevice` which is an instance represnting a claimed device in the current user session. Each object enables device-specific operation such as: getting its info, invoking functions and reading variables from it.

Here are few examples for the most common use cases to get your started:

#### Logging in to Particle cloud
You don't need to worry about access tokens, SDK takes care of that for you

**Objective-C**
```objc
[[SparkCloud sharedInstance] loginWithUser:@"ido@particle.io" password:@"userpass" completion:^(NSError *error) {
    if (!error)
        NSLog(@"Logged in to cloud");
    else
        NSLog(@"Wrong credentials or no internet connectivity, please try again");
}];
```
---
**Swift**

```swift
SparkCloud.sharedInstance().loginWithUser("ido@particle.io", password: "userpass") { (error:NSError!) -> Void in
    if let e=error {
        println("Wrong credentials or no internet connectivity, please try again")
    }
    else {
        println("Logged in")
    }
}
```
---

#### Get a list of all devices


List the devices that belong to currently logged in user and find a specific device by name:

**Objective-C**

```objc
__block SparkDevice *myPhoton;
[[SparkCloud sharedInstance] getDevices:^(NSArray *sparkDevices, NSError *error) {
    NSLog(@"%@",sparkDevices.description); // print all devices claimed to user

    for (SparkDevice *device in sparkDevices)
    {
        if ([device.name isEqualToString:@"myNewPhotonName"])
            myPhoton = device;
    }
}];
```
---
**Swift**

```swift
var myPhoton : SparkDevice?
SparkCloud.sharedInstance().getDevices { (sparkDevices:[AnyObject]!, error:NSError!) -> Void in
    if let e = error {
        println("Check your internet connectivity")
    }
    else {
        if let devices = sparkDevices as? [SparkDevice] {
            for device in devices {
                if device.name == "myNewPhotonName" {
                    myPhoton = device
                }
            }
        }
    }
}
```
---

#### Read a variable from a Particle device (Core/Photon)
Assuming here that `myPhoton` is an active instance of `SparkDevice` class which represents a device claimed to current user:

**Objective-C**
```objc
[myPhoton getVariable:@"temperature" completion:^(id result, NSError *error) {
    if (!error) {
        NSNumber *temperatureReading = (NSNumber *)result;
        NSLog(@"Room temperature is %f degrees",temperatureReading.floatValue);
    }
    else {
        NSLog(@"Failed reading temperature from Photon device");
    }
}];
```
---

**Swift**
```swift
myPhoton!.getVariable("temperature", completion: { (result:AnyObject!, error:NSError!) -> Void in
    if let e=error {
        println("Failed reading temperature from device")
    }
    else {
        if let temp = result as? Float {
            println("Room temperature is \(temp) degrees")
        }
    }
})
```
---

#### Call a function on a Particle device (Core/Photon)
Invoke a function on the device and pass a list of parameters to it, `resultCode` on the completion block will represent the returned result code of the function on the device

**Objective-C**
```objc
[myPhoton callFunction:@"digitalwrite" withArguments:@[@"D7",@1] completion:^(NSNumber *resultCode, NSError *error) {
    if (!error)
    {
        NSLog(@"LED on D7 successfully turned on");
    }
}];
```
---

**Swift**
```swift
let funcArgs = ["D7",1]
myPhoton!.callFunction("digitalwrite", withArguments: funcArgs) { (resultCode : NSNumber!, error : NSError!) -> Void in
    if (error == nil) {
        println("LED on D7 successfully turned on")
    }
}
```
---

#### List device exposed functions and variables
Functions is just a list of names, variables is a dictionary in which keys are variable names and values are variable types:

**Objective-C**
```objc
NSDictionary *myDeviceVariables = myPhoton.variables;
NSLog(@"MyDevice first Variable is called %@ and is from type %@", myDeviceVariables.allKeys[0], myDeviceVariables.allValues[0]);

NSArray *myDeviceFunctions = myPhoton.functions;
NSLog(@"MyDevice first Function is called %@", myDeviceFunctions[0]);
```
---

**Swift**
```swift
let myDeviceVariables : Dictionary? = myPhoton.variables as? Dictionary<String,String>
println("MyDevice first Variable is called \(myDeviceVariables!.keys.first) and is from type \(myDeviceVariables?.values.first)")

let myDeviceFunction = myPhoton.functions
println("MyDevice first function is called \(myDeviceFunction!.first)")
```
---

#### Get an instance of a device
Get a device instance by its ID:

**Objective-C**
```objc
__block SparkDevice *myOtherDevice;
NSString *deviceID = @"53fa73265066544b16208184";
[[SparkCloud sharedInstance] getDevice:deviceID completion:^(SparkDevice *device, NSError *error) {
    if (!error)
        myOtherDevice = device;
}];
```
---

**Swift**
```swift
var myOtherDevice : SparkDevice? = nil
    SparkCloud.sharedInstance().getDevice("53fa73265066544b16208184", completion: { (device:SparkDevice!, error:NSError!) -> Void in
        if let d = device {
            myOtherDevice = d
        }
    })
```
---

#### Rename a device
you can simply set the `.name` property or use -rename() method if you need a completion block to be called (for example updating a UI after renaming was done):

**Objective-C**
```objc
myPhoton.name = @"myNewDeviceName";
```

_or_
```objc
[myPhoton rename:@"myNewDeviecName" completion:^(NSError *error) {
    if (!error)
        NSLog(@"Device renamed successfully");
}];
```
---

**Swift**
```swift
myPhoton!.name = "myNewDeviceName"
```
_or_
```swift
myPhoton!.rename("myNewDeviceName", completion: { (error:NSError!) -> Void in
    if (error == nil) {
        println("Device successfully renamed")
    }
})
```
---

#### Logout
Also clears user session and access token

**Objective-C**
```objc
[[SparkCloud sharedInstance] logout];
```
---
**Swift**
```swift
SparkCloud.sharedInstance().logout()
```
---

### Additional reference
For additional reference check out the [Reference in Cocoadocs website](http://cocoadocs.org/docsets/Spark-SDK/) for full coverage of `SparkDevice` and `SparkCloud` functions and member variables. In addition you can consult the javadoc style comments in `SparkCloud.h` and `SparkDevice.h` for each public method. If Particle iOS Cloud SDK is integrated in your XCode project you should be able to press `Esc` to get an auto-complete hints for each cloud and device method.

## Installation

Particle iOS Cloud SDK is available through [CocoaPods](http://cocoapods.org). Cocoapods is an easy to use dependacy manager for iOS.
You must have Cocoapods installed, if you don't then be sure to [Install Cocoapods](https://guides.cocoapods.org/using/getting-started.html) before you start:
To install the iOS Cloud SDK, simply add the following line to your Podfile on main project folder:

```ruby
pod "Spark-SDK"
```

and then run `pod update`. A new `.xcworkspace` file will be created for you to open by Cocoapods, open that file workspace file in XCode and you can start interacting with Particle cloud and devices by
adding `#import "Spark-SDK.h"`. (that is not required for swift projects)

## Communication

- If you **need help**, use [Our community website](http://community.particle.io), use the `Mobile` category for dicussion/troubleshooting iOS apps using the Particle iOS Cloud SDK.
- If you are certain you **found a bug**, _and can provide steps to reliably reproduce it_, open an issue, label it as `bug`.
- If you **have a feature request**, open an issue with an `enhancement` label on it
- If you **want to contribute**, submit a pull request, be sure to check out spark.github.io for our contribution guidelines, and please sign the [CLA](https://docs.google.com/a/particle.io/forms/d/1_2P-vRKGUFg5bmpcKLHO_qNZWGi5HKYnfrrkd-sbZoA/viewform).


#### Support for Swift projects
To use iOS Cloud SDK from within Swift based projects [read here](http://swiftalicio.us/2014/11/using-cocoapods-from-swift/).
For a detailed step-by-step help on integrating the Cloud SDK within a Swift project check out this [Particle community posting](https://community.particle.io/t/mobile-sdk-building-the-bridge-from-swift-to-objective-c/12020/1).

The [Apple documentation](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/InteractingWithObjective-CAPIs.html) is an important resource on mixing Objective-C and Swift code, be sure to read through that as well.

_Notice_ that we've included the required bridging header file in the SDK, you just need to copy it to your project add it as the active bridging header file in the project settings as described in the links above.
There's also an [example app](https://github.com/spark/spark-setup-ios-example), this app also demonstrates the Particle DeviceSetup library usage, as well as several Cloud SDK calls.

## License

Particle iOS Cloud SDK is available under the Apache License 2.0. See the LICENSE file for more info.


iOS Device Setup Library
=======

The Particle Device Setup library is meant for integrating the initial setup process of Particle devices in your app.
This library will enable you to easily invoke a standalone setup wizard UI for setting up internet-connect products
powered by a Photon/P0/P1. The setup UI can be easily customized by a customization proxy class available to the user
that includes: look & feel, colors, fonts as well as custom brand logos and instructional video for your product. There are good defaults if you don’t set these properties, but you can override the look and feel as needed to suit the rest of your app.

As you may have heard, the wireless setup process for the Photon uses very different underlying technology from the Core. Where the Core used Smart Config, the Photon uses what we call “soft AP” — the Photon advertises a Wi-Fi network, you join that network from your mobile app to exchange credentials, and then the Photon connects using the Wi-Fi credentials you supplied.

With the Device Setup library, you make one simple call from your app, for example when the user hits a “setup my device” button, and a whole series of screens then guides the user through the soft AP setup process. When the process finishes, the user is back on the screen where she hit the “setup my device” button, and your code has been passed an instance of the device she just setup and claimed.

## Usage

### Basic
Import `SparkSetup.h` in your view controller implementation file, and invoke the device setup wizard by:

```objc
SparkSetupMainController *setupController = [[SparkSetupMainController alloc] init];
[self presentViewController:setupController animated:YES completion:nil];
```

Alternatively if your app requires separation between the Particle cloud authentication process and the device setup process you can call:

```objc
SparkSetupMainController *setupController = [[SparkSetupMainController alloc] initWithAuthenticationOnly:YES];
[self presentViewController:setupController animated:YES completion:nil];
```

This will invoke Particle Cloud authentication (login/signup/password recovery screens) only
after user has successfully logged in or signed up, control will be returned to the calling app.
If an active user session already exists control will be returned immediately.


### Customization

Customize setup look and feel by accessing the SparkSetupCustomization singleton appearance proxy `[SparkSetupCustomization sharedInstance]`
and modify its properties. All properties are optional.

#### Product/brand info:

You can modify the brand and product related info/images by assigning to these properties:

```objc
 NSString *deviceName;          // Device/product name
 UIImage *deviceImage;          // Device/product image

 NSString *brandName;           // Your brand name
 UIImage *brandImage;          	// Your brand logo to fit in header of setup wizard screens
 UIColor *brandImageBackgroundColor;    // brand logo background color
 NSString *welcomeVideoFilename;        // Welcome screen instructional video
 NSString *appName;                     // Your setup app name
```
---

#### Technical info:

Modify product technical data by assigning to these properties:

```objc
 NSString *modeButtonName;              // The mode button name on your product
 NSString *listenModeLEDColorName;      // The color of the LED when product is in listen mode
 NSString *networkNamePrefix;           // The SSID prefix of the Soft AP Wi-Fi network of your product while in listen mode
```
---


#### Links for legal/technical stuff:

You can edit links to your TOS and privacy policy as well as a troubleshooting page. Links will supersede static pages if supplied.

```objc
 NSURL *termsOfServiceLinkURL; // URL for terms of service of the app/device usage
 NSURL *privacyPolicyLinkURL;  // URL for privacy policy of the app/device usage
 NSURL *troubleshootingLinkURL; // URL for troubleshooting text of the app/device usage

 NSString *termsOfServiceHTMLFile; // Static HTML file for terms of service of the app/device usage
 NSString *privacyPolicyHTMLFile;  // Static HTML file for privacy policy of the app/device usage
 NSString *troubleshootingHTMLFile; // Static HTML file for troubleshooting text of the app/device usage
```
---

#### Look & feel:

Edit the looks of the setup wizard screens by assigning values to these properties:

```objc
 UIColor *pageBackgroundColor;     // setup screens background color
 UIImage *pageBackgroundImage;     // optional background image for setup screens
 UIColor *normalTextColor;         // normal text color
 UIColor *linkTextColor;           // link text color (will be underlined)
 UIColor *buttonBackgroundColor;   // Buttons background color
 UIColor *buttonTextColor;         // Buttons text color
 NSString *normalTextFontName;     // Customize normal setup text font - include OTF/TTF file in project
 NSString *boldTextFontName;       // Customize bold setup text font - include OTF/TTF file in project
 NSString *headerTextFontName;     // Customize header setup text font - include OTF/TTF file in project
 CGFloat fontSizeOffset;           // Set offset of font size so small/big fonts can be fine-adjusted
```
---

#### Organization:

Setting `organization=YES` will enable organization mode. You can modify organization name via the `organizationName` property.

```objc
 BOOL organization;                 // enable organization mode - activation codes, other organizational APIs
 NSString *organizationName;        // organization name
```
---

### Advanced

You can get an active instance of `SparkDevice` by making your viewcontroller conform to protocol `<SparkSetupMainControllerDelegate>` when setup wizard completes:

```objc
-(void)SparkSetupViewController:(SparkSetupMainController *)controller didFinishWithResult:(SparkSetupMainControllerResult)result device:(SparkDevice *)device;
```
---

method will be called, if `(result == SparkSetupMainControllerResultSuccess)` the device parameter will contain an active `SparkDevice` instance you can interact with
using the [Particle Cloud SDK](https://cocoapods.org/pods/Spark-SDK).

#### Support for Swift projects
To use Particle DeviceSetup library from within Swift based projects [read here](http://swiftalicio.us/2014/11/using-cocoapods-from-swift/),
also be sure the check out [Apple documentation](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/InteractingWithObjective-CAPIs.html) on this matter.

### Example
Usage example app (in Swift) can be found [here](https://github.com/spark/spark-setup-ios-example/). Example app demonstates - invoking the setup wizard, customizing its UI and using the returned ParticleDevice instance once
setup wizard completes (delegate). Feel free to contribute to the example by submitting pull requests.

### Reference

Check out the [Reference in Cocoadocs website](http://cocoadocs.org/docsets/SparkSetup/) or consult the javadoc style comments in `SparkSetupCustomization.h` and `SparkSetupMainController.h` for each public method or property.
If Particle Device Setup library installation completed successfully - you should be able to press `Esc` to get an auto-complete hints from XCode for each public method or property in the library.

## Requirements / limitations

- iOS 7.1+ supported
- Currently setup wizard displays on portait mode only.
- XCode 6.0 and up is required

## Installation

Particle-Setup is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod "SparkSetup"
```
---

## Communication

- If you **need help**, use [Our community website](http://community.particle.io)
- If you **found a bug**, _and can provide steps to reliably reproduce it_, open an issue.
- If you **have a feature request**, open an issue.
- If you **want to contribute**, submit a pull request.


## License

Particle Device Setup library is available under the Apache License 2.0. See the LICENSE file for more info.
