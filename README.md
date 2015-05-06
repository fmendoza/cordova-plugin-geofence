# Cordova Geofence Plugin

[![Code Climate](https://codeclimate.com/github/cowbell/cordova-plugin-geofence/badges/gpa.svg)](https://codeclimate.com/github/cowbell/cordova-plugin-geofence)

iOS Build [![Build Status](https://travis-ci.org/cowbell/cordova-plugin-geofence.svg?branch=master)](https://travis-ci.org/cowbell/cordova-plugin-geofence)

Plugin to monitor circular geofences using mobile devices. The purpose is to notify user if crossing the boundary of the monitored geofence.

*Geofences persists after device reboot. You do not have to open your app first to monitor added geofences*

##Example application

Check out our [example application](https://github.com/cowbell/ionic-geofence) built with ionic framework.

## Installation

From master
```
cordova plugin add https://github.com/cowbell/cordova-plugin-geofence
```

Latest stable version

```
cordova plugin add cordova-plugin-geofence
```

## Removing the Plugin from project

Using cordova CLI

```
cordova plugin rm com.cowbell.cordova.geofence
```

## Supported Platforms

- Android
- iOS >=7.0
- Windows Phone 8.1
    - using Universal App (cordova windows platform)
    - using Silverlight App (cordova wp8 platform retargeted to WP 8.1)

# Using the plugin

Cordova initialize plugin to `window.geofence` object.

## Methods

- `window.geofence.initialize(onSuccess, onError)`
- `window.geofence.addOrUpdate(geofences, onSuccess, onError)`
- `window.geofence.remove(onSuccess, onError)`
- `window.geofence.removeAll(onSuccess, onError)`
- `window.geofence.getWatched(onSuccess, onError)`

All methods returning promises, but you can also use standard callback functions.

For listening of geofence transistion you can override onTransitionReceived method
- `window.geofence.onTransitionReceived(geofences)`

## Constants

- `TransitionType.ENTER` = 1
- `TransitionType.EXIT` = 2
- `TransitionType.BOTH` = 3

## Plugin initialization

The plugin is not available until `deviceready` event is fired.

```javascript
document.addEventListener('deviceready', function () {
    // window.geofence is now available
    window.geofence.initialize();
}, false);
```

## Adding new geofence to monitor

```javascript
window.geofence.addOrUpdate({
    id:             String, //A unique identifier of geofence
    latitude:       Number, //Geo latitude of geofence
    longitude:      Number, //Geo longitude of geofence
    radius:         Number, //Radius of geofence in meters
    transitionType: Number, //Type of transition 1 - Enter, 2 - Exit, 3 - Both
    notification: {         //Notification object
        id:             Number, //optional should be integer, id of notification
        title:          String, //Title of notification
        text:           String, //Text of notification
        smallIcon:      String, //Small icon showed in notification area, only res URI
        icon:           String, //icon showed in notification drawer
        openAppOnClick: Boolean,//is main app activity should be opened after clicking on notification
        vibration:      [Integer], //Optional vibration pattern - see description
        data:           Object  //Custom object associated with notification
    }
}).then(function () {
    console.log('Geofence successfully added');
}, function (reason) {
    console.log('Adding geofence failed', reason);
});
```
Adding more geofences at once
```javascript
window.geofence.addOrUpdate([geofence1, geofence2, geofence3]);
```

Geofence overrides the previously one with the same `id`.

*All geofences are stored on the device and restored to monitor after device reboot.*

Notification overrides the previously one with the same `notification.id`.

## Notification vibrations

You can set vibration pattern for the notification or disable default vibrations.

To change vibration pattern set `vibrate` property of `notification` object in geofence.

###Examples

```
//disable vibrations
notification: {
    vibrate: [0]
}
```

```
//Vibrate for 1 sec
//Wait for 0.5 sec
//Vibrate for 2 sec
notification: {
    vibrate: [1000, 500, 2000]
}
```

###Platform quirks

Fully working only on Android.

On iOS vibration pattern doesn't work. Plugin only allow to vibrate with default system pattern.

Windows Phone - current status is TODO

## Notification icons

To set notification icons use `icon` and `smallIcon` property in `notification` object.

As a value you can enter:
- name of native resource or your application resource e.g. `res://ic_menu_mylocation`, `res://icon`, `res://ic_menu_call`
- relative path to file in `www` directory e.g. `file://img/ionic.png`

`smallIcon` - supports only resources URI

###Examples

```
notification: {
    smallIcon: 'res://my_location_icon',
    icon: 'file://img/geofence.png'
}
```

###Platform quirks

Works only on Android platform so far.

## Removing

Removing single geofence
```javascript
window.geofence.remove(geofenceId)
    .then(function () {
        console.log('Geofence sucessfully removed');
    }
    , function (reason){
        console.log('Removing geofence failed', reason);
    });
```
Removing more than one geofence at once.
```javascript
window.geofence.remove([geofenceId1, geofenceId2, geofenceId3]);
```

## Removing all geofences

```javascript
window.geofence.removeAll()
    .then(function () {
        console.log('All geofences successfully removed.');
    }
    , function (reason) {
        console.log('Removing geofences failed', reason);
    });
```

## Getting watched geofences from device

```javascript
window.geofence.getWatched().then(function (geofencesJson) {
    var geofences = JSON.parse(geofencesJson);
});
```

## Listening for geofence transitions

```javascript
window.geofence.onTransitionReceived = function (geofences) {
    geofences.forEach(function (geo) {
        console.log('Geofence transition detected', geo);
    });
};
```

## When the app is opened via Notification click

Android, iOS only

```javascript
window.geofence.onNotificationClicked = function (notificationData) {
    console.log('App opened from Geo Notification!', notificationData);
};
```

#Example usage

Adding geofence to monitor entering Gliwice city center area of radius 3km

```javascript
window.geofence.addOrUpdate({
    id:             "69ca1b88-6fbe-4e80-a4d4-ff4d3748acdb",
    latitude:       50.2980049,
    longitude:      18.6593152,
    radius:         3000,
    transitionType: TransitionType.ENTER,
    notification: {
        id:             1,
        title:          "Welcome in Gliwice",
        text:           "You just arrived to Gliwice city center.",
        openAppOnClick: true
    }
}).then(function () {
    console.log('Geofence successfully added');
}, function (reason) {
    console.log('Adding geofence failed', reason);
})
```

# Platform specifics

##iOS

Plugin is written in Swift. All xcode project options to enable swift support are set up automatically after plugin is installed.

##Windows phone 8.1

Plugin can be used with both windows phone 8.1 type projects Univeral App, Silverlight App.

In order to use toast notifications you have to enable this feature in appxmanifest file either using UI in Visual Studio or edit file setting attribute **ToastCapable="true"** in **m3:VisualElements** node under Package/Applications/Application.

If you are retargeting WP 8.0 to WP 8.1 you need to register background task to perform geofence notifications. Register it via UI in Visual Studio or add following code under Package/Applications/Application/Extensions

```xml
<Extension Category="windows.backgroundTasks" EntryPoint="GeofenceComponent.GeofenceTrigger">
    <BackgroundTasks>
        <m2:Task Type="location" />
    </BackgroundTasks>
</Extension>
```

##License

This software is released under the [Apache 2.0 License](http://opensource.org/licenses/Apache-2.0).

© 2014 Cowbell-labs. All rights reserved
