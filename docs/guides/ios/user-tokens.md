# Secure auth token storage

![iOS](https://img.shields.io/badge/platform-iOS-blue)

It's best practice to store the tokens on the [Keychain](https://developer.apple.com/documentation/security/keychain_services) as it handles encryption for you. [NSUserDefaults](https://developer.apple.com/documentation/foundation/nsuserdefaults) is not secure or encrypted so it can be easily opened and read, both on device and when synced to a Mac so **DO NOT** use it for storing sensitive information.

**When working with the Keychain, it's important to know that by default [the Keychain data persists across app installs](working-with-the-keychain-on-ios.md#Keychain-data-persistency).**

Apple's API to use the Keychain is old and ugly, so we prefer using a wrapper on top of it. Currently, our go-to library for working with the Keychain is [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess).

## Example using KeychainAccess

```swift
import KeychainAccess

public enum KeychainCredentialType: String, CaseIterable {
    case pin, token
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
