# Using Kony with Acoustic Mobile Push
This document describes the steps to incorporate Acoustic Mobile Push iOS SDK with Kony project. It assumes the project was exported to konyappiphone.KAR file (Kony Binary ) and then was converted to Xcode project.

A. Open project in Xcode and import AcousticMobilePush.xcframework (Embed & Sign)
    a. In  general tab - make sure Target Membership is checked for KRelease, KProtected and Preview 

B. Create VMAppDelegateLocal.h file with contents:

```
#import "VMAppDelegate.h"
@interface VMAppDelegateLocal : VMAppDelegate
@end
```

C. Create VMAppDelegateLocal.m file with contents:

```
#import "VMAppDelegateLocal.h"
#import <UserNotifications/UserNotifications.h>
#import <AcousticMobilePush/AcousticMobilePush.h>

@implementation VMAppDelegateLocal
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary<UIApplicationLaunchOptionsKey, id> *)launchOptions {
    
    [super application: application didFinishLaunchingWithOptions: launchOptions];
    
    [application registerForRemoteNotifications];
    
    NSUInteger options = 0;
    if(@available(iOS 12.0, *)) {
        options = UNAuthorizationOptionAlert|UNAuthorizationOptionSound|UNAuthorizationOptionBadge|UNAuthorizationOptionCarPlay|UNAuthorizationOptionProvidesAppNotificationSettings;
    } else {
        options = UNAuthorizationOptionAlert|UNAuthorizationOptionSound|UNAuthorizationOptionBadge|UNAuthorizationOptionCarPlay;
    }

    if(@available(iOS 10.0, *)) {
        UNUserNotificationCenter * center = [UNUserNotificationCenter currentNotificationCenter];
        [center requestAuthorizationWithOptions: options completionHandler:^(BOOL granted, NSError * _Nullable error) {
            if(granted) {
                NSLog(@"User Notifications Granted");
            } else if (error) {
                NSLog(@"User Notifications not granted, error: %@", error.localizedDescription);
            }
        }];
        center.delegate = MCENotificationDelegate.sharedInstance;
    }
    
    return true;
}
@end
```

D. Add VMAppDelegateLocal.m to all targets: KRelease, KProtected and Preview

E. Add MceConfig.json file to the project. Make sure all targets: KRelease, KProtected and Preview are checked. Set the appDelegateClass:

```
	"appDelegateClass": "VMAppDelegateLocal",
```

F. Add BackgroundModes capability to all targets: KRelease, KProtected and Preview (as normal)

G. If using location features, add info.plist entries as normal

H. Notification Service Extension 
    a. Add Notification Service Extension to each target (File->New->Target-> select Notification Service Extension.
    b. Import AcousticMobilePushNotification.xcframework (Do Not Embed)
    c. Make sure Target Membership is checked for KRelease, KProtected and Preview targets's general tab
    d. Match the architectures build settings to the main app target's architecture build settings:

1. `Architectures` = `Standard Architectures`
2. `Build Active Architectures Only` = `No`
3. `Valid Architectures` = `$(ARCHS_STANDARD_32_BIT)` `$(ARCHS_STANDARD_64_BIT)` `armv7s`

I. Add "Push Notifications" capability to all targets: KRelease, KProtected and Preview (as normal)

J. Update KonyAppDelegateClassFactory implementation class.
```
#import "KonyAppDelegateClassFactory.h"
#import "VMAppDelegate.h"
@implementation KonyAppDelegateClassFactory

+(Class)appDelegateClass
{
  //  return [VMAppDelegate class];
    return NSClassFromString (@"MCEAppDelegate");
}
@end
```

Explenations:
*MCEAppDelegate is set in KonyAppDelegateClassFactory to be the AppDelegate of the app.
*MCEAppDelegate gets the call from the iOS, it then forwards to the class defined in appDelegateClass. In this case VMAppDelegateLocal. All requests to MCEAppDelegate are forwarded to VMAppDelegateLocal. 
The super call in VMAppDelegateLocal calls it's superclass's implementation, VMAppDelegate.
If the method in class VMAppDelegateLocal doesn't have it defined it will still call up the stack to VMAppDelegate.
