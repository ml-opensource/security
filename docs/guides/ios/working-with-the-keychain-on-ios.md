# Working with the keychain on iOS

![iOS](https://img.shields.io/badge/platform-iOS-blue)

The iOS Keychain can be thought of as a specialized database for storing metadata and sensitive information. The Keychain is the bests place to store small pieces of critical data, such as passwords or secrets.

Apple's API to use the Keychain is old and ugly (you'd have to use the Security framework that is mostly written in C), so we prefer using a wrapper on top of it. Currently, our go-to library for working with the Keychain is [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess).

## Example of working with the Keychain

```swift
import KeychainAccess

public enum KeychainCredentialType: String, CaseIterable {
    case pin, token, currentUser // Add a case for each of the strings you want to store in the Keychain
}

public protocol KeychainRepositoryType: class {
    func storeCredential(_ string: String, type: KeychainCredentialType)
    func getCredential(_ type: KeychainCredentialType) -> String?
    func delete(_ type: KeychainCredentialType)
    func deleteAll()
}

public class KeychainRepository: KeychainRepositoryType {

    private let keychain = Keychain(service: Bundle.main.bundleIdentifier!)

    public func storeCredential(_ string: String, type: KeychainCredentialType) {
        do {
            try keychain.set(string, key: type.rawValue)
        } catch {
            print(error.localizedDescription)
        }
    }

    public func getCredential(_ type: KeychainCredentialType) -> String? {
        return keychain[type.rawValue]
    }

    public func delete(_ type: KeychainCredentialType) {
        keychain[type.rawValue] = nil
    }

    public func deleteAll() {
        for type in KeychainCredentialType.allCases {
            keychain[type.rawValue] = nil
        }
    }

}


```

## Keychain data persistency

Even when the app is deleted, the data stored in the Keychain persists. If you reinstall the app, it will be able to read the data it had stored previously in the Keychain.

In other words, if a user sells his device without performing a factory reset, the buyer of the device may be able to get access to the previous user's app accounts and data by reinstalling the same apps used by the previous owner.

There's no iOS API that you can use to force wipe data from the Keychain when the app has been uninstalled. Instead, if you want to prevent Keychain data persistence between app installs, you sen set a flag in `UserDefaults` to wipe all previous app data from the Keychain on the app's first run:

```swift
// Do this first thing after the app has finished launching
let userDefaults = UserDefaults.standard

if userDefaults.bool(forKey: "hasRunBefore") == false {
 // Remove Keychain items
 KeychainRepository().deleteAll()

 // Update the flag indicator
 userDefaults.set(true, forKey: "hasRunBefore")
 userDefaults.synchronize() // Forces the app to update UserDefaults
}
```

Whether we want to keep data in the Keychain between app installations as a "feature", or we want to delete it, as a security measure, is a different discussion. This is something that needs to be addressed per project; the requirements might be different from one project to another.
