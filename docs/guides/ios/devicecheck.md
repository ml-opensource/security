# DeviceCheck

![iOS](https://img.shields.io/badge/platform-iOS-blue)

The DeviceCheck is an API introduced in iOS 11. It allows the developer to set and query 2 bits of data per device, while maintaining the user's privacy. A common usecase for this API is to verify if a device has already taken advantage of a promotional offer, or to flag a device that you have determined to be fraudulent. It also allows you to verify that the token you receive comes from an authentic Apple device on which your app has been downloaded.

Do something like this to get the token:

```swift
let curDevice = DCDevice.current
if curDevice.isSupported
{
    curDevice.generateToken(completionHandler: { (data, error) in
        if let tokenData = data {
            print("Received token \(tokenData)")
        } else {
            print("Hit error: \(error!.localizedDescription)")
        }
    })
}
```

Then you send your token to your server, your server can set one of the bits (or not) and then send it to Apple. Then, when you query the token again from the app, it would have the updated value.

Further reading:

1. [Uniquely identify iOS device using DeviceCheck (Tutorial)](https://fluffy.es/devicecheck-tutorial/)
2. [iOS 11: The DeviceCheck API](https://medium.com/the-traveled-ios-developers-guide/devicecheck-6f3eafac60e5)
3. [DeviceCheck](https://developer.apple.com/documentation/devicecheck) and [Accessing and Modifying Per-Device Data](https://developer.apple.com/documentation/devicecheck/accessing_and_modifying_per-device_data)
