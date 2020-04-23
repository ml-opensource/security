# Encrypting local cache on iOS

![iOS](https://img.shields.io/badge/platform-iOS-blue)

How to do this depends on the caching solution you chose.

## [Cashier](https://github.com/nodes-ios/cashier)

To encrypt the contents of the cache, set the `encryptionEnabled` flag on the cache. This will make sure file that the cached objects are written to will be written using [NSDataWritingFileProtectionComplete](https://developer.apple.com/documentation/foundation/nsdata/writingoptions/1617198-completefileprotection). That flag is `false` by default, so you need to change it to `true` in order to encrypt the cache contents.

## [Cache](https://github.com/hyperoslo/cache)

Specify `protectionType` on `DiskConfig` to add a level of security to files stored on disk by your app in the app’s container. See [FileProtectionType](https://developer.apple.com/documentation/foundation/fileprotectiontype)

Example:

```swift
let diskConfig = DiskConfig(
  // The name of disk storage, this will be used as folder name within directory
  name: "Floppy",
  // Expiry date that will be applied by default for every added object
  // if it's not overridden in the `setObject(forKey:expiry:)` method
  expiry: .date(Date().addingTimeInterval(2*3600)),
  // Maximum size of the disk cache storage (in bytes)
  maxSize: 10000,
  // Where to store the disk cache. If nil, it is placed in `cachesDirectory` directory.
  directory: try! FileManager.default.url(for: .documentDirectory, in: .userDomainMask,
    appropriateFor: nil, create: true).appendingPathComponent("MyPreferences"),
  // Data protection is used to store files in an encrypted format on disk and to decrypt them on demand
  protectionType: .complete
)
```

## UserDefaults

UserDefaults is not a secure place to store stuff. If the info you need to save is sensitive, don't store it in UserDefaults.
