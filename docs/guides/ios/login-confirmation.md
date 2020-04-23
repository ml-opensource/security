# Biometrics

![iOS](https://img.shields.io/badge/platform-iOS-blue)

On iOS devices, authentication with biometrics means TouchID (fingerprint) and FaceID (face recognition).
Both of those are done on device, there's no server involved. To improve security, local authentication with biometrics should also be enforced at a remote endpoint.

The best way to integrate local authentication in an iOS app is using the [Local Authentication framework](https://developer.apple.com/documentation/localauthentication).

There are two policies that can be evaluated: `.deviceOwnerAuthentication` (which will require the device's passcode if Face ID / Touch ID aren't available and fail if the device passcode is not enabled) and `deviceOwnerAuthenticationWithBiometrics` (which will restrict to Face ID / Touch ID and fail if they're not available).

Not all devices support Local Authentication, and authentication can also fail for various reasons (no Touch ID or Face ID available or set up on device, no passcode set, user cancelled, etc). A fallback option needs to be provided (for example, the user has to type his email and password to login). Biometrics login should be used as a supplement to something the app is already doing, they shouldn't be the only authentication option.

## Sample code

In any project that uses biometrics, include the NSFaceIDUsageDescription key in your app’s Info.plist file. The system doesn’t require a comparable usage description for Touch ID.

```swift
let context = LAContext()
var error: NSError?

guard context.canEvaluatePolicy(.deviceOwnerAuthentication, error: &error) else {
 // Could not evaluate policy; look at error and present an appropriate message to user
}

context.evaluatePolicy(.deviceOwnerAuthentication, localizedReason: "Please, pass authorization to enter this area") { success, evaluationError in
 guard success else {
  // User did not authenticate successfully, look at evaluationError and take appropriate action
 }

 // User authenticated successfully, take appropriate action
}
```
