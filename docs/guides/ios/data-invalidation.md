# Data invalidation

![iOS](https://img.shields.io/badge/platform-iOS-blue)

## App uninstall

- data saved in `UserDefaults` is deleted automatically when the user deletes the app
- data saved in the Keychain, though, persists even when the user uninstalls the app. This can be seen as a feature, but also as a potential security risk. [Read more about this and how to prevent it](https://github.com/nodes-projects/readme/blob/master/mobile/ios/security/working-with-the-keychain-on-ios.md#Keychain-data-persistency).

## User logout

- how to do this depends on the specific app's logic, implementation and caching solution used
- when a user logs out, all his data needs to be wiped (local caches, user token, etc); otherwise, if a new user logs in, he might be able to see the data of the previous user
