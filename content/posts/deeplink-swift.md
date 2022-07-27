+++
title = "Deeplink Swift"
date = "2022-07-26T21:34:32+05:45"
author = "Utsav Shrestha"
authorTwitter = "" #do not include @
cover = ""
tags = ["Swift", "Deeplink", "universal link", "url scheme"]
keywords = ["Swift", "Deeplink", "universal link", "url scheme"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

# Deeplinking is iOS

Deep links are a form of link that directs consumers straight to an app, as opposed to a website or a shop. They are used to take visitors directly to specified in-app destinations, saving them the time and effort of manually accessing a page, so considerably enhancing the user experience.

Deep linking does this by defining a special URL scheme (iOS Universal Links) or intent URL (on Android devices) that launches your software if it is already installed. Deep links may also be configured to lead viewers to certain events or web sites, which can connect into campaigns you may undertake.

In this article we will discuss deeplinks in iOS

IOS provides two methods for creating deep linking: URL scheme and Universal Links. While URL schemes are a well-known method for deep linking, Apple's new Universal links allow you to seamlessly connect your website and app using the same connection.

Checkout this comparison table

### URL Schemes

| Pros                             | Cons                                          |
| -------------------------------- | --------------------------------------------- |
| Easy to implement out of the box | Will always ask for permission (pop up)       |
| No backend required              | Users are not familiar with the URL structure |
|                                  | Won't work if app is not installed            |

### Universal Links

| Pros                                                        | Cons                      |
| ----------------------------------------------------------- | ------------------------- |
| Won't ask for permission                                    | More complex to implement |
| Familiar URL structure                                      |                           |
| Handled by built in default browser if app is not installed |                           |

Note that if the app is not installed, neither of these alternatives will send you to the App Store. IOS will construct the connection between these URLs and the app based on the app's metadata after the app is installed; if the app is not installed, the metadata is not present and the connection is thus never made. I will write a separate post to address redirection to individual app stores.

## URL Schemes

URL Schemes implementation is simple. Simply inform the application of the scheme you wish to utilize. Open Xcode, navigate to Project Settings -> Info, and add a new URL scheme to the 'The URL Types' section. Add something similar to com.myApp, and you're done. Install your app, then open the app's notes and put com.myApp in the search bar.

Wait... nothing is happening. To make IOS identify this as a link, you must adhere to the URL syntax. Specifically, this is scheme://resource. Therefore, we type com.myApp://main. Now, when we hit enter, we can see that IOS has identified the link, and when we click it, a pop-up will request authorization to launch MyApp from Notes.

What if we wish to reroute the user within the application? If the user hits com.myApp://profile, the profile should be displayed. If the user taps com.myApp://resetpassword, the reset password page should be displayed. If we wish to respond once within the app to this URL, we must implement a method within the AppDelegate.

```swift
func application(_ app: UIApplication, open url: URL,
                 options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
    if let scheme = url.scheme,
        scheme.localizedCaseInsensitiveCompare("com.myApp") == .orderedSame,
        let view = url.host {

        var parameters: [String: String] = [:]
        URLComponents(url: url, resolvingAgainstBaseURL: false)?.queryItems?.forEach {
            parameters[$0.name] = $0.value
        }

        redirect(to: view, with: parameters)
    }
    return true
}
```

This way com.myApp://profile?user=Utsav will result in:

```
url.scheme = “com.myApp”
url.host = “profile”
parameters = [ “user” : Utsav ]
```

This concludes URLSchemes. Remember that this URL will not work if the app is not installed, therefore IOS will attempt to open the link in Safari, which will clearly go to nowhere, leaving the user perplexed and staring at a white screen. We have a second Universal Links to solve this.

## Universal Links

Universal linkages are rather more intricate. We want iOS to associate a website's URL with our app. Nonetheless, this is not so simple. Imagine this scenario. Our favourite application "Redd1t" is identical to the popular social network "Reddit" in every regard. Redd1t specifies in its metadata that if a user visits reddit.com, they should be sent to their app. Recognize the issue here? How should IOS validate that Redd1t is the legitimate owner of www.reddit.com? Obviously, ask https://www.reddit.com! Reddit.com must give a resource that identifies his IOS application. And it achieves this in the following manner: IOS will send a request to https://www.reddit.com/apple-app-site-association and expect to receive a JSON response. The JSON should be formatted as follows:

```json
{
    “applinks”: {
        “apps”: [],
        “details”: [
            {
                “appID”: “T5TQ36Q2SQ.com.reddit.production”,
                “paths”: [“*”],
            }
        ]
    }
}
```

Let's analyze this for a moment.

Applinks suggest that this is definitely for Universal Link.

Apps should be an empty array (Apple: "The apps key in an apple-app-site-association file must be present, and its value must be an empty array) This is probable because certain JSON types are also used for purposes other than universal links, but we will not take this into account.
The details will then include an array of your applications and a mapping of each subpath to the corresponding application.
For each app, you must provide an appID field that is derived by concatenating your teamID with your app's bundleID. If your TeamID is 123456 and your AppID is com.myApp, the resulting value is 123456.com.myApp.

In the pathways field, an array of string expressions indicating the paths corresponding to this application. If you want myApp.com/store to launch a different app than myApp.com/maps, for instance, you may define that `*` is a wildcard for any string and ? is a wildcard for any character. As there is only one application, any subpath will go here, therefore the `*`. If you wish to exclude a subpath, just prefix it with NOT.

```json
[ “/wwdc/news/”, “NOT /videos/wwdc/2010/*”, “/videos/wwdc/201?/*”]
```

Consider this fact while determining the order, as the system examines the path sequentially and stops when it encounters a NOT.
In reality, the reaction may be viewed at https://www.reddit.com/apple-app-site-association. Remember that the link cannot end in.json, and the request must return the content-type header application/json.

Once this is complete, we must now add the relevant information to the application. Open Xcode and navigate to Project settings > capabilities. Scroll down and enable Associated Domains. Once enabled, we will prepend app links to any URL that implements our apple-app-site-association property. Add applinks:myApp.com under the Domains section. After this is complete, proceed to test your application.
If you navigate to My App in Safari after installing an app, you should see a banner indicating that you may access the link within the app. When clicked, the application will launch.

If you enter www.myApp.com into notes, notes will detect this as a link and allow you to open it. If you click it, it will run your application without launching Safari. This will only occur when the user clicks on a link, not when they browse the webpage.
However, suppose we wish to redirect the visitor to the desired screen based on URL parameters. The following method must then be implemented in the AppDelegate.

```swift
func application(_ application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool{
    if let url = userActivity.webpageURL {
        var view = url.lastPathComponent
        var parameters: [String: String] = [:]
        URLComponents(url: url, resolvingAgainstBaseURL: false)?.queryItems?.forEach {
            parameters[$0.name] = $0.value
        }

        redirect(to: view, with: parameters)
    }
    return true
}
```

Universal links enable us to not only have a unique URL for the website, the iOS app, and maybe the Android app as well. Moreover, Universal links enable us to create a fallback page if the user does not have the app installed. If you send an email to a user with the URL www.myApp.com/app/profile, you might host there a static webpage that directs the user to the App Store or even automatically redirects it, as the link will open the app if the user already has it, and if not, the webpage will load.

Taking all of this into account, deep linking is an essential feature of modern apps, particularly if you send email alerts to your customers. Every day, users consume material more quickly, and the time it takes them to access your app and go to the right destination is time you are wasting. What are you waiting for, then?
