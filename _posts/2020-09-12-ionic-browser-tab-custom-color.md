---
layout: post
title: "Spring Boot Application Events Explained"
author: nandan
categories: [ionic, cordova]
excerpt: "BrowserTab plugin provides an interface to in-app browser tabs that exist on some mobile platforms, specifically Custom Tabs on Android (including the Chrome Custom Tabs implementation), and SFSafariViewController on iOS."
image: https://miro.medium.com/max/700/1*3A6oECCPwIOUxp1mfbZR9Q.jpeg
tags: [ionic, cordova, angular, featured]
---

Ionic v5 BrowserTab with Custom Tab Color

BrowserTab plugin provides an interface to in-app browser tabs that exist on some mobile platforms, specifically Custom Tabs on Android (including the Chrome Custom Tabs implementation), and SFSafariViewController on iOS.

The plugin we’ll be looking at today is a fork from the official [google/cordova-plugin-browsertab](https://github.com/google/cordova-plugin-browsertab). Since there are no active maintainers of the repository, developers are facing many issues with the plugin.

Link for the fork with fixes and enabling BrowserTab with Custom Color:
[https://github.com/itsLucario/cordova-plugin-browsertab](https://github.com/itsLucario/cordova-plugin-browsertab)

## Steps To Get It Working For Your App

First install the Ionic’s official browser tab JS library, which helps you to get easy access for the Cordova BrowserTab Plugin.

    npm install @ionic-native/browser-tab

After the installation, install the cordova-browser-tab plugin by using the below command:

    cordova plugin add @lucario/cordova-plugin-browsertab --variable CUSTOM_TAB_COLOR_RGB=”#ff0000"

You can change the variable CUSTOM_TAB_COLOR_RGB with whichever color you want in HEX format. Once the plugin gets installed, you can access the plugin in your Angular code as shown below.

    import { BrowserTab } from ‘[@ionic](http://twitter.com/ionic)-native/browser-tab/ngx’;

    constructor(private browserTab: BrowserTab) {

     browserTab.isAvailable()
     .then(isAvailable => {
     if (isAvailable) {
     browserTab.openUrl(‘[https://ionic.io'](https://ionic.io'));
     } else {
     // open URL with InAppBrowser instead or SafariViewController
     }
     });
    }

Although the changing browser tab color feature is merged with master branch, they have not rolled this feature out to npm repository. That is the reason you won’t get custom color working when you try to install it from official repository.

## Points you should keep in mind:

* This plugin can only take one color, which is set using variable CUSTOM_TAB_COLOR_RGB during the installation of the plugin.

* Also, this plugin force opens custom tab with Google Chrome by default, which I haven’t found being done in any of the PR/forks. If the user doesn’t have Google Chrome installed in his/her device, an action choice will be shown. This will give a good user experience cause all links are force opened with Chrome tab without an asking prompt.

* Unnecessary dependency “cordova-plugin-compat” is removed since its deprecated.

[**@lucario/cordova-plugin-browsertab**](https://www.npmjs.com/package/@lucario/cordova-plugin-browsertab)

[**itsLucario/cordova-plugin-browsertab**](https://github.com/itsLucario/cordova-plugin-browsertab)

If this helps don’t forget to leave a comment 😉, it motivates me to write more.
