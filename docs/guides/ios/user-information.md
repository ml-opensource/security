# Encrypting user information

![iOS](https://img.shields.io/badge/platform-iOS-blue)

Sensitive user information, such as email address, full name, shipping address, etc, can also be stored more securely in the Keychain. If we want to do that, we can serialize the whole user object and store it in the Keychain.

Apple's API to use the Keychain is old and ugly, so we prefer using a wrapper on top of it. Currently, our go-to library for working with the Keychain is [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess). When working with the Keychain, it's important to know that by default **[the Keychain data persists across app installs](working-with-the-keychain-on-ios.md#Keychain-data-persistency)**.

We will use a similar wrapper over the Keychain as when storing the user token, but we'll add another credential type in the enum (`currentUser`, in this example).

```swift
public enum KeychainCredentialType: String, CaseIterable {
    case pin, token, currentUser
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

Here's an example of how to save the current user in the Keychain:

```swift
public class UserManager {

    private static let keychain = KeychainRepository()

    static var currentUser: User? {
        get {
            if let serializedUserJson = keychain.getCredential(.currentUser) {
                let serializedUserData = Data(serializedUserJson.utf8)
                let decoder = JSONDecoder()
                return try? decoder.decode(User.self, from: serializedUserData)
            }
            return nil

        }
        set {
            if let user = newValue {
                // If there is a new value, set it
                let encoder = JSONEncoder()
                if let serializedUserData = try? encoder.encode(user), let serializedUserJson = String(data: serializedUserData, encoding: .utf8) {
                    keychain.storeCredential(serializedUserJson, type: .currentUser)
                }
            } else {
                // Otherwise delete from cache
                keychain.delete(.currentUser)
            }
        }
    }
}
```

Whenever something mutates the `currentUser`, we'll set it on the `UserManager` and it will automatically be saved in the Keychain. For example:

```swift
ConnectionManager.getCurrentUser { user in
  UserManager.currentUser = user
}
```

Implementation details will vary from one project to another. This is just an example.
